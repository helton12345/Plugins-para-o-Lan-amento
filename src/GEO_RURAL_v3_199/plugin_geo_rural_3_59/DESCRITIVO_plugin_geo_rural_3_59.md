# Descritivo técnico — GEO RURAL 3.0 (v3.199)

Este documento descreve o que o código faz de fato. Cada ponto traz uma marca:

- **[CONFIRMADO POR TESTE]**: comprovado executando o código;
- **[CONFIRMADO leitura]**: comprovado lendo o código;
- **[PROVÁVEL]**: indício forte, não executado;
- **[A CONFIRMAR]**: depende de norma ou dado externo.

**Escopo lido:** todo o código da pasta `plugin_geo_rural_3_59/` (cerca de 55 mil linhas, incluindo `core/`, `fase0`–`fase5`, `ui/`, `tests/`, templates e fixtures). O ambiente QGIS **não** foi executado. Testes executados: `tests/run_all.py`, `fase5_divisao/test_divisao.py` e `fase5_divisao/test_sprint2.py`, em Python puro com as dependências instaladas à parte.

---

## 1. Visão geral

- A janela única (`ui/main_dialog*.py`) tem 13 abas visíveis:
  - ⓪ Coleta;
  - ① Entrada, ② Vértices, ③ Confrontantes, ④ Imóvel & RT;
  - ⑤ Validação;
  - 🅰 Fase 1 SIGEF, 🅱 Fase 2 Peças, 🅲 Fase 3 Submissão, 🅳 Fase 4 Cartório;
  - 🏢 Gestão, ⚙ Configurações;
  - 🔪 Fase 5.
- A aba 🧠 Assistente IA existe no código, mas está **oculta** desde a v3.94.
- Há também uma interface de linha de comando (`cli.py`) para as fases 1 a 4 sem QGIS.
- Núcleo geodésico (`core/geodetic.py`):
  - elipsoide GRS80 / SIRGAS 2000;
  - conversão UTM→geográfica **só para o hemisfério Sul**, com zona = EPSG − 31960 (EPSG 31978–31985);
  - Sistema Geodésico Local (SGL) com origem na média geocêntrica;
  - área de Gauss no SGL;
  - distância horizontal a partir da corda geocêntrica, arredondada "half-up" a 2 casas;
  - azimute por Vincenty. A função se chama `azimuth_puissant_pts`, mas chama Vincenty.
- Saídas organizadas por `core/organizador_saidas.py` em `01_Documentos`, `02_Shapes` e `03_Sessao`, com nome "<base> - AAAA-MM-DD - Vn".

## 2. Funcionalidades por fase

### Fase 0 — Coleta (`fase0_coleta/`)

- Camadas de satélite:
  - ESRI;
  - Google;
  - Bing (por http);
  - mosaico Sentinel-2 do INPE (WMS).
- Topodata/INPE via STAC (`pystac-client`) e curvas de nível com GDAL.
- **CAR:** tenta o WFS do SICAR e, se falhar, recorta SHPs locais pelo bbox.
- **SIGEF:**
  - na interface, só pasta local com buffer;
  - existe ainda um módulo que consulta o WFS do INCRA.
- **ITR/CCIR:** abre o navegador ou usa Selenium. No CCIR, o Selenium é esqueleto.
- **RTKLIB:**
  - converte o bruto para RINEX (convbin, timeout de 600 s);
  - lê os timestamps do `.obs`;
  - escolhe a efeméride: final se o rastreio tiver 13 dias ou mais, rápida se tiver 17 h ou mais, senão ultra-rápida.
- **RBMC:**
  - catálogo manual de cerca de 70 estações;
  - URLs do IBGE (RINEX 2/3, diário e 1 Hz), IGS, BKG e CODE;
  - "kit PPK" com RINEX, efemérides e IONEX.
- Visualizador web: HTML Leaflet + WMS i3Geo. Sem botão na interface atual.

### Entrada e vértices (`core/`, `ui/main_dialog_part1.py`)

- **TSV ROVER-GNSS:**
  - separador automático;
  - N/E arredondados a cm na leitura;
  - `STATUS_MAP` converte status em método NTGIR (FIXO-RTK→PG6, FIXO→PG2, ESTATICO→PG1, CINEMATICO→PG3, PPP→PG4, PPP-RTK→PG5, REDE-RTK→PG9, além de PT, PA e PS).
- Mapeador de colunas para TSV/CSV de outro layout, com presets em `~/.georural/column_presets.json`.
- **Ordenação (`core/ordering.py`):**
  - 1º polígono e 1º anel;
  - sentido horário;
  - início no vértice mais ao norte (desempate: maior E);
  - casamento com o TSV em 1 mm, com fallback até 50 cm e aviso.
- **Rota "polígono sem GNSS" (`core/poly_reader.py`):**
  - coordenadas arredondadas a 2 casas;
  - desvio-padrão e método escolhidos na tela, aplicados a todos os vértices.
- **Divisa certificada (`core/divisa_certificada.py`):** adota o vértice SIGEF do confrontante mais próximo (até 0,50 m) e mantém o código SIGEF.
- **Numeração (`core/numeracao_vertices.py`):**
  - códigos CRED-P/M/V-NNNN;
  - P e V dividem o mesmo contador; M tem contador próprio;
  - banco `numeracao_vertices.ods`;
  - na aba ① a gravação é adiada até fechar a janela, com confirmação;
  - na divisão e unificação a gravação é imediata.
- Edição em lote do tipo de limite (LA1–LA7, LN1–LN6; Ctrl+D) e do método.
- **Relatório de inconsistências:** DP por método, duplicados em 1 cm, segmentos curtos.

### Validação (`fase1_sigef/validador_ntgir.py`)

- Campos obrigatórios;
- método na lista NTGIR;
- precisão pelo tipo de limite: LA 0,50 m, LN 3,00 m, IN 7,50 m;
- CNS no formato XX.XXX-X;
- cobertura dos confrontantes;
- segmentos menores que 10 cm;
- área menor que 1 ha ou maior que 10.000 ha.

### Fase 1 — SIGEF (`fase1_sigef/`)

- **ODS SIGEF:** preenche o template oficial `template_sigef.ods` (abas `identificacao` e `perimetro_1`). Método não mapeado vira "*** VERIFICAR ***".
- **Shapefiles:**
  - pontos, limite, bandeirinhas, confrontantes e transições;
  - estrada LA3 com offsets de 1 m e 6 m;
  - rio LN1.
- **XLSX** espelho da ODS.
- **XML "SIGEF"** com namespace e XSD próprios. Não é gerado pela interface (ver §6).
- **Multiparcela:**
  - SHP com N polígonos;
  - XLSX, ODS (odfpy, sem template), XML, memoriais, checklist, requerimento e planta.
- **Importação do SIGEF:** ODS, PDF (pdfplumber) ou CSV (`_vertice`/`_parcela`) para shapes ou "trabalho". Coordenada geográfica do CSV é preservada como autoritativa.

### Fase 2 — Peças (`fase2_pecas/`)

- **Memorial georreferenciado:**
  - Lat/Lon DMS com altitude, ou UTM sem altitude;
  - perímetro "geodésico" (soma bruta arredondada ao final; é o padrão) ou "arredondado";
  - override de área e perímetro SIGEF, aplicado só ao cabeçalho;
  - texto legal cita o cap. 9 do MTGIR 2ª ed.
- Memorial multiparcela e consolidado; memorial a partir de PDF SIGEF; memorial avulso.
- **Planta A3 (`planta_qpt.py`, MODELO_GEO.qpt):**
  - preenche o template por UUID;
  - 4 vagas de confrontantes;
  - declinação: pygeomag, depois NOAA online;
  - convergência e fator k;
  - escala automática em múltiplos de 50;
  - grades UTM e geográfica;
  - croqui ESRI com raio de 12,5 km;
  - abre no designer, sem exportar PDF.
- Relatório de levantamento, relatório de precisão ("Anexo I"), planilha de vértices (DOCX e XLSX multiparcela), caderneta de campo e ficha ART/RRT.
- Cotas geodésicas em GPKG.
- **SRTM (`core/srtm_reader.py`):**
  - OpenTopoData online, GeoTIFF local, interpolação;
  - correção "IDW" que usa o desvio entre altitude real e SRTM.

### Fase 3 — Submissão (`fase3_submissao/sigef_api.py`)

- Checklist, pacote ZIP com `manifest.json` e log JSONL.
- O upload por Selenium é um esqueleto que sempre retorna "Submeta manualmente".

### Fase 4 — Cartório

- **`fase4_cartorial/` (MG, Prov. Conj. 93/2020):**
  - anuências "modelo real" (PF e condômino) com tabela do trecho em confrontação;
  - 12 tipos genéricos;
  - declaração de ciência de divisas;
  - requerimento ao CRI com catálogo de 24 atos de registro e 24 de averbação;
  - 8 requerimentos e declarações;
  - checklist.
- **`fase4_brasil/` (27 UFs):** mesmos modelos, parametrizados por `normas_estaduais.py`. Só MG tem artigos específicos; as demais UFs citam o "art. 440-AX do CNN" e o nome do provimento estadual.

### Fase 5 — Divisão, desmembramento e unificação (`fase5_divisao/`)

- **Corte por linha (`divisao_engine`):**
  - interpolação no SGL;
  - conservação exata de área (Δ = 0 em float64).
  - A interface atual usa a **importação de SHP com um polígono por parcela**. A rotina de linha de corte existe, mas não está acessível.
- **Numeração da divisão:**
  - vértice original mantém o código (tolerância de 0,10 m);
  - vértice novo vira `<cred>-M-NNNN`;
  - rotação para iniciar no norte.
- Memorial por parcela e memorial da **área total** (marcos sobre o perímetro, tolerância de 0,05 m).
- Confrontantes derivados das parcelas vizinhas.
- **Unificação:**
  - remove as divisas internas (casamento por cm);
  - remapeia os confrontantes;
  - camadas de hachura e de divisas internas;
  - memoriais dos polígonos internos com prefixo local.
- Requerimento e checklist de divisão.

### Gestão e IA

- **Gestão:** `gestao_escritorio.ods` com processos, financeiro e dashboard.
- **IA (aba oculta):**
  - Ollama local ou API OpenAI/Anthropic/Gemini;
  - diagnósticos "proativos" sem LLM.

## 3. Normas citadas no código

As normas abaixo aparecem **no código ou nos textos dos documentos gerados**. A pertinência jurídica e a numeração dos artigos **não foram verificadas** [A CONFIRMAR].

- **Técnicas:**
  - NTGIR 3ª ed. e MTGIR 2ª ed. (INCRA), citando a "Tabela 1", o "Anexo I" e o "cap. 9";
  - Lei 10.267/2001;
  - Decreto 4.449/2002 e Decreto 12.689/2025;
  - Portaria INCRA 116/2017 e Norma de Execução INCRA 107/2010. As duas aparecem para o mesmo requerimento de credenciamento, em módulos diferentes.
- **Registrais:**
  - Lei 6.015/73: arts. 7º, 167, 176 (§§1º, 3º–5º, 13), 212, 213 (II; §§2º, 8º, 10, 14, 17), 216-A, 225, 227, 234 e 235;
  - Lei 10.931/2004;
  - Lei 13.838/2019;
  - Lei 14.620/2023.
- **Corregedorias:**
  - Prov. CNJ 65/2017, 119/2023, 149/2023 (CNN) e 195/2025;
  - Prov. Conj. TJMG 93/2020, 121/2023 e 142/2025 (arts. 14, 135, 716, 717, 785, 891, 908–910, 949–964, 1.027–1.029, 1.156 e 1.173-A);
  - provimentos das outras 26 UFs, só pelo nome.
- **Outras:**
  - CF, arts. 20 III, 26 I e 231;
  - Lei 12.651/2012, Lei 9.985/2000, Lei 8.629/1993, Lei 6.766/1979, Lei 4.591/1964, Lei 9.514/1997, Lei 13.465/2017, Lei 13.777/2018, Lei 13.709/2018 (LGPD) e Lei 9.278/1996;
  - DL 9.760/1946, Decreto 1.832/1996 e Decreto 1.775/1996;
  - CPC (art. 618 e o antigo art. 615-A), CTN art. 185-A, CC arts. 1.314, 1.331, 1.369 e 1.725;
  - Resolução COAF 36/2021.

## 4. Dependências

**Obrigatórias:** QGIS 3.16+ com PyQGIS e GDAL/OGR.

**Pacotes Python, nenhum declarado no `metadata.txt`:**

- **python-docx**: todos os DOCX;
- **openpyxl**: XLSX. Sem ele, `xlsx_exporter` quebra com `NameError: Border` [CONFIRMADO POR TESTE];
- **pdfplumber**: PDF SIGEF;
- **odfpy**: ODS multiparcela;
- **pyshp** e **fiona**: leitura de SHP fora do QGIS, nos testes e no CLI;
- **pystac-client**: Topodata;
- **pygeomag**: opcional;
- **selenium**: opcional;
- executável **RTKLIB convbin**: opcional.

**Serviços externos**, que enviam coordenadas e dados para terceiros:

- OpenTopoData (SRTM);
- NOAA (declinação);
- ESRI, Google e Bing (mapas e croqui);
- INPE (STAC e WMS);
- SICAR (WFS);
- INCRA (WFS e i3Geo);
- IBGE, IGS, BKG e CODE (RBMC e efemérides);
- Receita e SNCR (navegador);
- APIs de IA, se configuradas.

## 5. Arquivos gravados fora da pasta de saída

- Pasta do banco (padrão `~/GEO_RURAL_DADOS` ou pasta escolhida): `numeracao_vertices.ods`, `banco_pessoas.ods`, `gestao_escritorio.ods` e `historico_servicos/`.
- Configurações: `~/.georural/` (config e presets) e `~/.geo_rural_3/` (`ai_config.json` e sessões de IA).
- Sessão: `<projeto>_georref_rural*.json` ou `~/.georural/sessoes/`.
- Arquivos temporários em `%TEMP%`, como `MODELO_GEO_georural_tmp.qpt` e `geo_rural_poly_entrada.shp`.

---

## 6. Limitações, erros e pontos de risco

### 6.1 Alta gravidade — podem gerar documento técnico ou legal errado

1. **Tabelas de método inconsistentes** [CONFIRMADO leitura; A CONFIRMAR na NTGIR]:
   - o `STATUS_MAP` do leitor de TSV grava PPP→**PG4**, REDE-RTK→**PG9**, IRRADIACAO→**PT1**, POLIGONACAO→**PT8** e AEROFOTO→**PA1**;
   - a tabela descritiva do exportador ODS diz PG4 = cinemático, PG9 = PPP, PT1 = poligonação, PT5 = irradiação e PS1 = aerofoto.
   - Um dos dois está errado, e o SIGEF recebe o código que veio do TSV.
2. **FLUTUANTE vira PG3** no ODS SIGEF, e CONTROLE vira PG6 (`dos_sigef_exporter`) [CONFIRMADO leitura].
3. **Rota "polígono sem GNSS"** aplica a todos os vértices o mesmo σ (padrão 0,05 m) e o mesmo método (o combo começa em **PG1**). Isso cria precisão e método que não vieram do campo [CONFIRMADO leitura].
4. **Marcos novos de divisão/unificação** saem como M / PG6 / FIXO-RTK com **σN = σE = σh = 0**. O preenchimento depende do usuário [CONFIRMADO leitura].
5. **SRTM gravado como altitude elipsoidal:** a altitude SRTM é ortométrica (EGM96) e entra sem correção do geoide, com diferença de metros. O fluxo multiparcela faz isso **automaticamente**, e a unificação traz a opção marcada por padrão [CONFIRMADO leitura].
6. **Dados pessoais do autor** (nome, CREA, código INCRA, RG, CPF, endereço, celular e e-mail) aparecem:
   - como padrão no formulário do RT;
   - como fallback em vários modelos;
   - **fixos no texto** de `anuencia_real.py`, que as anuências da Fase 5 usam para todos os confrontantes;
   - no rodapé de todos os DOCX (`docx_template`/`doc_formato`).
   - Outro profissional que use o plugin pode emitir documento em nome do autor [CONFIRMADO leitura].
7. **Afirmações automáticas em documentos a assinar** [CONFIRMADO leitura]:
   - "NÃO FOI CONSTATADA sobreposição" com UC, TI, quilombolas e embargos IBAMA (nenhuma consulta é feita);
   - "foram obtidas as anuências de todos os confrontantes";
   - "não houve investida… planta assinada por todos… firmas reconhecidas" (requerimento A02);
   - "inexistindo inventário aberto" (espólio com herdeiros);
   - bloco PEP **pré-marcado "( x ) NÃO"** em `requerimento_averbacao_rural_helton`;
   - declaração afirmativa "não sou PEP" em `requerimento_divisao`.
8. **Numeração duplicada:** na aba ②, converter **vários** vértices para M (ou de M para P/V) de uma vez dá o **mesmo código** a todos. Cada linha lê `proximo_numero()` do banco, que só é gravado ao fechar [CONFIRMADO leitura].
9. **Unificação com credencial fixa "VQUE":** códigos `VQUE-P-NNNN`, com consumo do banco da VQUE, qualquer que seja o credenciado [CONFIRMADO leitura].
10. **Kit PPK (RBMC):** `baixar_efemerides()` e `baixar_ionex()` levantam `TypeError` (`os.path.basename(dict)`), e o kit para depois do RINEX [CONFIRMADO POR TESTE].
11. **Fusos Norte e outros EPSG:** a conversão assume hemisfério Sul e zona = EPSG − 31960. Imóveis em fuso Norte (EPSG 31972–31977) saem com coordenadas geográficas erradas [CONFIRMADO leitura].
12. **Importações sem reprojeção:**
    - `poligono_importer` (divisão/unificação) usa as coordenadas do SHP como se estivessem no EPSG do trabalho;
    - `poly_reader` trata qualquer CRS projetado como UTM.
    - Um SHP em outro CRS gera coordenadas e área absurdas [CONFIRMADO leitura / PROVÁVEL].
13. **Leitor ODS sem zona:** a zona é deduzida só pelo Este (E < 500 km → 23; senão → 24), o que erra fora de MG. A interface bloqueia "detectar automaticamente", mas o módulo e o CLI permitem [CONFIRMADO leitura].
14. **M × V invertido:** o leitor ODS/PDF marca como **M** (marco) os vértices com método PS4/PA1/PA2, que são de vértice virtual [CONFIRMADO leitura].
15. **Validador e relatório de precisão** comparam o **σh** com o limite de posição, o que pode reprovar vértices bons na horizontal [CONFIRMADO leitura; A CONFIRMAR na NTGIR].
16. **Catálogos de atos do CRI incoerentes entre si:**
    - `requerimento_cri` diz compra e venda = inciso XXVIII e hipoteca = II;
    - `requerimento_cri_brasil` diz XXIV e XXVII;
    - averbações citadas como art. 167, **I** (que trata de registros);
    - CPC/1973 art. 615-A (revogado) [CONFIRMADO leitura; mérito A CONFIRMAR].
17. **Citação truncada do art. 213 §14** no requerimento A02 ("poderão os requerentes… pelos prejuízos", falta "responder") [CONFIRMADO leitura].
18. **Fixtures com dados reais:** `tests/fixtures/projeto_sitio_boa_vista.json` declara "Dados REAIS" e distribui nome, CPF, matrícula, ART e SNCR do proprietário e CPFs de cerca de 10 pessoas. O `templates/MODELO.qpt` traz nome e CPF de um cliente real (LGPD) [CONFIRMADO leitura].

### 6.2 Média gravidade — perda de dados ou resultado incompleto

19. **XML SIGEF nunca é gerado pela interface:** `chk_f1_xml` é criado, mas não entra no layout nem vem marcado [CONFIRMADO leitura].
20. **Pacote ZIP (Fase 3)** procura nomes fixos na raiz (`sigef_pontos.shp`, `planta.pdf`, `memorial_descritivo.docx`…). As fases 1 e 2 gravam em `02_Shapes`/`01_Documentos` com nome datado, e `planta.pdf` nunca é gerado. O pacote e o checklist saem quase vazios [PROVÁVEL, forte].
21. **Conflito de atributo `self.tbl_parcelas`:** a aba Gestão (parcelas financeiras) e a Fase 5 (parcelas da divisão) usam o mesmo nome. A Fase 5 é criada depois e sobrescreve, então os botões da Gestão agem na tabela da divisão [CONFIRMADO leitura].
22. **Sessão incompleta:** ao carregar, não voltam o RG e o CPF do RT (ficam os padrões do autor), os condôminos e representantes dos confrontantes, os vértices virtuais e os mapeamentos de colunas [CONFIRMADO leitura].
23. **Auto-save único:** a flag `_autosave_feito` só é zerada em "Salvar Sessão". Depois de "Nova Sessão", o fechamento não salva o trabalho novo [CONFIRMADO leitura].
24. **"Carregar como trabalho" (ODS/PDF/CSV):**
    - preenche widgets que não existem (`edt_nome_imovel`, `edt_municipio`…), então os dados do imóvel não chegam à aba ④;
    - a zona lida não ajusta o combo de zona;
    - a interface vai para a aba ⓪ [CONFIRMADO leitura].
25. **Divisa certificada:** vértices do polígono sem par no TSV nem no certificado são **descartados**, o que muda a geometria (há aviso no log) [CONFIRMADO leitura].
26. **Divisão:**
    - com mais de 10 parcelas → `IndexError` (`letras[i]`);
    - "travar nº do marco" com texto inválido começa em 1 sem aviso;
    - no corte por linha, só a 1ª e a última interseção são usadas, sem aviso [CONFIRMADO leitura].
27. **Confrontante com código de vértice inexistente** vai para o índice 0 sem aviso [CONFIRMADO leitura].
28. **"Mudar pasta" do banco (Gestão)** importa `plugin_geo_rural_3_54.core.db_config` (nome antigo) e falha em silêncio, mas a mensagem diz que moveu. "Abrir banco" abre `~/.georural/gestao_escritorio.ods` fixo [CONFIRMADO leitura].
29. `core/gestao_escritorio.caminho_banco()` usa `_PATH`, que não está definido → `NameError` [CONFIRMADO leitura].
30. **Planta (`planta_qpt`):**
    - o centroide procura `lat`/`lon`, mas os vértices usam `lat_dec`/`lon_dec`, então cai na conversão de **EPSG 31983 fixo**: declinação, convergência e fator k errados fora do 23S [PROVÁVEL, forte];
    - a grade geográfica usa o Norte UTM (metros) como latitude [CONFIRMADO leitura];
    - renomeia as camadas `sigef_*` do projeto [CONFIRMADO leitura].
31. **`relatorio_inconsistencias`** lê `dp_h`/`sigma_h`, mas os pontos guardam `dp_u`, então a checagem vertical nunca dispara. Também grava "/MG" fixo [CONFIRMADO leitura].
32. **`ai_proativa`** usa `PRECISAO_MAX.get('LN1')`, que nunca casa, e aplica 0,50 m a todos os tipos de limite [CONFIRMADO leitura].
33. **`memorial_geo`:** o `_DATUM_MAP` rotula EPSG 22521–22525 como "SAD 69" (são Córrego Alegre) e 29180–29195 como "WGS 84" (são SAD 69) [CONFIRMADO leitura].
34. **`sigef_para_shp`** (fallback OGR, fora do QGIS) calcula a área em graus² × 111 320², sem cos(lat) [CONFIRMADO leitura].
35. **Formato "XML SIGEF"** com namespace e XSD próprios; o SIGEF recebe a planilha ODS. O **ODS multiparcela** é gerado sem o template oficial [PROVÁVEL].
36. **Não há submissão ao SIGEF:** o Selenium é um esqueleto [CONFIRMADO leitura].

### 6.3 Baixa gravidade ou cosméticos

37. `unload` remove a ação do menu Vetor e do `mainWindow`, mas não do submenu da suíte. Ao recarregar, o item pode duplicar [PROVÁVEL].
38. `run()` cria uma janela nova a cada clique [PROVÁVEL].
39. **UF "MG" como padrão:**
    - "Guiricema" / "Visconde do Rio Branco" como padrão de município e comarca;
    - "/MG" ou "-MG" fixo em vários requerimentos (`fase4_cartorial`), em `requerimento_multiparcela` e no checklist de divisão.
40. **Caderneta de campo** com "Céu limpo, ventos fracos, PDOP<4" fixo. Equipamentos padrão "Trimble R8s / R10".
41. **Tabela de trecho** das anuências com azimute truncado (só graus e minutos).
42. **Mapeador de colunas:** sugere 'lat'→Norte e 'lon'→Este, e "h" casa qualquer coluna (ex.: `hora_ini` → altitude).
43. Coluna "Tipo" da aba ② mostra só M ou P.
44. `checklist_prov93`: "Comprovante… cartoriais" herda o status do CAR (casa "car").
45. `ordering_standalone` (CLI e testes) não força o sentido horário nem o início ao norte, apesar de dizer "idêntico".
46. `poly_reader` arredonda as coordenadas a cm.
47. `planta_modelo_qpt` substitui qualquer rótulo com o texto "2" ou "A3".
48. **Código morto:**
    - `fase0/polygon_to_tsv` importa funções inexistentes;
    - o visualizador web não tem botão;
    - as rotinas de linha de corte usam `edt_linha`, que não é criado;
    - `_aplicar_offsets_confrontantes` está desativado.
49. **Mensagens trocadas:** "aba ④ Confrontantes" (é a ③), log "/requerimentos/" (é `01_Documentos`) e título "Fases 0 a 4" (há a Fase 5).
50. Chaves de API de IA em JSON texto puro; a chave do Gemini vai na URL. `banco_pessoas.ods` guarda dados pessoais sem proteção.

## 7. Testes executados

| Teste | Resultado |
|---|---|
| `tests/run_all.py` | **19/19 OK**: área 32,3097 ha, perímetro, divisão e unificação com Δ = 0, DOCX, XLSX e requerimentos. O metadata diz 18 testes; o README diz 12. |
| `fase5_divisao/test_divisao.py` | **1 OK, 1 FALHA**. O TEST 14 espera o vértice de divisa como PS4/V, e o código atual (v3.169) cria PG6/M. O teste está desatualizado. |
| `fase5_divisao/test_sprint2.py` | **4/4 OK** |
| `rbmc_integration.baixar_efemerides` / `baixar_ionex` | `TypeError` reproduzido |

Os testes gravam em `/tmp/…`, caminho Unix.

## 8. Não testado (depende do QGIS ou de rede)

Não foram executados:

- toda a interface;
- a planta (layout/QPT);
- as camadas no projeto;
- a importação de SHP pelo QGIS;
- SRTM online;
- WFS do CAR e do SIGEF;
- RBMC real;
- Topodata;
- declinação NOAA;
- Selenium;
- a conformidade dos arquivos ODS com o SIGEF real;
- o conteúdo jurídico dos modelos.
