# Divergências do metadata.txt — Precisa Sedimentação

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `qgisMinimumVersion` | 3.16 | Usa `QgsVectorFileWriter.writeAsVectorFormatV3` (máscara e reprojeção), disponível só a partir do **QGIS 3.20** | Alta |
| 2 | `description` | “…com curva CAV, perfil longitudinal e série temporal opcionais” | Curva CAV só está disponível na **Forma A**; série temporal só nas Formas A e B | Baixa |
| 3 | `precisa_tooltip` e `DESCRICAO` (plugin.py) | “Assoreamento e pacote sedimentar” | Omite a **Forma B (barragem de rejeito)**, presente no código e na `description` | Baixa |
| 4 | Dependências | Não declaradas | `numpy`, `openpyxl`, `python-docx`, `matplotlib`, GDAL (checados em `plugin.run` apenas se a suíte estiver instalada) | Média |
| 5 | `about`, `category`, `experimental`, `tracker`, `repository` | Ausentes | — | Baixa |
| 6 | Normas | Metadata não cita | O código cita Res. ANA/ANEEL 127/2022, ANM Res. 220/2025 e PNSB; o metadata só diz “PNSB” | Informativo |
| 7 | Nome | “Precisa — Sedimentação” | Pacote enviado como “precisa_sedimentacao_pro_v1.2”; `serie_temporal.py` cita “spec do Precisa Batimetria Pro” — nomenclatura “Pro” não aparece no metadata | Informativo |
| 8 | `version` | 1.2 | Datas coerentes: núcleo em 08/08, `plugin.py` 14/08 11:11, metadata 14/08 11:33 | — |
| 9 | `icon` | icon.png | Presente | — |

**Resumo:** metadata **com divergências** — principal é a versão mínima do
QGIS (3.16 × 3.20 exigido); tooltip sem a Forma B e dependências não declaradas.
