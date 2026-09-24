# Divergências do metadata.txt — Precisa Esgoto

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | Changelog 0.2.4/0.7 (comentários) | Orientação como árvore até o nó de entrega, com orçamento de subida de 2 m | Existe em `core/topologia.py`, mas o diálogo **nunca passa `no_descarga`**: a orientação é sempre por cota | Média |
| 2 | Changelog 0.2.5 | Recobrimento de 0,20 m na servidão | `recobrimentos` nunca é passado pelo diálogo: vale sempre o recobrimento único | Média |
| 3 | `about` | “declividade por autolimpeza (tensão trativa) ou pelo terreno … lâmina máxima 0,75” | Correto, mas quando nenhum DN comporta a vazão a lâmina é fixada em 0,75 sem alerta de sobrecarga (Descritivo, item 2) | Média |
| 4 | `icon` | Vazio | O `plugin.py` não usa ícone. `precisa_icon=icon.png` existe | Baixa |
| 5 | `about` | “CRS padrão SIRGAS 2000 / UTM 23S (EPSG:31983)” | Aceita qualquer SRC projetado. EPSG:31983 é só o *fallback* quando a camada não tem authid. O MDT não é reprojetado | Baixa |
| 6 | `description` / `about` | Não menciona | Segmentação automática, população por lotes, edição de DN/profundidade, inversão de sentido, perfil interativo, recálculo automático, CSV, DXF (perfil e planta), memória DOCX | Baixa |
| 7 | Dependências | Não declaradas | python-docx 1.2.0 embarcado (requer `typing_extensions`, que não está embarcado) | Baixa |
| 8 | `tracker` / `repository` | `https://example.invalid/...` | URLs de exemplo, inválidas | Baixa |
| 9 | Changelog 1.0 | “já entregues na drenagem 1.7/2.0” | O pacote de drenagem recebido é a **0.9.1** | Informativo |
| 10 | `version` / datas | 1.0 | `metadata.txt` e o código principal de 19/09. Consistente | — |
| 11 | `qgisMinimumVersion` | 3.16 | APIs usadas (QgsMapLayerComboBox, QgsRuleBasedRenderer etc.) disponíveis. Consistente | — |

**Resumo:** metadata **com divergências**:
- o changelog descreve capacidades de núcleo que a interface não usa;
- o ícone está ausente;
- as URLs são de exemplo.
