# Descritivo Técnico-Funcional — Precisa Earth Engine

**Pasta:** `precisa_earth_engine` · **Versão declarada:** 1.0.0 (experimental) · **QGIS mínimo:** 3.28
**Arquivos lidos:** `__init__.py`, `precisa_earth_engine.py`, `dialog_principal.py`,
`dialog_credenciais.py`, `ee_config.py`, `ee_processamento.py`, `ee_task.py`,
`inpe_processamento.py`, `camadas_qgis.py`, `README.md`, `requirements.txt`, `metadata.txt`

---

## 1. Arquitetura

| Módulo | Responsabilidade |
|---|---|
| `__init__.py` | `classFactory`; recarrega os submódulos em ordem (para o “Recarregar plugin” funcionar sem reiniciar o QGIS) |
| `precisa_earth_engine.py` | Ação no menu *Complementos → Precisa Earth Engine* e na barra |
| `dialog_principal.py` | Tela principal, validação, disparo da `QgsTask`, carga e estilo das camadas |
| `dialog_credenciais.py` | Tela de JSON da *service account* + Project ID + teste de conexão |
| `ee_config.py` | Persistência em `QSettings` (grupo `PrecisaEarthEngine`: caminho do JSON, Project ID, token BDC) |
| `ee_processamento.py` | Autenticação EE, união/buffer/reprojeção da geometria, montagem das imagens, download e URL de tiles |
| `inpe_processamento.py` | Cliente STAC do INPE/BDC: listagem de coleções, busca, escolha de cena, download e recorte GDAL |
| `ee_task.py` | `TarefaProcessamentoEE` (`QgsTask`, cancelável) — executa as fontes em sequência fora da thread da interface |
| `camadas_qgis.py` | Adição de raster/XYZ ao projeto e renderizadores |

## 2. Funcionalidades implementadas

### 2.1 Geometria de recorte (`obter_geometria_bufferizada`)
- `unaryUnion` de todas as feições com geometria da camada escolhida
  (filtro de camadas: somente polígono). Não usa seleção — todas as feições.
- Camada em CRS geográfico → reprojeta para **UTM WGS84** (EPSG 326xx/327xx)
  calculado pelo centroide, aplica o buffer em metros, volta para EPSG 4326.
- Camada em CRS projetado → buffer na unidade do próprio CRS, depois EPSG 4326.
- Buffer com 8 segmentos por quadrante; erro amigável se o resultado ficar vazio.

### 2.2 Google Earth Engine
- Autenticação: `ee.ServiceAccountCredentials(client_email, key_file)` +
  `ee.Initialize(credenciais, project=...)`.
- **Dynamic World V1** (`GOOGLE/DYNAMICWORLD/V1`): `filterBounds` + `filterDate`,
  **moda** da banda `label` no período, `clip` na geometria. Paleta oficial de 9 classes.
- **Sentinel-2** (`COPERNICUS/S2_HARMONIZED` — nível L1C/TOA): filtro
  `CLOUDY_PIXEL_PERCENTAGE ≤ 35`, **mediana**, bandas B4/B3/B2, `clip`.
  Visualização 0–3000.
- Escala padrão 10 m para ambas; pode ser trocada (1–1000 m).
- **Download:** `getDownloadURL(format=GEO_TIFF, region, scale)` + `requests`
  (timeout 180 s, gravação em blocos de 8 KB).
- **Streaming:** `getMapId(vis_params)` → URL XYZ carregada como camada
  `wms type=xyz` (zoom 0–20).

### 2.3 CBERS-4A WPM (INPE / Brazil Data Cube, STAC `https://data.inpe.br/bdc/stac/v1`)
- Lista coleções cujo id ou título contém “wpm” (tarefa em segundo plano).
- Busca `POST /search` com `intersects` (GeoJSON da geometria), intervalo
  `data_inicio T00:00:00Z / data_fim T23:59:59Z`, `limit = 100`.
- Escolhe a cena de **menor `eo:cloud_cover`**; sem essa propriedade, a mais recente.
- Asset: primeiro cujo nome contém “pan”; senão, o primeiro `.tif/.tiff`.
- Token passado como parâmetro `access_token` na URL.
- Download para arquivo temporário → `gdal.Warp` com *cutline* (WKT, EPSG 4326),
  `cropToCutline`, **reprojeção para EPSG 4326**, NoData = 0 → arquivo apagado.
- Sempre download: no modo Streaming grava em `tempfile.gettempdir()`.

### 2.4 Carga e estilo
- Dynamic World → `QgsPalettedRasterRenderer` (classes 0–8 com nomes em português).
- Sentinel-2 → `QgsMultiBandColorRenderer` 1/2/3, realce 0–3000.
- CBERS → `QgsSingleBandGrayRenderer`, realce mín–máx das estatísticas da banda.
- Estilo aplicado apenas às camadas baixadas (streaming já vem renderizado pelo EE).

## 3. Normas técnicas referenciadas
**Nenhuma norma técnica é citada no código.** Padrões usados: STAC (INPE/BDC),
EPSG 4326, catálogos oficiais do Earth Engine.

## 4. Dependências
| Dependência | Uso | Observação |
|---|---|---|
| QGIS ≥ 3.28 | Interface, `QgsTask`, geometria, raster | — |
| `earthengine-api` (pip) | Dynamic World e Sentinel-2 | Checada em tempo de execução; ausência gera aviso |
| `requests` | Downloads e STAC | **Importado no topo de `inpe_processamento.py`** — se faltar, o plugin inteiro não carrega |
| GDAL (`osgeo.gdal`) | Recorte do CBERS | Vem com o QGIS |
| Conta Google Cloud + Service Account EE | Autenticação EE | — |
| Token Brazil Data Cube | CBERS-4A | Cadastro gratuito |

## 5. Limitações conhecidas e pontos de atenção
1. **Limite de download do Earth Engine:** `getDownloadURL` recusa arquivos
   grandes (limite de tamanho da API); áreas extensas em 10 m falham com erro
   genérico. Não há divisão em blocos nem exportação via Drive/Cloud Storage.
2. **Data final exclusiva no EE:** `filterDate(inicio, fim)` não inclui o dia
   final; no CBERS o dia final é incluído (até 23:59:59Z). Comportamento
   diferente entre fontes.
3. **Sentinel-2 sem máscara de nuvem por pixel** — só filtro por cena (≤ 35 %)
   e mediana; coleção L1C (reflectância de topo de atmosfera), não SR.
4. **Links de streaming expiram** (tokens de `getMapId`); camadas XYZ salvas
   no projeto deixam de funcionar depois de algum tempo.
5. **Busca STAC sem paginação** (`limit = 100`): com mais de 100 cenas no
   período, a de menor nuvem pode não ser considerada.
6. **Escolha do asset por heurística** (“pan” no nome; senão primeiro `.tif`)
   — em coleções com outra nomenclatura, pode baixar uma banda que não é a
   pancromática sem aviso. O próprio README pede ajuste manual nesse caso.
7. **Cena inteira baixada** antes do recorte (pode ser grande/lenta; sem
   progresso intra-download).
8. No modo Streaming, o CBERS vai para a pasta temporária com nome fixo
   `CBERS4A_WPM_<datas>.tif` — sobrescreve execuções anteriores com o mesmo período.
9. Token BDC trafega como parâmetro de URL (pode aparecer em logs de proxy).
10. Buffer em camada projetada usa a unidade do CRS (se o CRS for em pés, o
    buffer é em pés, apesar do rótulo “m”).
11. **“Testar conexão” grava as credenciais** em `QSettings` mesmo que o
    usuário depois clique em Cancelar.
12. Progresso só avança por fonte concluída; cancelamento só pelo gerenciador
    de tarefas do QGIS (não há botão Cancelar na tela).
13. Plugin **não integrado à Suíte Precisa** (sem chaves `precisa_*` no
    metadata; menu próprio em *Complementos*).
14. Não há comentário no código indicando trecho não testado; os itens acima
    vêm da leitura do código.
