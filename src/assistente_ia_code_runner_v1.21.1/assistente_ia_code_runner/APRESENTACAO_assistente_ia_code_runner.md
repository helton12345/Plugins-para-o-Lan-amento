# Smith — o que ele faz

Smith é o assistente de IA que vive dentro do QGIS, num painel de chat lateral. Você conversa com ele em português, e ele age diretamente no seu QGIS — sem precisar sair do programa, copiar e colar código, ou trocar de janela.

## O que ele consegue fazer

- **Processar seu projeto.** Antes de agir, ele confere quais camadas estão carregadas, o CRS de cada uma, os campos e a quantidade de feições — para não trabalhar "no escuro".
- **Rodar tarefas de verdade.** Ele escreve e executa scripts que leem, cruzam e transformam suas camadas para gerar os entregáveis que você pede: shapefile, DXF, relatório, memorial, planilha.
- **Usar os plugins de engenharia da Precisa como ferramentas prontas.** Para terraplanagem, saneamento, drenagem e afins, ele prioriza os plugins já instalados em vez de recalcular fórmulas de engenharia por conta própria — e consulta as regras técnicas documentadas desses temas antes de agir.
- **Aproveitar a lógica de outros plugins instalados.** Se a conta que você precisa já existe dentro de outro plugin do QGIS, ele consegue ler o código e chamar a função diretamente — inclusive preenchendo campos e acionando botões de uma tela que já está aberta, sem precisar mexer no mouse ou no teclado de verdade. O que ele nunca faz é reescrever o código-fonte de outro plugin — isso fica fora do que ele resolve dentro do QGIS.
- **Faxina assistida.** Ele identifica arquivos na pasta do projeto que não estão mais referenciados por nenhuma camada carregada, mostra a lista completa pra você decidir, e só apaga os que você apontar explicitamente — com um backup automático da pasta inteira antes.
- **Trabalhar com repositórios do GitHub.** Qualquer repositório que você citar: ele lista, procura por palavra-chave, lê, baixa e envia arquivos de volta — sempre guardando uma cópia da versão anterior antes de sobrescrever qualquer coisa.
- **Lembrar de você entre uma sessão e outra.** Ele mantém uma memória permanente, guardada num repositório do GitHub, dividida por assunto (suas preferências, convenções do projeto, lições técnicas aprendidas) — assim ele não esquece o que já foi combinado, mesmo depois de fechar o QGIS. Você também pode abrir e editar essa memória direto no GitHub quando quiser.
- **Registrar aprendizado sozinho.** Quando percebe que algo funcionou, não funcionou, ou só deu certo depois de mais de uma tentativa, ele anota isso na memória por conta própria — sem esperar você pedir.
- **Entender um print.** Você pode anexar uma imagem (por exemplo, uma mensagem de erro na tela) para ele analisar junto com sua pergunta.
- **Escrever instruções longas de uma vez.** A caixa de mensagem aceita várias linhas — dá pra listar vários passos numa mensagem só, sem que ela seja enviada pela metade.
- **Mostrar que está trabalhando.** Enquanto processa, aparece um contador de tempo no chat, então você sabe que ele está pensando, e não travado.
- **Trocar de "cérebro" sem perder o fio.** Você escolhe qual inteligência artificial usar por trás (Google Gemini, Anthropic Claude — inclusive pela sua assinatura, sem gastar de API —, Groq, OpenAI, DeepSeek ou xAI/Grok), e pode trocar no meio da conversa: ele repassa um resumo do que estava em andamento para quem assumir.

## Como ele se comporta com você

- **Sempre avisa antes de mudar algo que já existe.** Explica o que vai acontecer e só executa depois da sua confirmação — a não ser que você ligue o modo automático de propósito.
- **Ações que rodam código ou apagam arquivo pedem confirmação num popup**, com um backup automático da pasta do projeto feito antes, por garantia.
- **Não desiste fácil.** Se uma tentativa falha algumas vezes seguidas, ele propõe uma abordagem diferente e pergunta se deve tentar — em vez de simplesmente te devolver a tarefa pra fazer na mão.

## O que ele nunca faz

- Não altera o código-fonte de outros plugins instalados.
- Não simula mouse ou teclado, nem controla outras janelas do seu computador — fica restrito ao que acontece dentro do próprio QGIS e dos repositórios que você autorizar.
- Não apaga, sobrescreve ou envia nada de forma definitiva sem sua confirmação explícita.
