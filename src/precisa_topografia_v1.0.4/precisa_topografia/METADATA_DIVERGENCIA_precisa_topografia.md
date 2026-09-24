# Divergências do metadata.txt — Precisa Topografia

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `about` | “cálculo e ajustamento de poligonal fechada com método Bowditch” | O ajuste existe, mas o cálculo tem **erro de indexação dos ângulos** (Descritivo, limitação 1) que impede o fechamento com dados corretos | Alta (funcional) |
| 2 | `precisa_icon` | `icon.svg` | **Não existe `icon.svg` na raiz do plugin** (só `icon.png`; os `.svg` estão nas subpastas dos módulos) | Média |
| 3 | `about` | Não menciona | Exportação CSV/DXF da caderneta, importação de coordenadas de camada, relatório próprio da poligonal/irradiação, manuais DOCX embarcados | Baixa |
| 4 | Dependências | Não declaradas | `python-docx` opcional para os relatórios DOCX | Baixa |
| 5 | `email` | Vazio | — | Baixa |
| 6 | `tracker` / `repository` / `homepage` | Vazios | — | Baixa |
| 7 | `category` | Plugins | Categoria genérica | Informativo |
| 8 | `version` | 1.0.4 | `metadata.txt` 14/08 11:24; `plugin.py` e plugins dos módulos 14/08 11:10–11:11; cálculos 10/08 — consistente. Os `manual.docx` embarcados dizem “Versão 1.0/1.0.0” e ainda citam “Plugin 15/16/17/18”, numeração antiga | Informativo |
| 9 | `qgisMinimumVersion` | 3.16 | APIs usadas disponíveis na 3.16; `core/compat.py` trata QVariant/QMetaType. Consistente | — |

**Resumo:** metadata **com divergências** — `precisa_icon` aponta para arquivo
inexistente e o `about` promete um ajuste de poligonal que hoje não fecha.
