# Divergências do metadata.txt — Precisa Earth Engine

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado no metadata.txt | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `description` | “Busca dados do **Google Earth Engine** (Dynamic World, Sentinel-2, **CBERS-4A WPM**)” | CBERS-4A WPM **não vem do Earth Engine**: vem da API STAC do INPE / Brazil Data Cube (`inpe_processamento.py`) | Alta |
| 2 | `about` | “CBERS-4A WPM (**via Asset próprio do usuário**)” | Não existe leitura de *Asset* do usuário no EE; o código busca coleções WPM no STAC do INPE com **token BDC** | Alta |
| 3 | `about` | Autenticação “via Service Account do Google Earth Engine” | Correto para DW/S2, mas omite o **token do Brazil Data Cube** exigido pelo CBERS | Média |
| 4 | `about` | Não menciona | Buffer negativo, resolução configurável, estilização automática e aviso de que o CBERS é sempre baixado/recortado localmente | Baixa |
| 5 | Dependências | Nenhuma declarada | Código usa `earthengine-api`, `requests` (import obrigatório no carregamento) e `osgeo.gdal`. Existe `requirements.txt` com `earthengine-api` e `requests` | Média |
| 6 | `version` | 1.0.0 | Todos os arquivos com data 14/09 09:43, **exceto `__init__.py` (14/09 10:26)**, alterado depois do metadata (recarga de submódulos) sem mudança de versão | Baixa |
| 7 | `tracker` / `repository` | `https://github.com/` | *Placeholder* genérico, não aponta para repositório real | Baixa |
| 8 | `author` / `email` | “Helton” / e-mail pessoal | Demais plugins da suíte usam “Precisa Agrimensura LTDA” / e-mail da empresa | Baixa (padronização) |
| 9 | Integração com a suíte | Sem chaves `precisa_suite`, `precisa_label`, `precisa_submenu` etc. | O plugin registra menu próprio; não aparece no menu da Suíte Precisa | Informativo |
| 10 | `qgisMinimumVersion` | 3.28 | Nenhuma API exclusiva de 3.28 encontrada; valor é conservador. Consistente | — |
| 11 | `category` / `icon` / `experimental` | Raster / icon.svg / True | Consistentes com o código | — |

**Resumo:** metadata **com divergências** — principal ponto é a descrição da
fonte CBERS-4A WPM (declarada como Earth Engine/Asset do usuário; o código usa
INPE/BDC com token).
