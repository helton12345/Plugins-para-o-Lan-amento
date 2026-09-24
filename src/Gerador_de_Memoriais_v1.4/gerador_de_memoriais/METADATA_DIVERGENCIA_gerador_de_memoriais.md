# Divergências do metadata.txt — Gerador de Memoriais

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código / pacote mostra | Gravidade |
|---|---|---|---|---|
| 1 | `about` (modo Texto) | “cole ou carregue um memorial descritivo narrativo” | Correto; não cita que desde a v1.4 aceita **.pdf**, remove timbre/rodapé repetido e avisa vértices não reconhecidos, nem a detecção automática de fuso (v1.3) | Baixa |
| 2 | `about` | “Não inclui classificação … — disponível nos produtos **DANI e GEO RURAL**” | O `FAQ.md` e o `README.md` dizem “**DANI e GEO URBANO**” para o mesmo recurso. Inconsistência entre documentos do próprio pacote | Baixa |
| 3 | `about` (modo SIGEF) | “leia o PDF oficial do SIGEF/INCRA e gere memorial em DOCX, pontos e polígono” | Vértices do tipo **marco (`-M-`) não são lidos** pela regex; o título do memorial é fixo “DIVISÃO DE IMÓVEL RURAL” | Média (funcional) |
| 4 | Dependências | Não declaradas | Requer `python-docx` e `pdfplumber` (documentados só no README/FAQ) | Média |
| 5 | `changelog` | “Ver CHANGELOG.md” | O Gerenciador de Complementos mostra esse texto literalmente; o `CHANGELOG.md` tem as entradas **fora de ordem** (1.4, 1.3, 1.2, 1.0, **1.1**) | Baixa |
| 6 | `version` | 1.4 | Consistente: `metadata.txt`, `CHANGELOG.md` e `parser_texto.py` datados de 23/09 14:43; `dialog.py` 23/09 14:39 — correspondem às mudanças listadas na 1.4 | — |
| 7 | Template de issue | — | `.github/ISSUE_TEMPLATE/bug_report.md` sugere “Versão do plugin: 1.0” | Informativo |
| 8 | `author` / `email` | Helton J. C. Lourenço / e-mail pessoal | Plugin gratuito publicado separado da suíte (sem chaves `precisa_*`); padrão diferente dos plugins da empresa | Informativo |
| 9 | `qgisMinimumVersion` | 3.16 | `camadas.salvar_shp` usa `writeAsVectorFormatV3` com *fallback* para V2 — compatível com 3.16 | — |
| 10 | `category` / `icon` / `tags` / `tracker` / `repository` / `license` | Preenchidos | Consistentes | — |

**Resumo:** metadata **com divergências menores**. É o metadata mais completo
da suíte até aqui; os pontos principais são as dependências não declaradas, a
inconsistência DANI/GEO RURAL × GEO URBANO e o `about` não refletir as
limitações reais do modo SIGEF.
