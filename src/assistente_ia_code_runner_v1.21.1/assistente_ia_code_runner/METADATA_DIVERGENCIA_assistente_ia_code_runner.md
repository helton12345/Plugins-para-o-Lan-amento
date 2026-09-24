# Divergências do metadata.txt — Assistente IA Code Runner

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `description` / `about` (provedores) | “Google Gemini **ou** Anthropic Claude” | Também suporta **Groq**, **Cadeia** (3 etapas Gemini com etapa paga sob confirmação) e **Claude Code local** (assinatura, via `claude-agent-sdk` + servidor MCP) | Média |
| 2 | `about` (funcionalidades) | Chat, `obter_info_projeto`, `executar_codigo_pyqgis`, GitHub, memória | Omite: análise/limpeza de arquivos não usados, backup zip automático, popup de confirmação, modo automático, pasta de trabalho, **servidor MCP local (porta 7331)**, `consultar_conhecimento_precisa`, anexos de imagem, indicador de cota | Média |
| 3 | `about` (segurança) | “Nenhuma chave ou token fica gravado no código-fonte” | Correto, mas chaves e token ficam em **texto simples** no QgsSettings; o metadata não menciona | Baixa |
| 4 | Dependências | Não declaradas | `requests` (obrigatório no carregamento), `google-generativeai`, `anthropic`, `Pillow`, `claude-agent-sdk` (instalados automaticamente com `pip --user`); CLI `claude`. Há `requirements.txt` | Média |
| 5 | `tracker` / `repository` | `https://github.com/` | *Placeholder* | Baixa |
| 6 | `author` / `email` | “Helton” / e-mail pessoal | Sem chaves `precisa_*` — fora do menu da suíte | Informativo |
| 7 | `category` | Plugins | Sem categoria específica — aceitável | — |
| 8 | `version` | 1.21.1 | `provedor_ia.py` 23/09 22:16, `metadata.txt` 23/09 22:17 — consistente | — |
| 9 | `qgisMinimumVersion` | 3.28 | Nenhuma incompatibilidade encontrada | — |
| 10 | `experimental` | True | Coerente com os riscos listados no Descritivo | — |

**Resumo:** metadata **com divergências** — descrição desatualizada em relação
aos provedores e às ferramentas (principalmente Groq, Cadeia, Claude Code e o
servidor MCP local, que tem impacto de segurança) e dependências não declaradas.
