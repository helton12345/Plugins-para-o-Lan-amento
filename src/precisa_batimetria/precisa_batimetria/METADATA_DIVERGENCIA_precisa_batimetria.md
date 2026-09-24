# Divergências do metadata.txt — Precisa Batimetria CAV

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `qgisMinimumVersion` | 3.16 | `calculator._mask_polygon` usa `QgsVectorFileWriter.writeAsVectorFormatV3`, que **só existe a partir do QGIS 3.20**. Em 3.16–3.18 o cálculo falha | Alta |
| 2 | `description` | “Curva Cota-Área-Volume para reservatórios e lagoas” | Também gera MDT por TIN, curvas de nível, perfil longitudinal, 3 seções transversais, planilha Excel e **laudo técnico DOCX com relatório fotográfico** | Média |
| 3 | `about` | Ausente | Campo recomendado pelo QGIS não existe | Baixa |
| 4 | `icon` | **Ausente** (só `precisa_icon=icon.png`) | O Gerenciador de Complementos do QGIS usa `icon=`; sem ele o plugin aparece sem ícone lá | Média |
| 5 | `category`, `experimental`, `deprecated`, `tracker`, `repository` | Ausentes | — | Baixa |
| 6 | Dependências | Nenhuma declarada | Usa `numpy`, `openpyxl`, `python-docx`, `matplotlib` e, opcionalmente, o pacote `precisa_agrimensura_suite` (e, na prática, **exige** a suíte por causa do bug em `dialog._log`) | Média |
| 7 | `precisa_tooltip` e `DESCRICAO` em `plugin.py` | “… a partir de curvas de nível” | O código aceita também pontos cotados, inclusive sem curvas | Baixa |
| 8 | `version` | 1.0 | Todos os `.py` com data 18/05/2026; `metadata.txt` e `icon.png` com 21/07/2026 — o metadata é posterior ao código. Sem conflito aparente, mas sem registro do que mudou em 21/07 | Informativo |
| 9 | `name` | “Precisa — Batimetria CAV” | Janela: “Precisa Batimetria CAV — Curva Cota-Área-Volume”; ação: “Batimetria CAV”. Consistente | — |

**Resumo:** metadata **com divergências** — a mais relevante é a versão mínima
do QGIS (3.16 declarada × 3.20 exigida pelo código), seguida da ausência do
campo `icon` e da descrição incompleta.
