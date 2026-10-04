# Assistente IA Code Runner 2.6.0-teste — Cadeia mista + descoberta de modelos

Base: 2.5.2-diario (a pasta/zip da 2.5.2 não foi tocada). Trabalho feito na cópia `assistente_ia_code_runner_2_6_0/`.
Data: 2026-10-04. `python -m pytest tests` → **65 passed** (sem QGIS e sem rede). `ast.parse` de todos os `.py` OK.

## 1. O que foi pedido

1. **Ativar a Cadeia mista** (Gemini → Groq → GitHub Models → OpenRouter).
2. **Aplicar a ela as regras da cadeia Gemini.**
3. Fases 0–3 do prompt 2.6.0 (descoberta e teste de modelos).

## 2. Causa raiz de a Cadeia mista "não estar ativa"

O dock já montava `cadeia_mista`, mas em ⚙ (`dialog_configuracoes.py`): (a) o combo não tinha a opção; (b) a linha do provedor salvo
forçava qualquer valor desconhecido para `"cadeia"`; (c) `groq`, `github`, `openrouter` não estavam em `_PROVEDORES_FOLHA`
(não havia onde digitar chave/modelo); (d) a etapa só listava as pernas Gemini.
`configuracoes._MODELOS_PADRAO` também não tinha `github`/`openrouter` (perna com modelo vazio).

## 3. Regras do Gemini aplicadas a Groq, GitHub e OpenRouter

Nova classe `_SessaoCompativelComRegrasGemini` (herda de `_SessaoCompativelOpenAI`); só as 3 sessões passam a herdá-la.
**OpenAI, DeepSeek e xAI ficam exatamente como na 2.5.2** (há teste que prova).

| Regra do Gemini | Agora nas 3 pernas |
|---|---|
| Janela de 5 turnos | sim (`_MAX_TURNOS_HISTORICO` = o do Gemini; 3 no perfil enxuto) |
| Turnos antigos guardados localmente, sem chamar a IA | sim (`_guardar_turno_antigo`, até 200) |
| `consultar_historico_antigo` | registrada a cada mensagem + definição da ferramenta acrescentada (antes só existia p/ Gemini) |
| Nota `_NOTA_HISTORICO_ANTIGO` no prompt | sim |
| Truncagem de 15.000 caracteres no retorno de ferramentas | sim (reusa `SessaoGemini._limitar_retorno`) |
| Handoff com mensagens antigas "sem resumir" | sim (`resumo_para_handoff`) |

Já valiam para todos (estão no dock): diário de execução, `reler_diario_execucao`, backup com retenção de 5.
**Não se aplica:** botão ⏭ e limite de espera do Gemini (dependem do SDK dele).

## 4. Fase 0 — pesquisa (2026-10-04)

**Limitação:** `console.groq.com`, `openrouter.ai` e `docs.github.com` estavam bloqueados pelo proxy da sessão. Os valores abaixo vêm de
resultados de busca (fontes secundárias, exceto onde indicado). **Confirme na 1ª execução real.**

| | Groq | OpenRouter | GitHub Models |
|---|---|---|---|
| Listagem | `GET https://api.groq.com/openai/v1/models`, Bearer. **Confirmado pelo seu print da doc oficial (04/10/2026):** "JSON list of all active models" | `GET https://openrouter.ai/api/v1/models` | `GET https://models.github.ai/catalog/models` (+ `Accept: application/vnd.github+json`, `X-GitHub-Api-Version`) |
| Campos úteis | `id`, `active`, `context_window` (não informa ferramentas → o ping decide) | `pricing.prompt/completion` (strings; grátis = ambos `0`), `supported_parameters` (contém `tools`), `context_length`, `architecture.output_modalities` | `capabilities` (contém `tool-calling`), `limits.max_input_tokens`, `rate_limit_tier`, `supported_output_modalities` |
| Chat | `https://api.groq.com/openai/v1/chat/completions` | `https://openrouter.ai/api/v1/chat/completions` | `POST https://models.github.ai/inference/chat/completions` (id `publisher/modelo`, token com `models:read`) |
| Grátis hoje | gpt-oss-120b e similares: 30 RPM, 1.000 RPD, **8.000 TPM**, 200.000 TPD; llama-3.3-70b: 12.000 TPM / 100.000 TPD; por organização **e por modelo** | `:free`: **20 RPM**; **50 req/dia** (1.000 com ≥ US$ 10 em créditos) | ≈ **8.000 tokens de entrada**/4.000 saída por requisição; ≈ 10 RPM / 50 RPD (nível alto), 150 RPD (mini) |
| Erros | 429 + cabeçalhos `x-ratelimit-*`; 413/400 p/ tamanho | 402 com saldo negativo (**até em modelo grátis**); 404 "No endpoints found that support tool use" | 413/`tokens_limit_reached` (corpo grande demais); 401/403 |

**Riscos:** (1) limites são de terceiros; (2) o modelo `:free` do OpenRouter muda toda hora; (3) ping com `max_tokens=64` pode cortar modelo "raciocinador" antes da chamada — vira `erro (inconclusivo)`, não `sem_ferramentas`.

## 5. ⚠️ PENDÊNCIA que precisa da sua decisão — endereço do GitHub Models

O prompt manda parar e perguntar quando o endereço mudou. **Mudou:** o código usa `https://models.inference.ai.azure.com/chat/completions`
(`SessaoGitHub._URL`, não alterei); a API atual é `https://models.github.ai/inference/chat/completions`, e os ids do catálogo são `publisher/modelo` (ex.: `openai/gpt-4.1`).

Consequência hoje: a **verificação** usa o endereço novo, mas a **Cadeia** usa o antigo. Modelo GitHub marcado no catálogo pode dar 404 na Cadeia
(que avança sozinha para a próxima etapa, sem travar). **Correção de 1 linha**, aguardando seu OK:
`provedor_ia.py`, em `class SessaoGitHub`: `_URL = "https://models.github.ai/inference/chat/completions"`.
Também ficou como padrão `github: gpt-4o-mini` (id antigo) e `openrouter: openai/gpt-oss-20b:free` — chutes; "Verificar modelos" descobre os reais.

## 6. Fase 3 — perfil enxuto (implementado, desligado por padrão)

Medido no código real: prompt do sistema **12.860 caracteres**, 18 ferramentas **16.194 caracteres** → ≈ **9.700 tokens** fixos (teste `test_perfil_completo_continua_grande` > 8.000).
Isso estoura Groq (8.000 TPM) e GitHub (8.000 de entrada) grátis.

`perfil_enxuto.py` (novo): prompt compacto com todas as regras de segurança (≈ 1.500 tokens) + 5 ferramentas
(`executar_codigo_pyqgis`, `obter_info_projeto`, `reler_diario_execucao`, `ler_memoria_permanente`, `consultar_historico_antigo`) + janela de 3 turnos + memória cortada em 2.000 caracteres.
Total medido: **< 3.500 tokens** (teste). Ligado **só** nas etapas `groq` e `github` da Cadeia mista; padrão do parâmetro = comportamento antigo.
O bloco do diário (até ≈ 7.500 caracteres ≈ 2.500 tokens) vem na 1ª mensagem e ainda cabe em 8.000.
**Custo:** nessas etapas o modelo não tem as ferramentas de GitHub, scripts salvos, lições e limpeza (o prompt avisa e manda usar as etapas Gemini).

## 7. Arquivos, funções e linhas alteradas (vs 2.5.2)

Diff completo para revisão: `DIFF_2_5_2_para_2_6_0.patch`. Linhas removidas/alteradas no total: 14 (todas listadas).

**NOVOS (nada existia antes):** `descoberta_modelos.py`, `perfil_enxuto.py`, `worker_verificacao.py`, `tests/*`.

**`provedor_ia.py`** (+116 / −4): classe nova `_SessaoCompativelComRegrasGemini` (linhas 1333–1442);
alteradas: 1446 `SessaoGroq`, 1450 `SessaoOpenRouter`, 1454 `SessaoGitHub` (só a classe-mãe);
`criar_sessao` (assinatura + ramo `classe_openai`: parâmetro opcional `perfil_enxuto`); `SessaoCadeia._sessao_atual` (+1 linha repassando `perfil_enxuto`).
Não tocados: `SessaoGemini`, `_modelos_pendentes`, `modelos_alternativos`, `_tentar_modelos_alternativos`, `SessaoCadeia._eh_erro_escalavel`.

**`configuracoes.py`** (+12 / −0): `_MODELOS_PADRAO` (+`github`, +`openrouter`); novas `obter_etapa_cadeia_mista` / `salvar_etapa_cadeia_mista`.

**`dialog_configuracoes.py`** (+228 / −5): `_PROVEDORES` (+`cadeia_mista`); novas `_PERNAS_CADEIA_MISTA`, `_PROVEDORES_VERIFICACAO`, `_ICONES_STATUS`, `_MODELOS_SUGERIDOS.update(...)` (só chaves groq/github/openrouter) e `_PROVEDORES_FOLHA += [...]`;
`__init__` (whitelist do provedor; `_repopular_pernas` + `_etapa_salva_para`; grupo no layout); `_slot_atual`, `_atualizar_visibilidade_campos`, `_ao_mudar_provedor`, `salvar_e_fechar` (etapa mista em chave própria; `_gravar_modelos_marcados`);
métodos novos no fim da classe (grupo "Modelos verificados", botões Verificar / Forçar / Cancelar, lista com checkbox, `done`).
**As listas `_MODELOS_SUGERIDOS["gemini*"]` e `_PERNAS_CADEIA` não foram tocadas.**

**`dock_assistente.py`** (+54 / −1): `abrir_configuracoes` (invalida a Cadeia mista ao mudar etapa/modelos marcados); métodos novos `_modelos_verificados_marcados` e `_complementar_pernas_mista`;
`_garantir_sessao` (chamada do complemento; etapa mista própria — a única linha removida). **`_PERNAS_CADEIA` / `_PERNAS_CADEIA_MISTA` intactas.**

**`metadata.txt`:** `version=2.6.0-teste`. `changelog` não preenchido.

## 8. Descoberta e teste de modelos (Fases 1–2)

- Só sob demanda (botão); **nada roda ao abrir o QGIS nem o diálogo**. Rede em `QThread`.
- Fluxo: **Verificar** → lista e filtra (sem gastar tokens) → mostra estimativa e pede **confirmação** → ping com ferramenta fictícia → teste de realidade com a **classe real** (perfil que a Cadeia usaria) → lista com ✅ ⚠️ 🐘 ⏳ ❌ → você marca → **OK** grava.
- Candidatos: exclui embed/whisper/tts/audio/image/rerank/moderation/guard/safeguard; OpenRouter só preço zero **e** `tools`; nunca testa modelo pago. Máx. **8** por execução (`MAX_TESTES_POR_EXECUCAO`), 1,5 s de pausa, sequencial, sem retentativa.
- Para **todo o provedor** no 1º 429 ou 401/403 (resto = "não testado"). 429 por tokens no teste de realidade vira `grande_demais` e não para.
- Cache `AssistenteIACodeRunner/modelos_verificados_<provedor>`; não retesta o mesmo modelo no mesmo dia (exceto "Forçar novo teste"). Os resultados são gravados na hora, mas a **marcação (`usar`) só ao confirmar o diálogo**.
- Marcados viram etapas **no fim** da Cadeia mista (ordem Groq → GitHub → OpenRouter), sem mudar as etapas fixas. Sem marcados: pernas idênticas à 2.5.2 (exceto o perfil enxuto em Groq/GitHub) — teste `test_sem_marcados_pernas_identicas_exceto_perfil_enxuto`.
- Chave de API nunca em `detalhe`, log, `print` nem mensagem de erro (`_sanear`; teste dedicado).

## 9. Checklist manual no QGIS (você executa)

1. Instalar a 2.6.0-teste em cópia do perfil. Abrir ⚙ → provedor **Cadeia mista**: o combo "Etapa" lista as 5 etapas mistas.
2. Em cada etapa Groq / GitHub / OpenRouter, digitar a chave (cada uma no seu slot).
3. **🔍 Verificar modelos** no Groq: aparece a estimativa → Sim → progresso na linha de status → lista com ícones. Desmarque algum, OK.
4. Reabrir ⚙: a marcação persiste. Conferir que o QGIS abriu sem rodar nenhuma verificação.
5. Chat com a Cadeia mista; trocar para uma etapa Groq/GitHub e conferir que responde (perfil enxuto).
6. Forçar 404: marcar um modelo inexistente (digitando um id errado) e ver a Cadeia avançar para a próxima etapa.
7. Voltar para "Fluxo Gemini": a etapa antiga da Cadeia Gemini continua como estava.
8. Conferir que o ⚙ da Cadeia Gemini e do Gemini único aparecem sem mudança.

## 10. Não testado (honestidade)

- `dialog_configuracoes.py`, `worker_verificacao.py` e a parte de UI do dock **não foram executados no QGIS** (só `ast.parse` e revisão); os testes cobrem núcleo, regras e montagem das pernas.
- Nenhuma chamada real a Groq/OpenRouter/GitHub foi feita (sem chaves; rede bloqueada).

## 11. DeepSeek — avaliado, NÃO implantado (aguarda sua decisão)

Pesquisa de 2026-10-04 (fontes secundárias): a API oficial do DeepSeek **não tem plano grátis recorrente**; só créditos de boas-vindas
únicos (≈ 5 milhões de tokens, com validade) e depois é pago por token (≈ US$ 0,28/M entrada, US$ 0,42/M saída). O chat web é grátis, mas não é API.
Uma fonte de julho/2026 diz que o OpenRouter **não tem mais** modelos DeepSeek `:free`.
Decisão tomada: **fora da Cadeia**, porque (a) não é grátis de forma contínua; (b) com saldo recarregado a Cadeia gastaria dinheiro sem aviso (regra 8 do prompt); (c) o prompt exclui o DeepSeek do escopo.
Já coberto sem código extra: se o OpenRouter voltar a listar um DeepSeek `:free`, "Verificar modelos" o encontra e você o marca.
Wrappers não oficiais de "DeepSeek grátis" (link `FreeDeepseekAPI-EN`) **não foram abertos nem recomendados**: costumam usar sessão/token de terceiros e o plugin envia código do projeto às IAs.
