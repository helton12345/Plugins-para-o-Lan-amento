# Descritivo Técnico-Funcional — Conversor de Memoriais Pro

**Pasta:** `gerador_de_memoriais` · **Versão declarada:** 1.4 · **QGIS:** 3.16 – 3.99 · **Licença:** GPL-3.0
**Arquivos lidos:** `__init__.py`, `plugin.py`, `dialog.py`, `parser_texto.py`,
`camadas.py`, `motor_camada.py`, `exportar_docx_camada.py`, `shp_generator.py`,
`pdf_parser_sigef.py`, `docx_generator_sigef.py`, `sigef_para_vertices.py`,
`metadata.txt`, `README.md`, `FAQ.md`, `CHANGELOG.md`, `.github/ISSUE_TEMPLATE/bug_report.md`

---

## 1. Arquitetura
Fusão de três plugins anteriores (Restituição, Memorial Rural Gerador, Memorial
SIGEF), conforme CHANGELOG. Diálogo único com 3 abas; cada aba chama seu motor.

| Módulo | Aba | Função |
|---|---|---|
| `parser_texto.py` | Texto | Limpeza de linhas repetidas, regex UTM e lat/long, azimute, distância, confrontante, fuso, vértices não reconhecidos |
| `camadas.py` | Texto, SIGEF | Camadas em memória de pontos e polígono; gravação em shapefile (V3 com *fallback* V2) |
| `motor_camada.py` | Camada | Texto do memorial (retas/curvas, método de cálculo, fuso) |
| `exportar_docx_camada.py` | Camada | Formatação .docx (Times New Roman 11, margens 2,5/2,5/3,0/2,0 cm) |
| `shp_generator.py` | Camada | Shapefile de vértices numerados com prefixo |
| `pdf_parser_sigef.py` | SIGEF | Extração do cabeçalho e dos vértices (pdfplumber + regex) |
| `docx_generator_sigef.py` | SIGEF | Narrativa com detecção de curva em graus e .docx (Arial 12) |
| `sigef_para_vertices.py` | SIGEF | DMS → decimal para `camadas.py` |

## 2. Funcionalidades implementadas

### 2.1 Aba Texto
- Entrada colada ou arquivo `.txt` (UTF-8), `.docx` (parágrafos) ou `.pdf`
  (texto extraído por página).
- Remove linhas que se repetem **identicamente** e marcadores “--- Página N ---”.
- Regex UTM: `vértice <cód> … de coordenada(s) N <n> m e E <e> m` (≤ 150
  caracteres entre código e coordenadas); números BR ou EN.
- Regex geográfico: `vértice <cód> … Longitude <DMS> Latitude <DMS>`.
- Por vértice: azimute (texto), distância, confrontante (“confrontando [neste
  trecho] com …” vale até nova declaração; ou “situado no limite com …”).
- Remove o último vértice se repetir o primeiro (tolerância 1 mm / 1e-6°).
- Fuso: primeira ocorrência de `fuso NN [N|S|norte|sul]` (hemisfério padrão S);
  CRS = EPSG 31960+NN para 17S–25S, senão PROJ montado (GRS80).
- Relatório de códigos citados no texto sem coordenadas extraídas.
- Camadas: pontos (`vertice`, `norte`/`latitude`, `confrontante`,
  `leste`/`longitude`, `azimute`, `distancia`) e polígono (`area_m2`,
  `perimetro`, `vertices`).

### 2.2 Aba Camada QGIS
- Aviso bloqueante (com opção de continuar) para CRS geográfico.
- Por feição: 1º anel do 1º polígono; remove segmentos < 0,1 mm; começa no
  vértice de maior N (empate: maior E), recuando até o início de uma curva.
- Reta: azimute plano `atan2(ΔE, ΔN)` em GMS e distância euclidiana (Decimal, 50 dígitos).
- Curva: ≥ **100 segmentos** consecutivos de mesmo comprimento (±1 mm),
  pontos a ±2 cm do raio; reavaliação a cada 20 segmentos (raio ±20 %,
  variação angular ±5°). Texto: raio médio, desenvolvimento e ângulo central (D/R).
- Método de cálculo lido do CRS: UTM (fuso/hemisfério do `+zone`/`+south`
  do PROJ, com *fallback* pela descrição), geográfico, ou “Plano Topográfico
  Local” para qualquer outro projetado; datum SIRGAS 2000 / WGS 84 / SAD 69
  ou a descrição do CRS.
- Nome da feição: campos `lote, nome, name, id, ID, LOTE, imovel, IMOVEL`, senão `FID n`.
- Shapefile de vértices: `seq`, `codigo` (`<prefixo>NNNNN`), `este`, `norte`,
  `feat_nome`; vértices duplicados (arredondados a 0,1 mm) numerados uma vez.

### 2.3 Aba SIGEF
- Cabeçalho por regex: denominação, proprietário, matrícula, município/UF,
  cartório (CNS), área (SGL, ha), perímetro, RT, formação, código de
  credenciamento, conselho, sistema geodésico, documento de RT.
- Parcelas separadas por “DESCRIÇÃO DA PARCELA n/m”; se não houver, uma única.
- Vértices por regex: `código lon lat alt código_vante azimute distância`, com
  código no formato **`[A-Z]{2,6}-[PV]-\d+`**.
- Curva: ≥ 3 segmentos de comprimento igual (±5 %), pontos a ±1 % do raio;
  raio, corda e arco convertidos para metros por fator médio
  111.320 m/grau × (1 + cos φ)/2; concavidade esquerda/direita.
- Camadas em EPSG 4674 por parcela.

## 3. Normas e referências no código
- **Nenhuma NBR é citada.** Referências explícitas: **SIGEF/INCRA** (formato do
  PDF e padrão de códigos de vértice), **SIRGAS 2000**, Sistema Geodésico Local
  (texto de rodapé copiado do padrão SIGEF), projeção UTM.
- O padrão narrativo (“Inicia-se a descrição deste perímetro no vértice…”)
  segue o modelo usado pelo SIGEF, mas o código não afirma conformidade com
  norma técnica.

## 4. Dependências
| Dependência | Onde | Declarada? |
|---|---|---|
| `python-docx` | Camada, SIGEF, Texto (.docx) | README/FAQ; não no metadata |
| `pdfplumber` | SIGEF, Texto (.pdf) | README/FAQ; não no metadata |
| QGIS ≥ 3.16 | Todo | Sim |

## 5. Limitações conhecidas e pontos de atenção
1. **Vértices do tipo “M” (marco) ignorados no SIGEF:** a regex aceita só
   `-P-` e `-V-`. Códigos SIGEF de marco (`XXXX-M-0001`) não são lidos e
   somem do memorial e das camadas **sem aviso**.
2. **Área/perímetro errados nas camadas geográficas:** em `camadas.py`,
   `area_m2 = geom.area()` e `perimetro = geom.length()` são calculados no
   CRS da camada; no modo SIGEF e no modo Texto em lat/long (EPSG 4674) os
   valores ficam em **graus²/graus**, não em m²/m.
3. **Formato numérico não brasileiro no modo Camada:** coordenadas com
   `{:,.3f}` (vírgula de milhar, ponto decimal), distâncias e segundos com
   ponto decimal.
4. **Modo Camada descreve só o 1º anel do 1º polígono** de cada feição
   (outras partes e furos ignorados), mas a área vem de `geom.area()` (todas
   as partes, descontando furos) — área e perímetro podem ficar incoerentes.
5. Detecção de curva no modo Camada exige ≥ 100 segmentos iguais (±1 mm);
   curvas com poucos vértices viram retas (documentado no FAQ).
6. **Falso positivo de curva no SIGEF:** 3+ segmentos consecutivos de mesmo
   comprimento com vértices concíclicos (ex.: parcela quadrada) são descritos
   como curva. Raio/corda em metros por conversão aproximada.
7. Qualquer CRS projetado não UTM é declarado como “Plano Topográfico Local”.
8. **Textos fixos:** SIGEF sempre “DIVISÃO DE IMÓVEL RURAL”; Camada sempre
   “confrontações físicas … cercas, muros e ruas” e “Livro 2 … Comarca de [PREENCHER]”.
9. **Import sem tratamento no carregamento de arquivo (aba Texto):** falta de
   `pdfplumber`/`python-docx` gera `ImportError`, que não é capturado
   (`except (OSError, ValueError)`), e o erro aparece como exceção do Python.
10. Detecção de fuso pega a **primeira** ocorrência de “fuso NN” no texto.
11. Shapefiles do modo Texto têm nome fixo e sobrescrevem execuções anteriores.
12. Rodapé promocional fixo nos .docx gerados.
13. Docstrings desatualizadas: `pdf_parser_sigef.py` e `gerar_narrativa_parcela`
    ainda falam em “planilha oficial de confrontações”/“cruzamento”, removidos
    na v1.1; `_separar_matricula_nome` é código morto.
14. Nenhum comentário marca trechos como “não testados”; itens acima vêm da
    leitura do código.
