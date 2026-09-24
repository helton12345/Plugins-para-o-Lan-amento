# Divergências metadata.txt × código — DANI v6.6

Este relatório compara o `metadata.txt` com o código. O `metadata.txt` **não foi editado**.

## Divergências encontradas

| # | Chave / trecho do metadata | O que diz | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `description` e `about` | "DANI v6.1" | `version=6.6` no próprio metadata. A janela mostra "v5.03", o statusTip "v5.01" e o PreviewDialog "v3.0" | Baixa (confunde o suporte) |
| 2 | `qgisMinimumVersion` | 3.0 | Usa `Qgis.TextRenderFormat.AlwaysText` (com fallback), `QgsLayoutExporter` SVG/PDF, `setLabeling(None)` etc. Não há verificação de versão, e o mínimo real não foi determinado. **[PROVÁVEL]** acima de 3.0 | Média |
| 3 | `about`, v6.1 Item 5 | "Removida a opção de puxar ÁREA/PERÍMETRO de campo externo… sempre calculados da geometria" | `layer_setup_dialog` ainda tem **"Usar Área da tabela de atributos"** (`usar_area_tabela`). O PREPEND injeta `USAR_AREA_TABELA` e os três scripts usam a área do campo quando a opção está ligada. As abas de Ferramentas fixam `preferir_campo=False` | Alta (o comportamento contradiz a documentação) |
| 4 | `about`, v6.1 Item 5 | Área gravada "de volta nos campos AREA/PERIMETRO" | Confirmado, mas o metadata não avisa que o **valor original do campo AREA é sobrescrito** e a camada é salva (commit) | Média |
| 5 | Histórico v5.17, Item 1 | "Diálogos não-modais — canvas acessível (zoom/pan)" | `getItem_canvas`/`getText_canvas` usam `exec_()`, ou seja, são modais na prática **[PROVÁVEL]**. Só "Validar lados", "Lote de esquina" e a classificação manual usam `QEventLoop` sem modal | Baixa |
| 6 | Histórico v5.17, Item 11 | "Tabela de Áreas em XLSX com fórmulas =SUM" | Confirmado, mas o TOTAL GERAL soma lotes + subtotais, e o total sai **em dobro** | Alta |
| 7 | Histórico v5.17, Item 17 | "Sessão com auto-save e 5 versões de backup" | O autosave grava só na 1ª vez por sessão do QGIS, e `save()` está desativado. Salvar "Configurar Camadas" apaga os Dados do Loteamento | Alta |
| 8 | Histórico v5.17, Item 10 | "Plantas salvas na pasta configurada (não mais no Desktop)" | Só vale com a pasta de saída preenchida. Sem ela, tudo vai para `~/Desktop` (ou `~`) | Baixa |
| 9 | v6.0 | "Toda área e todo perímetro publicado… bate até a 2ª casa decimal com o memorial" | Com "Usar Área da tabela", o TXT da PLANILHA usa a área do campo, e os dados de exportação usam a área calculada. O GEO e o TABULAR/PLANILHA têm detectores de curva diferentes; o perímetro só é igualado quando o GEO roda antes | Média |
| 10 | Histórico v5.17, Item 16 | "Dados do Loteamento inseridos uma vez; substituídos no docx" | Vale só para o .docx. TXT, HTML e CSV dos scripts têm RT fixo "Helton José Carmanini Lourenço – CREA-MG 141.370/D" e "Guiricema" (HTML e CSV são descartados; o TXT fica quando o preview está ligado) | Baixa |
| 11 | Dependências | Não declaradas (o metadata não tem campo `plugin_dependencies`, nem instruções além de "pip install pygeomag" no changelog 6.2) | O código exige **python-docx, openpyxl, matplotlib e numpy**. O pygeomag é opcional, e sem ele o plugin tenta a NOAA via internet (o changelog diz que sai "N/D") | Média |
| 12 | Changelog 6.2–6.6 | Documenta as mudanças de 6.2 a 6.6 | O `about` só descreve a v6.1 e as anteriores. As versões 6.2–6.6 ficam em comentários `;` que o QGIS não exibe | Baixa |
| 13 | `icon=icon.png` e `precisa_icon=icon.png` | — | `icon.png` existe na raiz. OK | — |
| 14 | `experimental=False` | Versão estável | Há bugs confirmados de perda de dados (itens 3, 6 e 7 desta tabela) | Opinião: revisar |
| 15 | `tracker=` e `repository=` | Vazios | — | Baixa (o repositório oficial do QGIS exige os dois) |

## Consistente

- `name`, `author`, `email`, `category`, `tags` e as chaves `precisa_*` da suíte.
- `icon.png` existe.
- O submenu "📁 Projetos" é coerente com a função do plugin.

**Resultado:** há divergências (15 itens, 3 de gravidade alta).
