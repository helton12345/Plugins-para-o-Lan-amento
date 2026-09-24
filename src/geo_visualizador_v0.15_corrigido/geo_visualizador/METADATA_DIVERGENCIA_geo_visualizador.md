# Divergências do metadata.txt — Precisa Geo Visualizador

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `description` / `about` | Visualizador de loteamento a partir de imagem georreferenciada ou shapefile/GeoJSON | Não menciona o **modo Venda (Produtos 2 e 3)**: status/valor via Google Apps Script com atualização automática, preparação de camada e exportação de CSV para planilha; nem camada de rua, camada de cotas e histórico de links publicados | Média |
| 2 | `version` | 0.15 | `metadata.txt` datado de 19/08 15:12, mas **`dialog_gerar.py` foi alterado em 08/09 01:14** sem mudança de versão (o zip se chama “v0.15_corrigido”) | Média |
| 3 | `icon` | **Ausente** (só `precisa_icon=icon.png`) | O Gerenciador de Complementos do QGIS usa `icon=` | Média |
| 4 | Dependências | Nenhuma declarada | Usa `numpy` e `Pillow` (import obrigatório na tela de geração) e serviços externos: Netlify, Google Apps Script, Leaflet via unpkg, tiles Google | Média |
| 5 | `experimental` | Ausente | Versão 0.x e modo imagem marcado no código como “não testado com raster real” | Baixa |
| 6 | `category` | Plugins | Categorias usuais do QGIS são Raster/Vector/Database/Web; o plugin é mais próximo de **Web** | Baixa |
| 7 | `tracker` / `repository` | Vazios | — | Baixa |
| 8 | `email` | precisagrimensura@gmail.com | Coincide com o rodapé do site; outros plugins da suíte usam `precisagrimensuratop@gmail.com` | Informativo (padronização) |
| 9 | `qgisMinimumVersion` | 3.16 | APIs usadas existem em 3.16 (`hasCurvedSegments`, `asJson(precision)`, `QgsMapLayerComboBox.setAllowEmptyLayer`). Consistente | — |
| 10 | Menu | Metadata não descreve | Por causa da *bridge* embutida, o plugin **não cria entrada no menu Complementos**, só ícone na barra e acesso pela suíte | Informativo |

**Resumo:** metadata **com divergências** — descrição sem o modo Venda,
alteração de código após a versão 0.15 sem incremento, e ausência de `icon=`
e de dependências.
