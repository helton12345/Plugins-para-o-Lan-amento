# Descritivo Técnico-Funcional — Assistente IA Code Runner

**Pasta:** `assistente_ia_code_runner` · **Versão declarada:** 1.21.1 (experimental) · **QGIS mínimo:** 3.28
**Arquivos lidos:** `__init__.py`, `assistente_ia_code_runner.py`, `dock_assistente.py`,
`dialog_configuracoes.py`, `configuracoes.py`, `agente_worker.py`, `provedor_ia.py`,
`ferramentas_ia.py`, `ferramentas_github.py`, `github_cliente.py`,
`conhecimento_precisa.py`, `mcp_servidor.py`, `requirements.txt`, `metadata.txt`

---

## 1. Arquitetura
| Módulo | Responsabilidade |
|---|---|
| `__init__.py` | Recarrega submódulos em ordem a cada `classFactory` |
| `assistente_ia_code_runner.py` | Ação/menu; cria o `QDockWidget`; no `unload` para o MCP e encerra a sessão Claude Code |
| `dock_assistente.py` | Interface do chat, prompt de sistema, `_ExecutorThreadPrincipal` (execução na thread do Qt com `BlockingQueuedConnection`, popups de confirmação, backup zip) |
| `agente_worker.py` | `QThread` do loop agente ↔ ferramentas |
| `provedor_ia.py` | Sessões `SessaoGemini`, `SessaoClaude`, `SessaoGroq`, `SessaoCadeia`, `SessaoClaudeCode`; janela de histórico; resumo/compactação; cotas; auto-instalação de pacotes |
| `ferramentas_ia.py` | Ferramentas QGIS e schema JSON das 13 ferramentas; restrição de pasta |
| `ferramentas_github.py` / `github_cliente.py` | Ferramentas GitHub e cliente REST (Contents/Trees API) |
| `conhecimento_precisa.py` | Resumo estático + busca ao vivo em `helton12345/terraplanagem-` (CLAUDE.md, CATALOGO_FUNCIONALIDADES.md, PROJETOS.md) |
| `mcp_servidor.py` | Servidor MCP HTTP (JSON-RPC 2.0) em 127.0.0.1:7331 |
| `configuracoes.py` / `dialog_configuracoes.py` | `QgsSettings` (grupo `AssistenteIACodeRunner`) e tela ⚙ |

## 2. Funcionalidades implementadas

### 2.1 Ferramentas do agente (13)
| Ferramenta | Thread | Confirmação mecânica |
|---|---|---|
| `executar_codigo_pyqgis` (exec com `iface`, `QgsProject`; retorna `resultado`) | Principal | **Popup Sim/Não** + backup zip + detecção de 10 padrões perigosos |
| `obter_info_projeto` (JSON com arquivo, CRS, camadas, campos, nº de feições) | Principal | — |
| `analisar_arquivos_nao_usados` | Principal | — |
| `limpar_arquivos_nao_usados` (só dentro da pasta do projeto, nunca o .qgs) | Principal | **Popup** + backup zip |
| `consultar_conhecimento_precisa` | Worker | — |
| `listar_arquivos_repositorio`, `buscar_palavra_chave_repositorio` (até 300 arquivos .md/.txt, 8 downloads paralelos, cache por SHA, 15 trechos), `ler_arquivo_repositorio_texto`, `baixar_arquivo_repositorio` | Worker | — |
| `enviar_arquivo_repositorio`, `atualizar_memoria_permanente` | Worker | **Nenhuma** (só instrução no prompt); backup em `_backups/` no repositório |
| `salvar_licao_aprendida` (arquivo novo `licoes/AAAA-MM-DD_HHhMM_slug.md`), `ler_memoria_permanente` (trunca em 32 000 caracteres) | Worker | — |

Backup local: zip da pasta do projeto em `~/AssistenteIA_Workspace/backups_projeto/`
(≤ 500 MB; acima disso copia só o .qgs/.qgz).

### 2.2 Provedores
- **Gemini** (`google-generativeai`): *timeout* de 25 s por chamada; em cota
  espera o tempo sugerido e repete; em cota/lentidão troca para “modelos
  irmãos” da mesma chave, preservando o histórico.
- **Claude** (`anthropic`): *prompt caching* no system, memória e ferramentas;
  retentativa única em `RateLimitError`; cotas lidas dos cabeçalhos.
- **Groq** (HTTP, formato OpenAI): retentativa única em 429; cotas dos cabeçalhos; sem imagem.
- **Cadeia:** pernas Gemini leve → Gemini → Gemini pago; troca só no início de
  uma mensagem, com resumo *handoff*; popup antes da etapa paga; até 3 ciclos
  com espera de 30 s; tenta voltar ao grátis após 30 s na etapa paga; opção de
  travar etapa.
- **Claude Code local** (`claude-agent-sdk`): conexão persistente em *event loop*
  próprio, `allowed_tools = mcp__qgis_smith__*`, `permission_mode="dontAsk"`,
  PATH do Registro do Windows, janela de console oculta, detecção de login expirado; *timeout* 300 s.
- **Janela de histórico:** corte por tamanho (80 000 caracteres), sempre no
  início de mensagem; o trecho cortado vira resumo (até 800 tokens) reinjetado no sistema.
- **Troca manual de provedor:** resumo da sessão anterior anexado à 1ª mensagem.

### 2.3 Servidor MCP
`initialize`, `tools/list`, `tools/call`; notificações → 202; mesmas travas do
chat para ferramentas QGIS; escuta só em 127.0.0.1.

### 2.4 Restrição de pasta de trabalho
Substitui temporariamente `builtins.open`, `os.remove/unlink/rename/replace/rmdir/mkdir/makedirs`,
`shutil.rmtree/move/copy/copy2/copytree` e métodos de `Path`; caminhos
relativos passam a ser relativos à pasta; fora dela → `PermissionError`.

## 3. Normas técnicas
O plugin **não implementa norma técnica**. O texto de conhecimento
(`conhecimento_precisa.py`) repassa à IA regras de negócio dos outros plugins
(ex.: DN mínimo esgoto 200/drenagem 400/água 50, recobrimentos, y/D ≤ 0,75/0,80,
EPSG 31983) e cita a **NBR 9732** como referência do módulo de volumes da
terraplanagem — é informação para a IA, não cálculo deste plugin.

## 4. Dependências
QGIS ≥ 3.28; `requests` (import no topo de `github_cliente.py`); por provedor,
instalados automaticamente via `pip install --user` quando faltam:
`google-generativeai` (descontinuado pelo Google — TODO no código),
`anthropic`, `Pillow`, `claude-agent-sdk`; CLI `claude` para o modo local;
conta/token GitHub para as ferramentas de repositório.

## 5. Limitações conhecidas e pontos de atenção
1. **Servidor MCP sem autenticação nem checagem de origem:** qualquer processo
   local — e, em tese, uma página web que envie um POST “simples”
   (`text/plain`, sem *preflight*) para `127.0.0.1:7331` — pode chamar as
   ferramentas. As de GitHub (`atualizar_memoria_permanente`,
   `enviar_arquivo_repositorio`, `salvar_licao_aprendida`) rodam **sem popup**
   com o token do usuário. Risco enquanto o MCP estiver ligado.
2. **Sem trava mecânica para escrita no GitHub:** `enviar_arquivo_repositorio`
   pode enviar qualquer arquivo local legível para qualquer repositório ao
   alcance do token; a confirmação depende só de o modelo obedecer ao prompt.
3. **Restrição de pasta parcial:** não cobre gravações feitas por QGIS/GDAL/Qt
   (`QgsVectorFileWriter`, `gdal.Warp`, `QFile`), `io.open`, `os.open` etc. O
   próprio código declara que não é *sandbox*. Durante o `exec` a troca é
   global no processo (afeta outras threads Python).
4. **Erro não tratado em ferramentas GitHub quebra o turno:** as ferramentas só
   capturam `ErroGithub`; `requests.HTTPError`/erros de rede (via
   `raise_for_status`) ou argumentos errados (`TypeError`) sobem até o
   `AgenteWorker`, o turno termina em traceback e o histórico fica com uma
   chamada de ferramenta sem resposta — no Claude isso tende a gerar erro 400
   na mensagem seguinte.
5. **🆕 Nova sessão não encerra o Claude Code local:** `self.sessoes = {}`
   descarta a sessão sem chamar `encerrar()` — o processo `claude` e a thread
   do *loop* ficam vivos até o QGIS fechar.
6. **Instalação automática de pacotes** (`pip install --user`) sem perguntar ao usuário.
7. **Chaves/token em texto simples** no perfil do QGIS (QgsSettings).
8. Chat renderizado como HTML sem *escape*: código com `<`/`>` exibido no chat
   pode sumir ou deformar a exibição (o código executado não é afetado).
9. O modo 🚀 Automático também pula a confirmação de **limpeza de arquivos**
   (o *tooltip* cita só scripts).
10. `google-generativeai` está descontinuado (TODO no código); os IDs de
    modelo padrão estão fixos no código e dependem do catálogo de cada provedor.
11. Sem botão de cancelar uma resposta em andamento.
12. Repositório de conhecimento e de memória padrão fixos no código
    (`helton12345/terraplanagem-`, `helton12345/mem-ria-ia-qgis`).
