# Manual de Instruções — Assistente IA Code Runner (“Smith”)

**Pasta:** `assistente_ia_code_runner` · versão declarada 1.21.1 (experimental) · QGIS 3.28+

---

## 1. Para que serve

Um **chat lateral dentro do QGIS** com um agente de IA (“Smith”) que:
- lê o projeto aberto (camadas, sistemas de coordenadas, campos);
- **escreve e executa scripts PyQGIS** para você (com confirmação);
- aponta e apaga arquivos que não estão mais em uso na pasta do projeto (com confirmação);
- consulta as regras e os módulos dos plugins de engenharia da Precisa;
- trabalha em repositórios do GitHub: lista, busca, lê, baixa e envia arquivos
  (com backup automático da versão anterior);
- mantém uma **memória própria** no GitHub (lições datadas e um `MEMORIA.md`).

## 2. Preparação

### 2.1 Escolher o “cérebro” (⚙ Configurações)
| Provedor | O que precisa |
|---|---|
| **Google Gemini** | Chave da API do Google AI Studio + modelo |
| **Anthropic Claude** | Chave da API da Anthropic + modelo |
| **Groq (grátis)** | Chave da Groq + modelo (não lê imagens) |
| **Cadeia** | Até 3 chaves Gemini: Flash-Lite grátis → Flash grátis → Pro **pago**. Troca sozinha quando uma etapa fica sem cota ou lenta; **pergunta antes** de usar a etapa paga |
| **Claude Code local** | O programa `claude` instalado e logado no PC (assinatura Pro/Max) — sem chave. Liga sozinho o servidor MCP |

Na Cadeia, o campo **Etapa da cadeia** escolhe qual etapa você está
configurando; **Travar modelo manualmente** faz usar só essa etapa, sem trocar.
No Claude Code local dá para escolher **modelo** e **esforço** (baixo/médio/alto).

As bibliotecas Python de cada provedor (`google-generativeai`, `anthropic`,
`requests`, `Pillow`, `claude-agent-sdk`) são **instaladas automaticamente**
na primeira vez, se faltarem.

### 2.2 Outras opções em ⚙
- **Pasta de trabalho:** se preenchida, os scripts da IA só podem abrir,
  salvar ou apagar arquivos dentro dela (ver cuidados no item 6).
- **Enviar automaticamente um resumo do projeto a cada mensagem** (padrão: ligado).
- **Token do GitHub** (fine-grained, *Contents Read/Write* nos repositórios
  permitidos) e **Repositório de memória** (padrão `helton12345/mem-ria-ia-qgis`).

## 3. Como abrir
Ícone **Assistente IA (Code Runner)** na barra de ferramentas ou no menu
Complementos. O painel abre à direita.

## 4. Usando o chat
1. Digite o pedido no campo **“Comande o QGIS…”** e tecle Enter ou clique em **Executar**.
2. **📎** anexa uma imagem (print, foto). Clique no nome do anexo para removê-lo.
3. O Smith responde no chat. Quando for rodar um script, o código aparece no
   chat e abre uma janela **“Smith quer rodar um script no QGIS”**:
   - clique em **Show Details…** para ver o código;
   - se o código tiver algo perigoso (apagar arquivo/pasta, comando do
     sistema, DROP/DELETE em banco), aparece um aviso em vermelho;
   - **Yes** executa; **No** cancela.
   Antes de rodar, a pasta inteira do projeto (se salvo e até 500 MB) é
   zipada em `~/AssistenteIA_Workspace/backups_projeto/`.
4. Para limpeza: peça “quais arquivos não estão em uso?”; o Smith lista,
   você diz quais apagar, e aparece a janela de confirmação com a lista.
5. Botões:
   - **💾** pede ao Smith para resumir a sessão e salvar como lição no GitHub;
   - **🆕** apaga o histórico da conversa (economiza tokens);
   - **⚙** configurações.
6. **🚀 Automático:** quando ligado, **scripts e limpezas rodam sem perguntar**.
   Volta desligado a cada reinício do QGIS. Use só se souber o que está fazendo.
7. **🔌 Servidor MCP:** liga um servidor local (porta 7331) para o Claude Code
   instalado no PC usar as mesmas ferramentas deste chat.

Mensagens em itálico no chat informam trocas de modelo/etapa (Cadeia/Gemini)
e, para Claude e Groq, os tokens/requisições restantes.

## 5. GitHub e memória
- “Liste o repositório dono/repo”, “procure *palavra* em dono/repo”, “leia o
  arquivo X”, “baixe o shapefile Y” (vai para `~/AssistenteIA_Workspace/`).
- Para **enviar** um arquivo ou **alterar o MEMORIA.md**, o Smith deve
  explicar a mudança e esperar você confirmar no chat. Não há janela de
  confirmação para essas ações. A versão anterior vai para `_backups/` no repositório.
- Lições (“salvar lição”) viram arquivos novos datados em `licoes/`.

## 6. Cuidados
- A conversa **some ao fechar o QGIS**; o que importa deve ir para lição/MEMORIA.md.
- Chaves e token ficam salvos **em texto simples** nas configurações do QGIS.
- A “pasta de trabalho” só barra as funções de arquivo do Python
  (`open`, `os`, `shutil`, `Path`); gravações feitas por funções do
  QGIS/GDAL no script **não são barradas**.
- Com o **Servidor MCP ligado**, qualquer programa do próprio PC pode chamar as
  ferramentas (as de GitHub rodam sem janela de confirmação). Desligue quando não usar.
- O Smith recusa por regra alterar o **código-fonte** dos plugins — isso é
  tarefa de sessão de desenvolvimento.
