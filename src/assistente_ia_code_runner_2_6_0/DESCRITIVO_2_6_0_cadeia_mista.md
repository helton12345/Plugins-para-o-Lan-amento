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

## 5. Endereço do GitHub Models — ATUALIZADO (aprovado pelo usuário)

`provedor_ia.py`, `SessaoGitHub._URL`: `https://models.inference.ai.azure.com/chat/completions` → `https://models.github.ai/inference/chat/completions`
(único uso do endereço antigo no plugin; verificação e Cadeia agora usam o mesmo). Ids passam a ser `publisher/modelo`:
padrão `github: openai/gpt-4o-mini` (configuracoes.py) e sugestões `openai/gpt-4o-mini`, `openai/gpt-4o` (dialog). Se você já tinha salvo um id antigo (ex.: `gpt-4o`) em ⚙, troque para `openai/gpt-4o`.
O token do GitHub precisa da permissão `models:read`. Padrão do OpenRouter (`openai/gpt-oss-20b:free`) continua um chute: "Verificar modelos" descobre os reais.

## 6. Fase 3 — perfil enxuto (implementado, desligado por padrão)

Medido no código real: prompt do sistema **12.860 caracteres**, 18 ferramentas **16.194 caracteres** → ≈ **9.700 tokens** fixos (teste `test_perfil_completo_continua_grande` > 8.000).
Isso estoura Groq (8.000 TPM) e GitHub (8.000 de entrada) grátis.

`perfil_enxuto.py` (novo): prompt compacto com todas as regras de segurança (≈ 1.500 tokens) + 6 ferramentas
(`executar_codigo_pyqgis`, `obter_info_projeto`, `reler_diario_execucao`, `ler_memoria_permanente`, `consultar_historico_antigo`) + janela de 3 turnos + memória cortada para começo (800) + fim (2.400 caracteres).
Total medido: **≈ 2.900 tokens** (teste exige < 4.000). Ligado **só** nas etapas `groq` e `github` da Cadeia mista; padrão do parâmetro = comportamento antigo.
O bloco do diário (até ≈ 7.500 caracteres ≈ 2.500 tokens) vem na 1ª mensagem e ainda cabe em 8.000.
**Custo:** nessas etapas o modelo não tem as ferramentas de GitHub, scripts salvos, lições e limpeza (o prompt avisa e manda usar as etapas Gemini).

## 7. Arquivos, funções e linhas alteradas (vs 2.5.2)

Diff completo para revisão: `DIFF_2_5_2_para_2_6_0.patch`. Linhas removidas/alteradas no total: 14 (todas listadas).

**NOVOS (nada existia antes):** `descoberta_modelos.py`, `perfil_enxuto.py`, `worker_verificacao.py`, `tests/*`.

**`provedor_ia.py`** (+143 / −6, já com os itens 15 e 16): classe nova `_SessaoCompativelComRegrasGemini` (linhas 1333–1442);
alteradas: 1446 `SessaoGroq`, 1450 `SessaoOpenRouter`, 1454 `SessaoGitHub` (classe-mãe **e** `_URL`);
`criar_sessao` (assinatura + ramo `classe_openai`: parâmetro opcional `perfil_enxuto`); `SessaoCadeia._sessao_atual` (+1 linha repassando `perfil_enxuto`).
Não tocados: `SessaoGemini`, `_modelos_pendentes`, `modelos_alternativos`, `_tentar_modelos_alternativos`, `SessaoCadeia._eh_erro_escalavel`.

**`configuracoes.py`** (+12 / −0): `_MODELOS_PADRAO` (+`github`, +`openrouter`); novas `obter_etapa_cadeia_mista` / `salvar_etapa_cadeia_mista`.

**`dialog_configuracoes.py`** (+244 / −6, já com os itens 14 e 18): `_PROVEDORES` (+`cadeia_mista`); novas `_PERNAS_CADEIA_MISTA`, `_PROVEDORES_VERIFICACAO`, `_ICONES_STATUS`, `_MODELOS_SUGERIDOS.update(...)` (só chaves groq/github/openrouter) e `_PROVEDORES_FOLHA += [...]`;
`__init__` (whitelist do provedor; `_repopular_pernas` + `_etapa_salva_para`; grupo no layout); `_slot_atual`, `_atualizar_visibilidade_campos`, `_ao_mudar_provedor`, `salvar_e_fechar` (etapa mista em chave própria; `_gravar_modelos_marcados`);
métodos novos no fim da classe (grupo "Modelos verificados", botões Verificar / Forçar / Cancelar, lista com checkbox, `done`).
**As listas `_MODELOS_SUGERIDOS["gemini*"]` e `_PERNAS_CADEIA` não foram tocadas.**

**`dock_assistente.py`** (+163 / −13, já com os itens 12, 13, 16, 17 e 18): `abrir_configuracoes` (invalida a Cadeia mista ao mudar etapa/modelos marcados); métodos novos `_modelos_verificados_marcados` e `_complementar_pernas_mista`;
`_garantir_sessao` (chamada do complemento; etapa mista própria — a única linha removida). **`_PERNAS_CADEIA` / `_PERNAS_CADEIA_MISTA` intactas.**

**`metadata.txt`:** `version=2.6.0-teste`. `changelog` não preenchido.

## 8. Descoberta e teste de modelos (Fases 1–2)

- Só sob demanda (botão); **nada roda ao abrir o QGIS nem o diálogo**. Rede em `QThread`.
- Fluxo: **Verificar** → lista e filtra (sem gastar tokens) → mostra estimativa e pede **confirmação** → ping com ferramenta fictícia → teste de realidade com a **classe real** (perfil que a Cadeia usaria) → lista com ✅ ⚠️ 🐘 ⏳ ❌ → você marca → **OK** grava.
- Candidatos: exclui embed/whisper/tts/audio/image/rerank/moderation/guard/safeguard; OpenRouter só preço zero **e** `tools`; nunca testa modelo pago. Máx. **8** por execução (`MAX_TESTES_POR_EXECUCAO`), 1,5 s de pausa, sequencial, sem retentativa.
- Para **todo o provedor** no 1º 429 ou 401/403 (resto = "não testado"). 429 por tokens no teste de realidade vira `grande_demais` e não para.
- Cache `AssistenteIACodeRunner/modelos_verificados_<provedor>`; não retesta o mesmo modelo no mesmo dia (exceto "Forçar novo teste"). Os resultados são gravados na hora, mas a **marcação (`usar`) só ao confirmar o diálogo**.
- ~~Marcados viram etapas no fim da Cadeia mista~~ **substituído no item 18**: os verificados entram NO LUGAR da etapa fixa do provedor. Sem marcados: pernas idênticas à 2.5.2 (exceto o perfil enxuto em Groq/GitHub) — teste `test_sem_marcados_pernas_identicas_exceto_perfil_enxuto`.
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

## 12. Ajustes pedidos depois (A e B — chamadas à toa no início)

Diagnóstico: o diário só era lido na 1ª mensagem; com o projeto ainda sem arquivo salvo, nunca mais era tentado, e o Smith improvisava scripts PyQGIS e relia a memória (já carregada) — 8 chamadas em vez de 2.
- **A — `dock_assistente.py`:** `_bloco_diario_primeira_mensagem` marca `_diario_pendente=True` quando o projeto ainda não foi salvo (aviso uma vez só); `enviar_mensagem` passa a tentar o diário também quando `_diario_pendente`
  (linha `if conversa_nova or getattr(self, "_diario_pendente", False):`, a única linha alterada). Qualquer outra situação (lido, não achei, erro) zera o pendente.
- **B — `_SYSTEM_INSTRUCTION` (dock):** parágrafo "ECONOMIA DE CHAMADAS" ao final (não chamar `obter_info_projeto` se o contexto já veio; não reler a memória carregada; ler o diário com `reler_diario_execucao`, nunca por script). O perfil enxuto não mudou.
- Testes novos: 3 (`68 passed`).
- **Backup (corrigido a pedido do usuário):** a pasta de backup tinha sido escolhida **dentro** da pasta do projeto; o plugin então apagava a configuração salva (`salvar_pasta_backup("")`) e abria o diálogo a cada script.
  Agora, em `dock_assistente.py` (`_ExecutorThreadPrincipal._fazer_backup_projeto_atual`):
  - subpasta do projeto como pasta de backup é **aceita e mantida**, sem diálogo; o zip simplesmente **deixa essa subpasta de fora** (senão um backup entraria no outro). Helpers novos: `_arquivos_do_projeto`, `_tamanho_sem_pasta`, `_zipar_sem_pasta`; `import os` e `import zipfile` acrescentados;
  - o limite de 500 MB passa a ignorar a pasta de backups nesse caso;
  - **pasta fora do projeto: caminho antigo idêntico** (`make_archive`);
  - a **própria pasta do projeto** como destino continua recusada (apaga a configuração e avisa), pois nada sobraria para zipar — única linha de texto alterada;
  - mantidos: poda de 5 zips por projeto, fallback "só copia o arquivo do projeto" acima do limite.
  - 4 testes novos (72 no total), incluindo "o diálogo de pasta não pode abrir" e "um zip não contém o outro".
  - **Atenção:** `limpar_arquivos_nao_usados` só ignora pastas chamadas `_backups_automaticos_smith` ou `_backups` (`ferramentas_ia.py`, `_PASTAS_BACKUP_IGNORADAS`, não alterado). Se a sua pasta de backup dentro do projeto tem outro nome, os zips dela podem aparecer na lista de "arquivos não usados" — não aprove a limpeza dela, ou renomeie a pasta para `_backups`.

## 13. Limpeza de arquivos DESLIGADA (pedido do usuário)

`ferramentas_ia.py` (+9 linhas no fim): conjunto `FERRAMENTAS_DESLIGADAS = {"analisar_arquivos_nao_usados", "limpar_arquivos_nao_usados"}` e filtro in-place dos 3 registros
(`FERRAMENTAS_AGENTE` — Gemini; `DEFINICOES_FERRAMENTAS_CLAUDE` — Claude, OpenAI-compatíveis e lista do MCP; `_FERRAMENTAS_POR_NOME` — worker e MCP). Ficam 16 definições em vez de 18.
O código das funções permanece: **para religar, esvazie `FERRAMENTAS_DESLIGADAS`**.
`dock_assistente.py`: parágrafo "LIMPEZA DE ARQUIVOS NÃO USADOS: está DESLIGADA…" ao final de `_SYSTEM_INSTRUCTION`. `perfil_enxuto.py`: frase ajustada. Teste novo (73 no total).
Não alterados: popup/backup do `executar_codigo_pyqgis`, `FERRAMENTAS_DESTRUTIVAS_LOCAIS`, `_PASTAS_BACKUP_IGNORADAS`.

## 14. Janela de configuração com barras de rolagem (pedido do usuário)

`dialog_configuracoes.py`, só o trecho de layout do `__init__`: formulário + grupo "Modelos verificados" dentro de `QScrollArea` (`setWidgetResizable(True)`, barras vertical e horizontal "conforme necessário");
**OK/Cancelar ficam fixos fora da rolagem**; tamanho inicial 780×620. Import acrescentado: `QScrollArea`, `QWidget`. Lógica dos campos, visibilidade por provedor e gravação: inalteradas.
**Verificado em Qt real** (PyQt5 offscreen, `tests/dialogo_qt_real_script.py`, rodado em processo separado pelo pytest): rolagem presente e vertical ativa em janela pequena; botões fora da rolagem;
Cadeia mista lista as 5 etapas, guarda chave por provedor, modelo padrão `openai/gpt-4o-mini` no GitHub, etapa mista em chave própria; Fluxo Gemini intacto. É a primeira execução real do diálogo (74 testes no total).
Ainda não testado: worker de rede, dock completo e aparência no QGIS do Windows.

## 15. "Pulando entre Gemini e Groq" + listagem do GitHub

**Causa (log do usuário):** a cota diária do Gemini (20 pedidos/dia por modelo) esgotou. `SessaoCadeia.enviar_mensagem` voltava à etapa principal 60 s depois de qualquer erro ("recuperando prioridade"): cada mensagem
recomeçava a lista de modelos do Gemini (4–5 tentativas à toa), caía no Groq de novo e ainda pausava ("Pausei porque o modelo mudou"). Comportamento da 2.5.2, só visível agora que a Cadeia mista é alcançável.
Além disso, um **erro meu**: o perfil enxuto cortava a memória nos primeiros 2.000 caracteres, e o resumo de passagem entre etapas é colado no FIM da mesma memória — o Groq nunca recebia o contexto ("qual é a próxima ação?").
- **A — `provedor_ia.py`, `SessaoCadeia`:** `_sem_prioridade_ate`, `_ESPERA_COTA_DIARIA_SEG = 1800`. Quando a etapa **principal** falha por limite **diário** (`SessaoGemini._classificar_cota(...)["tipo"] == "dia"`), a Cadeia não tenta recuperá-la por 30 min.
  Limite por minuto continua recuperando em 60 s; limite diário de etapa que não é a principal não bloqueia nada. Vale também para o Fluxo Gemini (fica 30 min na reserva em vez de testar a principal a cada minuto).
- **B — perfil enxuto:** memória = primeiros 800 + últimos 2.400 caracteres (`MAX_CHARS_MEMORIA_INICIO/FIM` em `perfil_enxuto.py`), o resumo de passagem sobrevive.
- **GitHub "Resposta de listagem inválida (não é JSON)":** sem acesso ao GitHub daqui, não deu para reproduzir. `listar_modelos` agora (1) aceita JSON com BOM, (2) no GitHub tenta uma 2ª vez com `Accept: application/json` e sem o cabeçalho de versão,
  (3) se ainda não vier JSON, mostra HTTP, tipo de conteúdo, **endereço final** (redirecionamento) e o começo da resposta, sem a chave. Esse texto diz qual é a causa real (HTML de login, redirecionamento, proxy, token sem `models:read`…).
- Testes novos (81 no total). Sem código: em ⚙ escolher a etapa "3. Groq" faz dela a principal e evita voltar ao Gemini enquanto a cota dele não volta (≈ 04h de Brasília).

## 16. Flash-Lite como agente secundário em TODAS as etapas + "não salve edições"

**Pergunta do usuário:** o Flash-Lite faz, na Cadeia mista, as funções de agente secundário (pesquisa e lições) com qualquer IA ativa? **Antes: só com o Gemini.**
- **Pesquisa** (`consultar_conhecimento_precisa` → `agente_pesquisador`, Flash-Lite): faltava no perfil enxuto de Groq/GitHub. Agora `consultar_conhecimento_precisa` está nas ferramentas enxutas (6 no total; ≈ +260 tokens, perfil ≈ 2.900). A busca roda no Flash-Lite e não pesa na cota da etapa.
- **Lição 💾** (`agente_redator`, Flash-Lite): as sessões Groq/GitHub/OpenRouter não tinham `texto_da_conversa`, então o painel caía no plano B (pedir ao próprio modelo da etapa, gastando a cota dele; no Groq/GitHub a ferramenta nem existe).
  `provedor_ia.py`: `_SessaoCompativelComRegrasGemini.texto_da_conversa` (resumo acumulado + turnos antigos + histórico atual, só leitura). OpenAI/DeepSeek/xAI: inalterados.
- **Regra "NÃO SALVE EDIÇÕES" (qualquer IA):** `dock_assistente.py`, `_SYSTEM_INSTRUCTION` (vale para Gemini, Claude, OpenAI-compatíveis e Claude Code) e `perfil_enxuto.py`: nunca `commitChanges()` nem regravar a camada original; deixar a camada em edição e **avisar o usuário** quais camadas ficaram com edições pendentes para ele salvar. Camadas novas pedidas pelo usuário (exportações) continuam permitidas.
  A frase antiga ("exige startEditing() antes e commitChanges() depois") foi trocada para não contradizer a regra nova.
  **Limite conhecido:** é regra de prompt, não trava mecânica. Scripts salvos (`executar_script_salvo`) que já tenham `commitChanges()` dentro continuam gravando; o popup de confirmação do `executar_codigo_pyqgis` continua sendo a trava real.
- Testes novos (85 no total).

## 17. TRAVA no código: scripts não salvam edições de camada nem apagam (pedido do usuário)

`ferramentas_ia.py` (total do arquivo: +152 / −4; as 4 linhas removidas são 3 de docstring e a linha do `with` do executor). Duas camadas, sempre ativas (inclusive no modo automático):
1. **`verificar_script_bloqueado(script)` — análise ANTES de rodar** (AST). Bloqueia `commitChanges`, `saveEdits`, `os.remove/unlink/rmdir/removedirs`, `shutil.rmtree`, `.unlink`, `.rmdir`, `send2trash`, `QFile.remove`, `deleteShapeFile`,
   `from os import remove…` e a string `"commitChanges"` (getattr). Nada é executado. Chamada em `executar_codigo_pyqgis` e, **antes do popup e do backup**, em `_ExecutorThreadPrincipal._executar` (dock) — serve também ao `executar_script_salvo`.
2. **`_bloquear_salvar_e_apagar()` — reforço durante a execução**, para o que escapar da análise (`getattr`, alias, módulos importados): `os.remove/unlink/rmdir/removedirs`, `shutil.rmtree`, `Path.unlink/rmdir`, `QgsVectorLayer.commitChanges` e as gravações diretas
   `addFeatures/deleteFeatures/changeAttributeValues/changeGeometryValues/addAttributes/deleteAttributes` do provedor levantam erro com a mensagem `BLOQUEADO…`. **Liberados:** camadas e provedores em **memória** (resultados temporários) e apagar dentro da **pasta temporária do sistema** (processing).
   Cada troca é protegida por `try`: se uma classe do QGIS não aceitar a troca, só a camada 1 vale. Tudo é restaurado no `finally`.
- **Continua permitido:** editar com a edição pendente (inclusive `deleteFeature` no buffer), criar camadas/arquivos novos, `removeMapLayer`, o botão Salvar edições do QGIS.
- **Limites conhecidos:** não cobre código nativo (C++/GDAL), p. ex. `QgsVectorFileWriter` regravando um arquivo existente; apagar fica proibido também para arquivos temporários do próprio script fora da pasta temp do sistema. O popup/Lixeira (`_apagar_para_lixeira`) permanece no código, mas nunca é alcançado.
- Prompts (completo e enxuto): avisam que o plugin bloqueia e que, ao receber "BLOQUEADO", a IA deve avisar o usuário em vez de contornar.
- Testes: `tests/test_trava_salvar_apagar.py` (análise, execução, reforço, restauração, dock antes do popup). **Não testado no QGIS real** (a troca de métodos de classes sip foi testada com classes de PyQt/Python comuns).

## 18. Cadeia mista: os modelos verificados SUBSTITUEM a etapa fixa (pedido do usuário)

Problema: a verificação só acrescentava modelos no fim; a etapa fixa quebrada continuava na Cadeia e o que passava só entrava depois de confirmar o diálogo.
Agora (`descoberta_modelos.py`: `modelo_funciona`, `modelo_nao_funciona`, `usar_efetivo`, `modelos_para_cadeia`; `dock_assistente.py`: `_complementar_pernas_mista`, `_indice_inicial_mista`, `_descrever_cadeia`; `dialog_configuracoes.py`):
- Provedor **nunca verificado**: etapa fixa de sempre (igual à 2.5.2).
- Provedor **verificado**: a etapa fixa é trocada pelos modelos que funcionam, **na mesma posição** (uma etapa por modelo, mesma chave, perfil enxuto em Groq/GitHub), e as etapas são renumeradas.
- **Entram sozinhos** os que passaram nos dois testes (ping + requisição real), sem precisar confirmar o diálogo. **Saem** os `nao_existe` e `sem_ferramentas`. `limite`, `erro`, `grande_demais` e chave inválida **não tiram** ninguém (são passageiros).
- Se **todos** os testados forem `nao_existe`/`sem_ferramentas` (ou o usuário desmarcar tudo), a etapa do provedor **sai** da Cadeia. Se houver dúvida (só limites/erros), a etapa fixa fica.
- A escolha do usuário em ⚙ (campo `confirmado` na gravação) vale sobre a automática; falha passageira não derruba modelo já confirmado; modelo confirmado que passa a não existir sai.
- A etapa inicial escolhida em ⚙ acompanha a montagem (se a etapa saiu, vale a próxima que sobrou). Ao criar a conversa o chat mostra: **"Sistema: Cadeia mista montada: 1. … · 2. …"** (só etapas com chave).
- A Cadeia nova vale na próxima conversa; com OK no ⚙ o plugin reinicia a conversa da Cadeia mista e avisa.
- Testes: 121 no total; o fluxo verificação → lista → OK → Cadeia roda também em Qt real (`tests/dialogo_qt_real_script.py`).
