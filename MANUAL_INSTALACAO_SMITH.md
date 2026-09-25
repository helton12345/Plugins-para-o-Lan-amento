# Comandos de instalação — Smith (passo a passo, sem pular nenhum)

*Atualizado para o plugin Assistente IA Code Runner v1.21.5.*

## Antes de tudo: duas janelas diferentes, não confunda

- **Prompt de Comando** (cmd.exe): a linha começa com algo tipo `C:\Users\...\>`. É AQUI que roda `pip install`, `dir`, os `.bat`.
- **Python interativo**: a linha começa com `>>>`. Isso abre quando você digita só `python` e aperta Enter dentro do cmd. **`pip install` NUNCA funciona aqui** — dá o erro `SyntaxError: invalid syntax`.

Se você ver `>>>` na tela, digite `exit()` e Enter pra sair e voltar pro `C:\...>`.

Pra abrir um Prompt de Comando do zero: aperte `Win + R`, digite `cmd`, Enter.

---

## 1. Instalar as bibliotecas Python do plugin (no Python do QGIS, não no Python comum)

Isso é diferente de abrir o `python` sozinho — tem que ser o Python **de dentro do QGIS**, usando o `.bat` certo. Nesta máquina (QGIS instalado pelo OSGeo4W, na pasta do usuário), o comando é:

```
"C:\Users\Precisa Agrimensura\AppData\Local\Programs\OSGeo4W\bin\python-qgis-ltr.bat" -m pip install -U google-generativeai anthropic requests Pillow claude-agent-sdk
```

Copie e cole essa linha inteira (com as aspas) no Prompt de Comando e aperte Enter.

> **Em outro PC o caminho pode mudar.** QGIS instalado normalmente → `"C:\Program Files\QGIS 3.44.9\bin\python-qgis-ltr.bat"` (troque pela versão instalada); OSGeo4W para todos os usuários → `"C:\OSGeo4W\bin\python-qgis-ltr.bat"`. Se não existir `python-qgis-ltr.bat`, use `python-qgis.bat` na mesma pasta. Isso instala (ou atualiza, por causa do `-U`) as 5 bibliotecas de uma vez:
- `google-generativeai` (Gemini)
- `anthropic` (Claude via API)
- `requests` (Groq)
- `Pillow` (ler imagens anexadas)
- `claude-agent-sdk` (Claude Code local / Smith)

**Importante:** desde a v1.21.4 o Smith usa um recurso mais novo do `claude-agent-sdk` (as ferramentas do QGIS vão por dentro do plugin, sem porta de rede). Se você já tinha instalado antes, rode o comando acima com o `-U` para atualizar.

**Nota**: o plugin tenta instalar essas bibliotecas sozinho quando você usa cada provedor pela primeira vez. Esse comando manual é o **plano B**, pra quando o auto-instalador não conseguir (ele avisa quando falha).

Se aparecer erro de "acesso negado", abra o Prompt de Comando como Administrador (botão direito no ícone → "Executar como administrador") e rode de novo.

---

## 2. Conferir se instalou certo

Ainda no Prompt de Comando:

```
"C:\Users\Precisa Agrimensura\AppData\Local\Programs\OSGeo4W\bin\python-qgis-ltr.bat" -m pip show claude-agent-sdk
```

Se aparecer um bloco com `Name: claude-agent-sdk`, `Version: ...` etc., instalou certo. Se aparecer `WARNING: Package(s) not found`, não instalou — repita o comando do passo 1 e veja se aparece algum erro em vermelho no meio do processo.

Repita esse `pip show` trocando o nome do pacote pra conferir os outros (`google-generativeai`, `anthropic`, `requests`, `Pillow`).

---

## 3. Instalar o Claude Code CLI (se ainda não tiver)

Isso é **diferente** do passo 1 — ali você instalou bibliotecas Python; aqui você instala o programa `claude` em si, que fala com sua assinatura Pro/Max.

Use o **PowerShell** (não o Prompt de Comando): `Win + R` → `powershell` → Enter. A linha começa com `PS C:\...>`. Cole:

```
irm https://claude.ai/install.ps1 | iex
```

O instalador já ajusta a PATH sozinho. O `winget install Anthropic.ClaudeCode` também funciona, mas só se o Windows tiver o "Instalador de Aplicativo" da Microsoft Store — se aparecer "'winget' não é reconhecido", use o comando do PowerShell acima.

Se já tiver instalado, mantenha atualizado (no Prompt de Comando):

```
claude update
```

---

## 4. Confirmar login do Claude Code

Feche o Prompt de Comando e abra um **novo** (importante — PATH só atualiza em terminal novo). Rode:

```
claude --version
```

Se aparecer um número de versão, o comando foi encontrado. Se der erro "não é reconhecido", reinicie o PC (não só o QGIS) e tente de novo — é um problema de cache de PATH do Windows.

Depois rode:

```
claude
```

Isso abre o Claude Code. Dentro dele, digite:

```
/login
```

e siga a autenticação pelo navegador com sua conta Anthropic (Pro/Max). Depois de logado, digite `/exit` pra sair.

---

## 5. Testar no QGIS

1. Feche o QGIS completamente (se estiver aberto) e abra de novo.
2. Abra o painel do Smith, vá em ⚙ e confirme que o provedor é **"Claude Code local"**.
3. **Não precisa marcar a caixa "Servidor MCP".** Desde a v1.21.4 o Smith recebe as ferramentas do QGIS por dentro do plugin. A caixa só serve para conectar um Claude Code **de fora** do QGIS (terminal/app) — deixe **desmarcada** (mais seguro: nenhuma porta aberta no PC).
4. Com um projeto aberto, mande uma mensagem de teste que use o QGIS, por exemplo: *"liste 5 feições da camada X"*.
5. Deve aparecer no chat **"🔌 Claude Code chamou: executar_codigo_pyqgis"** e, em seguida, a resposta com os dados.

Se der erro, copie o texto completo (ou tire um print) — veja também a seção **Problemas comuns** abaixo.

---

## 6. Configurações do Smith (⚙, com provedor "Claude Code local")

| Campo | O que faz |
|---|---|
| **Modelo (Claude Code local)** | Escolhe o modelo. "Padrão" usa o da sua assinatura. Opções incluem `claude-opus-5-5`, `claude-opus-5`, `claude-sonnet-5` e outros. |
| **Esforço (Claude Code local)** | Baixo / Médio / Alto. Alto = pensa mais (melhor para tarefas complexas, mais lento). Médio = bom para tarefas simples. |
| **Aviso "trabalhando" a cada** | De quanto em quanto tempo o chat mostra "⏳ Smith ainda trabalhando… X min" em tarefas longas. 1 a 30 min, ou **Desligado** (0). |

As mudanças valem a partir da próxima **Nova sessão**.

### O que aparece no chat durante uma tarefa

- **💭 Smith: …** — o que ele está fazendo entre um passo e outro ("Vou ler a camada…").
- **🔌 Claude Code chamou: …** — cada ferramenta do QGIS que ele usou.
- **⏳ Smith ainda trabalhando… X min** — aviso de que continua trabalhando (conforme o intervalo configurado).
- A **resposta final** aparece normalmente no fim.

Cada resposta pode durar até **30 minutos** (incluindo o tempo que você leva para clicar "Sim" nos popups de confirmação).

---

## 7. Ao atualizar o plugin

1. QGIS → **Complementos → Gerenciar e instalar complementos → Instalar a partir de ZIP** → escolha o ZIP novo (pode instalar por cima do antigo, sem desinstalar).
2. **Feche o QGIS completamente** e abra de novo. Só desativar/reativar o plugin **não basta** — o QGIS continua com a versão antiga na memória.
3. Confira a versão em **Complementos → Instalados → Assistente IA Code Runner**.

---

## 8. Problemas comuns

| Sintoma | O que fazer |
|---|---|
| O Smith diz que **não encontra as ferramentas** (`executar_codigo_pyqgis` etc.) | Feche o QGIS por completo, abra de novo e clique em **Nova sessão**. Se continuar, atualize as bibliotecas (passo 1 com `-U`) e o Claude Code (`claude update`). |
| **"Claude Code não terminou em 30 min — a resposta foi interrompida"** | A tarefa foi grande demais para uma resposta só. Divida em etapas menores. A próxima mensagem funciona normalmente. |
| Aparece **"🔒 MCP recusou uma chamada…"** | Só acontece com a caixa "Servidor MCP" marcada, quando algum programa de fora tentou usar a porta sem senha. Se você não usa Claude Code fora do QGIS, desmarque a caixa. |
| **"'winget' não é reconhecido"** | Use o instalador do PowerShell do passo 3. |
| **"comando 'claude' não encontrado"** | Refaça o passo 3, abra um Prompt de Comando novo, confira `claude --version` e reinicie o QGIS (ou o PC). |
| **Erro de login / autenticação** | No Prompt de Comando: `claude` → `/login` → `/exit`. Depois, Nova sessão no Smith. |
| Chat mostrando **💭** e **⏳** | Normal — são os passos e o aviso de que o Smith está trabalhando. Para menos avisos, aumente o intervalo ou desligue em ⚙. |

---

## Resumo rápido (só os comandos, sem explicação)

```
"C:\Users\Precisa Agrimensura\AppData\Local\Programs\OSGeo4W\bin\python-qgis-ltr.bat" -m pip install -U google-generativeai anthropic requests Pillow claude-agent-sdk
(PowerShell)  irm https://claude.ai/install.ps1 | iex
claude update
claude --version
claude
/login
```
