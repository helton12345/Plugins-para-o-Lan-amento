# Manual de uso — GEO RURAL 3.0 (v3.199)

Plugin QGIS para georreferenciamento de imóveis rurais: coleta de dados, arquivos SIGEF, peças técnicas, pacote de submissão, documentos de cartório e divisão/desmembramento/unificação.

> Os passos seguem a ordem das abas. Os avisos ⚠ vêm da leitura do código e estão detalhados no `DESCRITIVO_plugin_geo_rural_3_59.md`.

---

## 0. Antes de começar

1. **Dependências Python** (instale no Python do QGIS, pelo OSGeo4W Shell no Windows):
   - obrigatórias na prática: **python-docx** (documentos Word) e **openpyxl** (planilhas XLSX);
   - para funções específicas:
     - **pdfplumber**: ler PDF de memorial SIGEF;
     - **odfpy**: ODS multiparcela;
     - **pystac-client**: baixar Topodata;
     - **pygeomag**: declinação magnética offline;
     - **selenium**: modo automático de ITR/CCIR.
2. Tenha à mão:
   - os arquivos **TSV** do rastreio GNSS (formato ROVER-GNSS), ou um CSV/TSV com outro layout;
   - a camada de **polígono do imóvel** (e, se houver, a de pontos de limite), em SIRGAS 2000 / UTM Sul.
3. O plugin só calcula corretamente em **SIRGAS 2000 / UTM fusos 18S a 25S (EPSG 31978–31985)**. ⚠ Fusos Norte (ex.: Roraima, Amapá) não são suportados.
4. Salve o projeto QGIS antes: a sessão do plugin é gravada ao lado do projeto.

## 1. Abrir o plugin

Abra pelo menu **Precisa Agrimensura → 🗺️ Georreferenciamento → GEO RURAL 3.0 — Fases 0 a 5** ou pelo ícone da barra.

- Rodapé da janela: **💾 Salvar Sessão**, **📂 Carregar Sessão**, **📜 Versões**, **🆕 Nova Sessão** e **✖ Fechar**.
- Ao fechar (ou teclar ESC), o plugin:
  - pede confirmação;
  - pergunta se deve **gravar a numeração de vértices no banco**. Só confirme quando o trabalho estiver concluído;
  - faz o auto-save da sessão.
- ⚠ Cada clique no menu abre uma janela nova. Mantenha só uma aberta.

---

## 2. Aba ⚙ Configurações (faça uma vez)

Escolha a **pasta do Banco** (pode ser uma pasta sincronizada do Google Drive). Ali ficam três planilhas:

- `numeracao_vertices.ods`: códigos já usados;
- `banco_pessoas.ods`: proprietários e confrontantes;
- `gestao_escritorio.ods`: serviços e financeiro.

⚠ Essas planilhas guardam CPF, RG, filiação e dados de cônjuge **sem criptografia**. Proteja a pasta.

---

## 3. Aba ⓪ Coleta (opcional)

| Sub-aba | O que fazer |
|---|---|
| 🗺️ **Satélite e Relevo** | Marque os provedores (ESRI e Google vêm marcados) e clique em **ADICIONAR CAMADAS**. Para relevo: **BAIXAR Topodata** (exige ter processado a aba ①) e **Gerar Curvas de Nível** (intervalo padrão de 5 m). |
| 🌱 **CAR / SIGEF** | **CAR:** o plugin tenta a API do SICAR; se ela falhar, usa uma pasta local de SHPs. **SIGEF:** informe a pasta com os SHPs do acervo fundiário e o buffer (padrão 100 m). Clique em **CARREGAR**. |
| 📄 **ITR / CCIR** | Informe CPF/CNPJ (ou o NIRF). O **Modo A** abre o navegador; o **Modo B** usa Selenium. |
| 📡 **RTKLIB / RBMC** | Aponte o `.obs` e clique em **LER TIMESTAMPS** (datas e efeméride saem automáticas). Opcionalmente, converta o bruto com o convbin do RTKLIB. Escolha a estação (**Sugerir 5 mais próximas**) e clique em **BAIXAR KIT PPK COMPLETO**. ⚠ Nesta versão, o kit baixa o RINEX da estação, mas **falha nas efemérides e no IONEX**. Baixe-os à mão. |

Os arquivos vão para `<pasta de saída>/0_coleta/…`.

---

## 4. Aba ① Entrada

Há duas rotas.

**Rota A — TSV do GNSS (recomendada)**

1. Clique em **+ Adicionar TSV**.
   - Para arquivos com outro layout, marque antes **📋 Formatar entrada de dados**. Abre-se o mapeador de colunas: Norte e Este são obrigatórios, e a primeira linha precisa ser o cabeçalho.
   - ⚠ Confira o mapeamento sugerido: colunas "lat/lon" são sugeridas como Norte/Este, e colunas com "h" como altitude.
2. Em **Shapefiles**, escolha o **Polígono do imóvel** e, se houver, os **Pontos de limite**.
3. Informe:
   - a **Zona UTM** (padrão 23S);
   - o **Código do credenciado INCRA** (ex.: VQUE);
   - se quiser, **Marcos M** e **Virtuais V** pelo número do vértice (ex.: `3,5,7`).
4. **Próxima numeração** mostra os próximos códigos P, M e V do banco. Com **✏️ Forçar**, você define o número inicial.
5. Se o imóvel confronta com uma parcela **já certificada**, marque a opção correspondente e clique em **Definir confrontantes certificados…**. Para cada confrontante, informe:
   - a camada de pontos SIGEF;
   - o polígono (opcional);
   - o CSV SIGEF, que fornece a altitude;
   - a tolerância (padrão 0,50 m).
   ⚠ Vértices do seu polígono sem par no TSV nem no certificado são **descartados**.
6. Defina a **Pasta de Saída** e clique em **🔄 PROCESSAR DADOS DE ENTRADA**. O log mostra:
   - o casamento dos vértices (vértice sem TSV aparece como "❌ sem TSV");
   - a área e o perímetro;
   - o resumo de inconsistências.

**Rota B — só o polígono (sem GNSS)**

1. Em **⚡ Gerar vértices a partir do polígono**, escolha o SHP ou a camada.
2. Informe o **DP planimétrico** e o **Método**.
3. Clique em **🔧 Gerar vértices VQUE**.

⚠ Todos os vértices recebem o mesmo desvio-padrão e o mesmo método (o combo começa em **PG1**). Isso **não é dado de campo**. Revise antes de gerar qualquer peça para o SIGEF.

---

## 5. Aba ② Vértices

A tabela é editável com clique duplo. As colunas são:

- Código e Tipo;
- N, E e h;
- σN, σE e σh;
- Método;
- **Tipo Limite** (LA1–LA7, LN1–LN6);
- Arquivo e Nome Campo.

As ações disponíveis são:

- **↻ Renumerar:** renumera os vértices selecionados (ou todos). Vértices "CERTIFICADO" não mudam.
- **◌ Virtual (V)**, **● Ponto (P)** e **🏴 Marco (M)** reclassificam os vértices selecionados.
  - ⚠ Ao converter **vários** vértices para M de uma vez, todos podem receber o **mesmo código**. Converta um de cada vez e confira.
- **🗑 Apagar:** não renumera, então sobram "buracos" na sequência.
- **➕ Inserir manual:** N, E, h, σ, método, tipo e posição.
- **Tipo de limite:** selecione as linhas e use **Ctrl+D** para repetir o valor. O botão direito abre copiar/colar, preencher e "Detectar segmentos".
- O botão direito também preenche o **Método** em lote.

⚠ A coluna "Tipo" mostra só M ou P. Confira o V pelo código.

---

## 6. Aba ③ Confrontantes

1. Para cada trecho, clique em **+ Adicionar confrontante** e preencha:
   - Nome, **Tipo** (pf, pj, poder_publico, estrada, curso_agua, ferrovia, terra_indigena, assentamento, sigef_certificado, espolio, condominio, nao_localizado);
   - CPF/CNPJ, Matrícula e CNS;
   - **Vért. Início** e **Vért. Fim**.
   - ⚠ Se você digitar um código que não existe, o trecho vai para o **vértice 1** sem aviso.
2. Outras formas de preencher:
   - **📥 Carregar de PDF SIGEF** (só a 1ª parcela do PDF);
   - **🔍 Buscar no banco**;
   - a caixa "NOME | CPF", que completa CPFs faltantes.
3. **👥 Proprietários (vários donos)** serve para condomínio, espólio, PJ ou herdeiros. Marque quem assina e use "O titular NÃO assina" quando for o caso.
   - ⚠ Esses dados **não voltam** ao carregar a sessão. Refaça após carregar.

---

## 7. Aba ④ Imóvel & RT

Preencha:

- **Denominação**, **Matrícula(s)** (tabela), **Município**, **UF** (padrão MG), **Comarca**, **Cartório (CNS)**, **Data** e **ART/RRT**;
- **Código INCRA (SNCR)**;
- a natureza do serviço, a situação do imóvel e a natureza da área;
- a opção "Não é a primeira certificação", que cita a dispensa de anuência do art. 176 §13;
- o **Tipo de serviço** que sai na planta.

**Proprietários:** clique em **+ Adicionar** para abrir a qualificação completa (RG, filiação, cônjuge etc.). Cada proprietário é salvo automaticamente no banco de pessoas.

**Responsável Técnico:** ⚠ o formulário vem **preenchido com os dados do autor do plugin**, incluindo RG e CPF. Troque todos os campos pelos seus. Ao carregar uma sessão, **RG e CPF voltam ao padrão do autor**: confira sempre.

---

## 8. Aba ⑤ Validação NTGIR

Clique em **🔍 EXECUTAR VALIDAÇÃO NTGIR** e corrija os erros bloqueantes:

- campos obrigatórios;
- método;
- precisão;
- CNS;
- cobertura dos confrontantes.

⚠ O validador compara também o **σh (altura)** com o limite de posição. Um vértice bom na horizontal pode aparecer como reprovado.

---

## 9. Aba 🅰 Fase 1 — SIGEF

1. Marque o que gerar:
   - XLSX de conferência;
   - **ODS SIGEF** (template oficial);
   - Shapefiles;
   - Relatório de inconsistências.
   - ⚠ A opção de **XML SIGEF não aparece na tela** e o XML não é gerado.
2. Use **📍 GERAR APENAS VÉRTICES** para ver os códigos no mapa antes de preencher os confrontantes.
3. Clique em **⚙ GERAR ARQUIVOS SIGEF**. Os shapes vão para `02_Shapes` e os documentos para `01_Documentos`, com nome datado e versionado.

**Multiparcela:** marque **Ativar modo multiparcela** e depois:

1. **CARREGAR SHP MULTIPARCELA** (1 polígono por parcela) ou **Adicionar parcela atual**;
2. escolha o formato (Lat/Lon e/ou UTM);
3. clique em **EXPORTAR DOCUMENTAÇÃO MULTIPARCELA**.

⚠ Esse fluxo preenche as altitudes pelo SRTM automaticamente (ver item 10).

**Importar do SIGEF:**

- **ODS** (recomendado): escolha a **zona UTM**, que é obrigatória;
- **PDF** (sem sigmas);
- **CSV** (`_vertice` + `_parcela`).

Para cada fonte, há **GERAR SHAPEFILES** e **CARREGAR … COMO TRABALHO**.

⚠ Ao carregar como trabalho:

- os dados do imóvel **não** preenchem a aba ④ (preencha à mão);
- a zona do arquivo **não** altera o combo de zona da aba ①. Ajuste-o.

---

## 10. Aba 🅱 Fase 2 — Peças Técnicas

1. Marque as peças:
   - **Memorial**: padrão Lat/Lon DMS, com opção UTM; o perímetro é geodésico, com a opção "arredondado";
   - Planta A3;
   - Relatório de Levantamento;
   - Relatório de Precisão;
   - Planilha de vértices;
   - Caderneta de Campo;
   - Ficha ART/RRT;
   - Memorial a partir de PDF SIGEF, que é opcional.
2. **Override SIGEF:** digite o perímetro e a área do SIGEF para que o cabeçalho do memorial e da planta bata com o certificado. O plugin avisa quando o valor difere do calculado.
3. **Altitudes:**
   - **PREENCHER ALTITUDES SRTM** (online → GeoTIFF → interpolação);
   - **Interpolar selecionados**;
   - **Altitude manual**, que fica protegida.
   - ⚠ SRTM é altitude **ortométrica** e o plugin a grava como **elipsoidal**, sem correção do geoide (diferença de metros). Use só como último recurso e revise.
4. Preencha os **Equipamentos**. Os padrões são "Trimble R8s / R10", "RTK … / PPP" e "RTKLIB / TBC": troque pelos reais.
5. Clique em **⚙ GERAR PEÇAS TÉCNICAS**.
   - A **planta** abre no editor de layout do QGIS. Revise e **exporte o PDF à mão**.
   - ⚠ A caderneta sai com "Céu limpo, ventos fracos, PDOP<4" fixo. Edite.

Outros botões:

- **GERAR MEMORIAL AVULSO:** a partir de uma camada de pontos já codificada, sem renumerar;
- **COTAS GEODÉSICAS NA PLANTA:** gera um GPKG em `<projeto>/cotas`.

---

## 11. Aba 🅲 Fase 3 — Submissão

- Use **Verificar checklist** e depois **PREPARAR PACOTE (ZIP)**. O histórico fica em `sigef_submissoes.jsonl`.
- ⚠ O pacote procura arquivos com nomes antigos na raiz da pasta, enquanto as fases 1 e 2 gravam em subpastas com nome datado. O ZIP tende a sair incompleto: confira a lista de "faltando".
- ⚠ **Não há envio automático ao SIGEF.** Envie pelo site.

---

## 12. Aba 🅳 Fase 4 — Cartório

1. Escolha a **UF do cartório**.
   - **MG** usa os modelos do Provimento Conjunto 93/2020.
   - As demais usam modelos genéricos. Revise as citações legais.
2. Sub-abas:
   - **📜 Anuências:** **GERAR TODAS AS ANUÊNCIAS** (uma por confrontante) e **DECLARAÇÃO DE CIÊNCIA DE DIVISAS**;
   - **📋 Requerimentos:**
     - **Requerimento ao CRI** (só MG): nº do ofício, valor e ato;
     - Certificação INCRA, Inexistência de Edificações, Ausência de Sobreposição e Minuta de Averbação (marcados);
     - Cancelamento SIGEF (só MG) e Credenciamento RT;
   - **✅ Checklist.**
3. ⚠ **Revise todo documento antes de assinar.** Vários modelos:
   - trazem **nome, RG e CPF do autor do plugin** como testemunha ou RT;
   - afirmam fatos que o programa não verifica, por exemplo:
     - "não foi constatada sobreposição";
     - "anuências de todos os confrontantes obtidas";
     - "não sou PEP", em alguns modelos já marcado.

---

## 13. Aba 🔪 Fase 5 — Divisão / Unificação

**Divisão ou desmembramento**

1. Escolha **DESMEMBRAMENTO** ou **DIVISÃO**.
2. Preencha as denominações, a matrícula original e o destinatário.
3. **📥 Importar parcelas prontas:** um SHP com um polígono por parcela, cujos vértices de divisa coincidem exatamente com o limite da Fase 1.
4. Revise a ordem e os nomes na tabela (clique duplo e ▲▼). Clique em **GERAR POLÍGONOS E VÉRTICES**, que grava em `02_Shapes`.
5. Clique em **GERAR DOCUMENTAÇÃO**. Saem:
   - um memorial por parcela;
   - o memorial da área total;
   - o ODS, a planilha e as anuências;
   - o requerimento e o checklist.

⚠ Marcos novos de divisa saem como **M / PG6 com σ = 0**. Loque em campo e preencha as precisões antes de enviar ao SIGEF. ⚠ Há um limite de 10 parcelas.

**Unificação**

1. Adicione as parcelas: **↻ Do Plugin**, **➕ SHP/JSON** ou **🎯 Feature Selecionada**.
   - ⚠ O SHP é lido **sem reprojeção**: ele deve estar no mesmo EPSG do trabalho.
2. Preencha a denominação e a matrícula principal.
3. Clique em **VERIFICAR ADJACÊNCIA** e depois em **GERAR DOCUMENTAÇÃO**.

⚠ Os códigos do imóvel unificado saem com a credencial **VQUE** fixa. Ajuste se a sua for outra.

**Polígonos internos** (demonstrativo): escolha o prefixo e a tolerância e use:

1. **Renumerar internos**;
2. **Gerar memoriais + shapes internos**.

**📁 Produtos Gerados:** lista os arquivos, que abrem com clique duplo.

---

## 14. Aba 🏢 Gestão

Oferece:

- o banco de dados: abrir, fazer backup e abrir a numeração;
- o dashboard;
- o registro de serviço (processo, fase, status, prazo e valor).

⚠ **Mudar pasta** não muda o caminho de fato (o banco continua no local anterior).

⚠ A tabela "Parcelas do Serviço Atual" conflita com a tabela da Fase 5. Não confie nas parcelas financeiras desta versão.

---

## 15. Sessão

- **Salvar Sessão** grava `<projeto>_georref_rural.json` e mantém 5 versões, além de uma cópia em `03_Sessao`.
- Use **📜 Versões** para restaurar, inclusive o auto-backup.
- ⚠ Ao carregar uma sessão, **não voltam**:
  - o RG e o CPF do RT;
  - os proprietários múltiplos e os representantes dos confrontantes;
  - os vértices virtuais (campo "V");
  - o mapeamento de colunas.
- ⚠ Depois do primeiro auto-save, novos fechamentos só salvam se você clicar em **Salvar Sessão**. Salve manualmente sempre.
