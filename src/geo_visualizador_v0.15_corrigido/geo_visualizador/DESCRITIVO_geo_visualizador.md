# Descritivo Técnico-Funcional — Precisa Geo Visualizador

**Pasta:** `geo_visualizador` · **Versão declarada:** 0.15 · **QGIS mínimo declarado:** 3.16
**Arquivos lidos:** `__init__.py`, `_precisa_bridge.py`, `geo_visualizador.py`,
`dialog_lista_projetos.py`, `dialog_selecionar_camada.py`, `dialog_gerar.py`,
`core/*.py` (10 módulos), `templates/site_template.html`, `metadata.txt`

---

## 1. Arquitetura

| Módulo | Responsabilidade |
|---|---|
| `__init__.py` + `_precisa_bridge.py` | `classFactory` envolvido pela “bridge” da suíte: durante `initGui`, anula `addPluginTo*Menu` e `addToolBar` do `iface` e remove menus novos da barra de menus |
| `geo_visualizador.py` | Ação “Geo Visualizador - Gerar Site”; `run()` para a suíte |
| `dialog_lista_projetos.py` | Janela principal: histórico, checagem de estado/URL no Netlify |
| `dialog_selecionar_camada.py` | Escolha do modo, parâmetros, geração do HTML e do CSV |
| `dialog_gerar.py` | API key, publicação (ZIP → Netlify) ou gravação do HTML |
| `core/dados_vetorial.py` | GeoJSON + JS dos lotes, rua e cotas |
| `core/dados_imagem.py` | Overlay de imagem (PNG base64) |
| `core/dados_venda.py` | GeoJSON + JS com *polling* do Apps Script |
| `core/preparar_camada.py` | Camada em memória com ID_LOTE/STATUS/VALOR |
| `core/exportar_planilha.py` | CSV para a planilha (Produto 2/3) |
| `core/template_html.py` | Substitui `{{TITULO}}` e `{{DADOS_JS}}` no template |
| `core/ofuscacao.py` | Renomeia identificadores JS/CSS/ids, remove comentários HTML |
| `core/netlify_client.py` | `POST /api/v1/sites` (ZIP), consulta de site e de deploys (urllib) |
| `core/historico_links.py` | CSV `geo_visualizador_links.csv` com migração de formatos v0.3/v0.4 |
| `core/config_local.py` | API key em `QSettings` codificada em base64 |

## 2. Funcionalidades implementadas

### 2.1 Site gerado (template)
- Leaflet 1.9.4 via CDN `unpkg.com`; fundo satélite **Google** (`mt1.google.com/vt/lyrs=s`), zoom até 22.
- GPS por `navigator.geolocation.watchPosition` (alta precisão), marcador e
  círculo de precisão; faixas: < 10 m “GPS OK”, < 50 m “Melhorando”, senão “Sinal fraco”.
- Botões satélite liga/desliga e centralizar (zoom 19). Rodapé com título e
  e-mail fixo `precisagrimensura@gmail.com`.

### 2.2 Modo vetorial
- Reprojeção para EPSG 4326 com `QgsCoordinateTransform`.
- Simplificação Douglas-Peucker (`simplify(0.5)`) **na unidade do CRS original**,
  exceto geometrias com segmentos curvos (preserva arcos).
- GeoJSON com 7 casas decimais; centroide calculado após reprojeção.
- Estilo por categoria (institucional/verde/lote) a partir do campo quadra.
- Rótulos: exibidos quando a distância em pixels entre os **dois centroides de
  lote mais próximos** ≥ 90 px (cálculo O(n²) no navegador, uma vez).
- Rua: polígonos cinza tracejados, não interativos.
- Cotas: deduplicação de pontos com mesmo texto a < 0,3 m; texto rotacionado
  por CSS com o valor do campo `rotacao`; exibidas quando o percentil 20 da
  distância ao vizinho mais próximo ≥ 45 px.

### 2.3 Modo imagem
- Extensão da camada raster transformada para WGS84 (`transformBoundingBox`).
- Arquivo de origem aberto com **PIL**, pixels com R, G e B > limiar ficam
  transparentes; PNG embutido em base64 no HTML; `L.imageOverlay` com opacidade.

### 2.4 Modo venda
- Campos obrigatórios: ID, lote, quadra. Popup com perímetro e 4 lados,
  status e valor (valor só se *disponivel*).
- `fetch(URL_STATUS)` imediato e a cada N s; cores: disponível `#2e7d32`,
  reservado `#f9a825`, vendido `#c62828`; ID ausente no JSON → tratado como *disponivel*.
- Rótulos visíveis a partir do zoom 18 (fixo).
- Preparação de camada: memória, `ID_LOTE = "<QUADRA>-<LOTE>"`, bloqueio por
  duplicidade ou campos já existentes.
- CSV: `id_lote, quadra, lote, status, valor` (+ `forma_pagamento, num_parcelas,
  vencimento, adimplencia` no Produto 3); status vazio → `disponivel`.

### 2.5 Publicação
- ZIP em memória: `index.html` **ofuscado** + `_headers` forçando
  `Content-Type: text/html; charset=utf-8`.
- `POST https://api.netlify.com/api/v1/sites?name=<slug do título>`; retorna
  `ssl_url`/`url` e `id`; registro no histórico como *pendente*.
- Lista de projetos: para *pendente*, lê `GET /sites/{id}/deploys` e marca
  *pronto* quando `state == ready`; URLs sem HTTPS ou com `--` no host são
  trocadas pela URL definitiva do site.
- “Gerar HTML” grava o HTML **sem ofuscação**.

## 3. Normas técnicas referenciadas
**Nenhuma.** O código não cita norma técnica. Padrões usados: GeoJSON/EPSG 4326,
API REST do Netlify, Leaflet.

## 4. Dependências
| Dependência | Uso | Observação |
|---|---|---|
| QGIS (PyQt, `qgis.core`, `qgis.gui`) | Tudo | — |
| `numpy`, `Pillow` | Modo imagem | Import no topo de `dados_imagem.py`, que é importado por `dialog_selecionar_camada.py` — **sem elas, a tela “+ Novo site” não abre em nenhum modo** (ver limitação 3) |
| Conta Netlify + *Personal Access Token* | Publicação direta | Opcional |
| Google Apps Script publicado | Modo venda | O script `apps_script_espelho_vendas.gs` citado no código **não está no pacote** |
| Internet no celular | Leaflet (unpkg), satélite Google, Apps Script | — |

## 5. Limitações conhecidas e pontos de atenção
1. **Simplificação na unidade do CRS:** `tolerancia_simplificacao = 0.5` é
   aplicada antes da reprojeção, documentada como “metros”. Em camada com CRS
   geográfico (EPSG 4674/4326) isso vira **0,5 grau (~55 km)** e destrói as
   geometrias sem lados curvos. Não há checagem.
2. **Modo imagem marcado no código como não testado:** comentário
   “ASSUNCAO (nao testada com raster real)” — PIL lê só PNG/TIFF/JPEG 8 bits
   comuns; raster multibanda, 16 bits, PDF ou origem com opções GDAL no
   `source()` devem falhar. Planta rotacionada/UTM vira *bounding box* em
   lat/long (pequena distorção). Imagem grande infla o HTML (base64).
3. **Dependência de numpy/Pillow no carregamento da tela de geração:** o
   import de `core.dados_imagem` no topo de `dialog_selecionar_camada.py`
   faz a tela inteira falhar se faltar numpy/Pillow, embora a mensagem de
   dependência só exista no modo imagem.
4. **Cada publicação cria um site novo**; não há atualização de site
   existente. Nome repetido → Netlify recusa (HTTP 422, mensagem genérica).
5. Publicação e checagem de status rodam **na thread da interface**
   (timeouts 60 s/15 s): o QGIS congela durante as chamadas; a lista consulta
   o Netlify para cada item pendente ao abrir.
6. **Modo venda:** lote ausente na resposta do Apps Script aparece como
   *disponível*; o campo de área não é exibido; rótulo em zoom fixo 18
   (o modo vetorial usa a regra dinâmica). Camada preparada é **em memória**
   (perde-se ao fechar o projeto sem salvar); QUADRA/LOTE nulos geram ID `NULL-NULL`.
7. **Fundo de satélite do Google** usado por URL direta, sem atribuição —
   uso possivelmente em desacordo com os termos do serviço.
8. API key salva em base64 (ofuscação, não criptografia) — declarado no código.
9. Ofuscação fraca (renomeia ~15 identificadores) — declarada no código.
10. **Bridge da suíte sempre ativa:** como `_precisa_bridge.py` está no próprio
    pacote, o menu *Complementos* é suprimido mesmo sem a suíte; fica apenas o
    ícone da barra de ferramentas (`addToolBarIcon` não é bloqueado).
11. Docstring de `template_html.py` diz que os módulos de dados “ainda não
    foram implementados” — desatualizada.
12. `arquivos_extras` do `DialogGerarSite` existe mas nenhum modo o usa.
