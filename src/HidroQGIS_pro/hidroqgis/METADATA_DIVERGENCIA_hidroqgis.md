# Divergências do metadata.txt — HidroQGIS

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `about` | “processamento via subprocess, sem depender do Processing Framework do QGIS” | O pipeline usa `processing.run` (`gdal:cliprasterbymasklayer`, `gdal:polygonize`, `gdal:merge`, `native:extractbyexpression`, rastercalculator) e o WhiteboxTools pela biblioteca `whitebox` | Média |
| 2 | `about` | “Datum padrão: SIRGAS 2000 (EPSG:4674)” | Não há reprojeção para 4674. O SRC é o do MDE; o texto “EPSG:4674” fica fixo só na planta e no memorial | Média |
| 3 | `icon` / código | `icon=icon.png` (existe na raiz) | `initGui` carrega `resources/icon.png`, que não existe: o botão da barra fica sem ícone | Média |
| 4 | `description` / `about` | Delimitação de corpos d'água, drenagem e sub-bacias | Não menciona: planta PDF, memorial DOCX, KMZ, visualizador Netlify, comparação temporal, download TOPODATA, recorte pelo limite | Baixa |
| 5 | `about` | “delimitação automatizada de corpos d'água” | Os corpos são áreas de fluxo acumulado acima de um limiar, não espelhos d'água | Baixa (terminologia) |
| 6 | Dependências | Não declaradas | `whitebox` (obrigatória), `python-docx`, `pystac-client` | Baixa |
| 7 | `version` | 1.1.0 | O rodapé da interface diz “HidroQGIS v1.0” | Baixa |
| 8 | `qgisMinimumVersion` / `qgisMaximumVersion` | 3.16 – 4.99 (“Compatível com QGIS 3.16 a 4.x”) | Não foi encontrado uso de API posterior à 3.16; compatibilidade com o QGIS 4 não verificada | Informativo |
| 9 | `email` / `homepage` / `tracker` / `repository` | Vazios | — | Baixa |
| 10 | `precisa_*` | Submenu “📁 Projetos”, `precisa_icon=icon.png` | Consistente (o arquivo existe na raiz) | — |

**Resumo:** metadata **com divergências**:
- o `about` descreve uma arquitetura (sem Processing, datum 4674) que o código não segue;
- omite a maior parte dos produtos;
- o ícone da barra não é encontrado pelo código.
