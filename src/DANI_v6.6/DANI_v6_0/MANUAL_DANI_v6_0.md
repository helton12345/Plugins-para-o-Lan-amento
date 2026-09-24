# Manual de uso — DANI (Memorial Descritivo) v6.6

Plugin QGIS para gerar memoriais descritivos de lotes urbanos (GEO, TABULAR e PLANILHA), plantas, tabelas de áreas, planilha ABNT NBR 12721, cotas, declividade e shapefile ONR.

> Os passos seguem a ordem das telas do plugin. Os avisos marcados com ⚠ vêm da leitura do código e estão detalhados no `DESCRITIVO_DANI_v6_0.md`.

---

## 0. Antes de começar

1. **Salve o projeto QGIS** (.qgz/.qgs). A sessão do DANI e os backups ficam na pasta do projeto. Sem projeto salvo, ficam em `~/dani_sessao*`.
2. Tenha no projeto:
   - **LOTES:** camada de polígonos em SRC projetado (UTM/SIRGAS 2000), com os campos de quadra e de número do lote. Todas as medidas são calculadas no plano da projeção.
   - **RUAS:** camada de polígonos com um campo de nome da via.
   - Opcionais:
     - camada de confrontantes externos (polígonos com um campo de nome);
     - curvas de nível com campo de cota (usadas na declividade).
3. Dependências Python no QGIS:
   - **python-docx** e **openpyxl**, para os arquivos Word e Excel;
   - **matplotlib**, para as plantas técnicas;
   - opcional: **pygeomag**, para a declinação magnética (sem ele, o plugin tenta o serviço NOAA pela internet; se falhar, o carimbo sai "N/D").
4. Uma **cópia de segurança da camada de LOTES** é recomendada. ⚠ O plugin grava nela os campos AREA, PERIMETRO, AREA_CALC, DANI_* e DECLIVIDADE.
5. O botão **📋 Guia Shapefile** abre o `README_SHAPEFILE.md`, que explica como preparar as camadas.

## 1. Abrir o plugin

Abra pelo menu **DANI - Memorial Descritivo → DANI - Gerar Memorial Descritivo**, pelo ícone da barra ou pelo menu da suíte (📁 Projetos).

A janela tem três abas: **① Projeto**, **② Gerar Memorial** e **③ Ferramentas**. No rodapé ficam os botões Salvar Sessão, Carregar Sessão, Guia Shapefile e Fechar.

---

## 2. Aba ① Projeto

Faça a configuração nesta ordem.

### 2.1 🗂 Configurar Camadas e Colunas

Esta tela tem três abas.

- **🏗 Lotes**
  - Escolha a camada e os campos:
    - Quadra;
    - número do lote (imóvel);
    - ID do lote;
    - Proprietário, CPF/CNPJ, Endereço, Cidade e Comarca;
    - UF (obrigatória).
  - No grupo **Área, Perímetro e Declividade**:
    - **Usar Área da tabela de atributos:** com esta opção, o memorial usa o valor do campo em vez da área calculada, mas só quando o campo for > 0. Se ele divergir mais de 5 % da geometria, o plugin gera uma observação.
    - **Usar Declividade da tabela de atributos.**
  - No grupo **Saída e Identificação do Loteamento**, informe o nome do loteamento e a pasta de saída (📁 Escolher…). Sem pasta, a saída vai para a Área de Trabalho.
- **🛣 Ruas:** a camada e o campo de nome da via.
- **🏘 Confrontantes Ext.:** opcional. A camada e o campo de nome.

Clique em OK para salvar.

> ⚠ **Importante:** salvar esta tela **apaga** o que foi preenchido em "Dados do Loteamento" (item 2.2) e o campo de área, se a opção "Usar Área da tabela" estiver desmarcada. Sempre que reabrir esta tela, preencha o item 2.2 de novo.

### 2.2 🏘 Dados do Loteamento / Proprietário

Preencha:

- proprietário, CPF/CNPJ, endereço, município, comarca e matrícula;
- o bloco do **Responsável Técnico**: nome, título e CREA;
- a **posição da assinatura** nos documentos Word: ao final de cada lote, ou só no final do documento.

Estes dados substituem os do cabeçalho nos arquivos .docx.

### 2.3 📝 Linhas Customizadas do Memorial

Para adicionar linhas extras ao cabeçalho ou ao rodapé, clique em **+ Adicionar linha** e informe o rótulo, o campo e a posição.

### 2.4 ⚙ Configurações de IA

Esta configuração é opcional.

- **Claude:** informe a chave da API e use **Testar**.
  - "Usar Claude quando disponível";
  - "Modo adaptativo".
- **Gemini:** chave da API, **Testar** e "Usar Gemini quando disponível".
- **Base de Aprendizado Local:** a pasta onde as correções ficam registradas.
- **Visualização:** as opções de preview.

> ⚠ As chaves ficam gravadas em texto puro nas configurações do QGIS. Ao usar a IA, o texto do memorial é enviado ao serviço externo.

---

## 3. Aba ② Gerar Memorial

1. Escolha o tipo:
   1. **Memorial GEO (georreferenciado):** descrição vértice a vértice, com azimutes, distâncias e coordenadas N/E.
   2. **Memorial TABULAR:** frente, fundo, lateral direita e lateral esquerda de cada lote.
   3. **Memorial PLANILHA:** tabela de vértices com texto narrativo, em XLSX.
   4. **Plantas técnicas individuais:** A4 retrato, sem carimbo, em PDF e SVG.
   5. **★ GERAR TODOS:** os três memoriais e as plantas técnicas.
2. Escolha o escopo: **Todos os lotes**, **Quadra inteira** ou **Lote específico** (com a quadra e o lote).
   - ⚠ Um lote com letra (ex.: "5 B") pode falhar no modo "Lote específico". Nesse caso, use "Quadra inteira".
3. Opções:
   - **Abrir preview e edição antes de salvar:** mantém o TXT e abre o editor (ver item 5).
   - **Incluir Declividade nos memoriais:** vem desmarcado por padrão.
   - **Confrontantes:**
     - *Automático*: nunca pergunta. Com vários candidatos, usa o primeiro em ordem alfabética; sem candidato, escreve "CONFERIR EM PLANTA".
     - *Perguntar só quando não identificar.*
     - *Perguntar sempre que houver dúvida:* é o modo recomendado para um loteamento novo.
   - **🔇 Silencioso:** não abre as perguntas de lote de esquina nem a validação de lados, e registra "VERIFICAR antes de assinar" nas observações.
4. Clique em **▶ Gerar Memorial(is)**.

A janela do DANI se esconde durante a execução. Podem surgir perguntas, com o trecho destacado no mapa:

- **Confrontante:** escolha da lista ou use [DIGITAR MANUALMENTE].
- **Lote de esquina:** qual rua é a frente. O padrão sugerido é a rua com a menor testada.
- **Lote com 2 frentes:** qual rua é a principal. A outra vira fundo.
- **Validar lados:** a distribuição proposta de frente, laterais e fundo é destacada no mapa (azul = direita, laranja = esquerda, roxo = fundo). Com **Não**, o plugin pede a classificação de cada segmento (Frente, Lateral Direita, Lateral Esquerda ou Fundo).

### Saídas (na pasta configurada ou na Área de Trabalho)

- **GEO e TABULAR:** Word (.docx). O TXT só é mantido com o preview ligado.
- **PLANILHA:** XLSX.
- **Relatório de observações:** XLSX separado, com os avisos de divergência, curvas estimadas e casos especiais.
- **Plantas técnicas:** PDF (um arquivo por lote ou por quadra) e um SVG editável por lote.
- Nos .docx, as linhas com "CONFERIR EM PLANTA" ou "lote triangular" aparecem **em amarelo**. Revise-as antes de assinar.

> ⚠ Ao gerar os memoriais, o plugin **grava na camada de LOTES** os campos AREA (substituída pela área calculada, a menos que "Usar Área da tabela" esteja ativo), PERIMETRO e AREA_CALC, e salva a camada (commit).

---

## 4. Aba ③ Ferramentas (Funcionalidades)

| Aba | O que fazer |
|---|---|
| 🗺 **Planta Geral** | Escolha as camadas, o template (o padrão é `templates/MODELO_GEO.qpt`), a rosa dos ventos, o logo, o tamanho da folha e a escala (automática ou fixa), as cotas e a pasta de saída. Depois, **▶ Gerar Planta Geral (QPT)**. O croqui de satélite (ESRI) exige internet; sem ela, o croqui é feito localmente. |
| 📄 **Planta do Lote** | Escolha as camadas, o que gerar (**PDF**, **SVG editável**, **Croqui de localização**), a folha (A4 ou A3), a camada de cotas (vértices V1…, raio nas curvas) e a pasta. Depois, **▶ Gerar Plantas dos Lotes**. O carimbo descreve o lote (título, área, perímetro). |
| 📐 **Declividade** | Escolha os lotes, as ruas, as curvas de nível e o campo de cota. Depois, **▶ Calcular Declividade de Todos os Lotes**. O resultado (em %) é gravado na camada de lotes. |
| 📏 **Cotas Automáticas** | Escolha a camada e as opções (comprimento, azimute, área e perímetro no centróide). Depois, **▶ Gerar Cotas Automáticas**, que cria uma camada de cotas. |
| 📊 **Tabelas de Áreas** | Escolha as camadas, os campos e a saída. Depois, **▶ Gerar Tabelas de Áreas**. Sai um texto na tela e um XLSX com fórmulas. ⚠ Confira o "TOTAL GERAL" do XLSX (ver Descritivo). |
| 📋 **ABNT 12721** | Escolha as camadas, as áreas de uso comum, os arquivos e as opções. Depois, **▶ Gerar Planilha ABNT NBR 12721**. ⚠ Revise as células em amarelo (ajuste de arredondamento) e o nome do empreendimento. |
| 🏛 **ONR** | Escolha a camada, os campos de origem e de ordenação, e o shapefile de saída. Depois, **▶ Gerar Shapefile ONR**. A camada de entrada não é alterada. |

---

## 5. Preview e edição (quando ativado)

A janela abre depois da geração, com o texto do memorial de cada lote.

- **◀ Anterior / Próximo ▶** navega entre os lotes. **🔍 Zoom no Lote** centraliza o lote no mapa.
- Edite o texto direto no editor.
- **¶ Unir em parágrafo único** junta a descrição em um só parágrafo.
- Recursos de IA:
  - **Revisar com Claude**;
  - **Fazer pergunta ao Claude**;
  - **Registrar correção na base IA**, que salva as diferenças na base local.
- **💾 Salvar este lote** ou **💾 Salvar todos**. Sem salvar, as edições se perdem.

> ⚠ A mensagem "Lotes vizinhos atualizados automaticamente" não garante que o texto do vizinho foi corrigido. Confira os lotes vizinhos à mão.

---

## 6. Sessão

- A configuração fica em memória enquanto o QGIS estiver aberto.
- **💾 Salvar Sessão** grava um backup numerado (até 5 versões) em `dani_backups/`, na pasta do projeto.
- **📂 Carregar Sessão** lista o autosave e os backups.
- ⚠ O autosave ao fechar só grava na **primeira** vez em cada sessão do QGIS. Use **Salvar Sessão** antes de fechar o QGIS.

## 7. Dicas rápidas

- Um loteamento com curvas digitalizadas sem densificação pode gerar lados extras. O GEO detecta curvas "suaves" a partir da v6.5.
- O TABULAR, a PLANILHA e o GEO podem numerar vértices e curvas de formas diferentes (usam detectores diferentes). O perímetro da PLANILHA e do TABULAR é copiado do GEO quando o GEO roda na mesma execução.
- Sempre confira, antes de assinar:
  - as linhas amarelas no .docx;
  - o XLSX de observações;
  - o fuso e o datum no rodapé do GEO.
