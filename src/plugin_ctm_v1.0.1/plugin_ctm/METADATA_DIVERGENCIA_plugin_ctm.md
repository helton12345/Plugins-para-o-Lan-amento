# Divergências do metadata.txt — CTM/BCI

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `description` / `about` (norma) | “conforme **Portaria Min. Cidades 511/2009**” | O código declara como referência a **Portaria MDR nº 3.242/2022** e diz explicitamente que ela **revogou** a 511/2009; o rodapé do boletim cita a 3.242/2022 | Alta |
| 2 | `description` / `about` (REURB) | “em conformidade com … a REURB (**Lei 13.465/2017**)” | A Lei 13.465/2017 **não é citada nem implementada** no código; existe apenas um campo opcional “Situação REURB” | Alta (conformidade não comprovada no código) |
| 3 | `about` | “cálculo de … testadas e **confrontantes**” e “emissão do BCI” | Confrontantes são calculados e exibidos na tabela, mas **não saem no boletim** (decisão documentada em `exportador.py`) | Baixa |
| 4 | `about` | Não menciona | Boletim com blocos opcionais de edificação, infraestrutura e situação jurídica, PDF único ou por lote, brasão | Baixa |
| 5 | `version` / `changelog` | version=1.0.1; changelog só com **1.0.0** | Não há registro do que mudou na 1.0.1. Código: `bci.py` 06/08, `dialog_ctm.py` 06/08, `plugin_ctm.py` 14/08 11:09, metadata 14/08 11:13 | Média |
| 6 | `tags` | inclui `sigef` e `inrca` | Nada no código usa SIGEF; “inrca” parece erro de digitação de “incra” | Baixa |
| 7 | `email` | contato@precisaagrimensura.com.br | Difere dos e-mails usados nos demais plugins (`precisagrimensuratop@gmail.com`) | Informativo |
| 8 | `homepage` / `tracker` / `repository` | Vazios | — | Baixa |
| 9 | `qgisMinimumVersion` | 3.34 | Nenhuma API exclusiva da 3.34 encontrada (código compatível com versões anteriores). Consistente, apenas conservador | — |
| 10 | Dependências | Nenhuma | Correto: só usa QGIS/PyQt | — |
| 11 | `experimental` | True | Coerente com os bugs encontrados (ver Descritivo, limitações 1 e 2) | — |

**Resumo:** metadata **com divergências relevantes** — cita norma revogada
(511/2009) em vez da que o código usa (MDR 3.242/2022) e afirma conformidade
com a Lei 13.465/2017 sem implementação correspondente.
