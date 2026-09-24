# Divergências metadata.txt × código — Precisa Terraplanagem Suite v2.4.0

Este relatório compara o `metadata.txt` com o código. O `metadata.txt` **não foi editado**.

## Divergências encontradas

| # | Chave / trecho do metadata | O que diz | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `changelog` 2.0.0, "VOLUME M4 EXCLUI SECOES NAO CONFIAVEIS" | O M4 pula seções não confiáveis | O motor (`m4_volumes.py`) pula, mas o campo `secao_confiavel` não é gravado pelo M3 em `secoes_json` e é descartado pelo diálogo do M4. **No fluxo manual, a exclusão nunca acontece.** | Alta |
| 2 | `about` "M5 Relatório" e `description` | Relatório final completo | O relatório lê chaves do quadro que o M4 manual não grava. Áreas, volume homogeneizado, saldo acumulado e Brückner por via saem zerados ou planos. O "Resumo de platôs" nunca tem dados (`platos_resumo_json` não é gravado por nenhum módulo). | Alta |
| 3 | `changelog` 2.3.1 e 2.3.0: "540 passed", "testes/conftest.py", "testes/test_*.py" | Suíte de 540 testes | A pasta `testes/` **não está no pacote**. Só existe `engine/tests`: **380 passed** nesta revisão; o metadata diz 374 para o motor. | Média |
| 4 | `supportsQt6=yes`, `qgisMaximumVersion=4.99` | Compatível com Qt6/QGIS 4 | Restam 2 `exec_()` (tela da IA), 9 usos de `QVariant.*` e importações de `backend_qt5agg`, estas com alternativa. Não testado no QGIS 4. | Média |
| 5 | `about` ② "Terraplanagem Adaptativa (DIP) — Classificação de declividade…" | Classificação utilizável | A unidade padrão é **Graus** e os intervalos padrão são em **%**: sem trocar a unidade, a classificação sai errada. As ações da IA na DIP (estratégia e talude por classe) não afetam o processamento. `np.trapz` quebra as seções do relatório DIP no NumPy 2.4 (confirmado por teste). | Alta |
| 6 | `about` ④ "Assistente IA — modo offline (Ollama) e online (Claude / OpenAI)" | Três provedores | Os três existem, mas o orquestrador **exige chave Claude** mesmo com OpenAI escolhido. A chave da OpenAI (e a do Claude, sem senha-mestra) fica em texto claro no QSettings. Os IDs de modelo (`claude-opus-5`, `claude-opus-4-8`…) estão fixos no código **[A CONFIRMAR]**. | Média |
| 7 | `description` / `about` / `precisa_tooltip`: "NBR 13133:2021 \| Lei 6.766/79" e "Conforme: NBR 13133:2021 \| NBR 9732 \| Lei 6.766/79 \| GRAPROHAB/SP" | Conformidade normativa | São citações. O próprio código avisa que os padrões **27%** de declividade e **talude 1:1 em aterro** vão além do usual e exigem conferência municipal e geotécnica. O M3 manual usa 1,5/2,0 e a configuração usa 1:1 (padrões divergentes). **[A CONFIRMAR]** nas normas. | Média |
| 8 | `about` ③ "Bacias Hidrográficas — GRASS r.watershed \| Morfometria \| TWI \| TPI \| … \| Hidrologia" | Hidrologia completa | Existe, com limites: a intensidade I é digitada (sem IDF), o Método Racional não tem limite de área e a ordem de Horton é estimada. Requer GRASS. | Baixa |
| 9 | `version=2.4.0` | Versão 2.4.0 | Janela principal "Precisa Terraplanagem Suite **v1.0**"; DIP "Precisa Terraplanagem **v3.5.2**", também no texto do relatório DOCX; `core/__init__` `__version__ = "4.0.1"`. | Baixa |
| 10 | `icon=icon.png` e `precisa_icon=icon.png` | Ícone do plugin | `icon.png` existe e é usado pela Suíte. O `plugin.py` procura `icon.svg`, que não existe: a ação da barra fica sem ícone. | Baixa |
| 11 | `tracker`, `repository` e `homepage` = site da Precisa | Links de suporte e repositório | Apontam para o site comercial, não para um repositório de código ou rastreador de issues. O repositório oficial de plugins do QGIS exige um repositório de código público. | Baixa |
| 12 | `category=Plugins` | Categoria | O plugin não registra entrada em Vetor, Raster etc.; o menu vem da Suíte. OK, apenas genérico. | — |
| 13 | Dependências (não há chave no metadata) | — | O código usa numpy, scipy, GDAL, **matplotlib**, **python-docx**, **reportlab**, **openpyxl**, **osqp** (DIP), ezdxf (opcional), shapely e Pillow, além do **GRASS** para as Bacias. Sem scipy, a aba DIP não carrega. | Média |
| 14 | `experimental=False` | Versão estável | Há bugs confirmados de gravidade alta (itens 1, 2 e 5; ver também o Descritivo §5: o orquestrador sobrescreve a configuração, alertas duplicados, CRS geográfico aceito). | Opinião: revisar |
| 15 | `about`, "CREA-MG 141.370/D \| Guiricema-MG" | Identificação do autor | Os mesmos dados (com CNPJ e nome do RT) ficam **fixos** no relatório DOCX da DIP, sem campo para outro responsável técnico. | Média |
| 16 | `changelog` 2.1.1, "Editor de Malha … dlg.show()" | Não modal | Confirmado (`suite_dialog._abrir_editor_malha` usa `show()`). OK. | — |
| 17 | `changelog` 2.0.5, `format_estaca` corrigido | Resto da estaca | Confirmado. Caso de borda: resto ≈ intervalo sai "N+020.0". | Baixa |

## Consistente

- `name`, `author`, `email`, `qgisMinimumVersion=3.16` (sem uso detectado de API mais nova, mas **não verificado**).
- `hasProcessingProvider=no`: confirmado, não há provedor.
- As chaves `precisa_suite`, `precisa_label`, `precisa_submenu` ("📁 Projetos"), `precisa_tooltip` e `precisa_icon` estão no bloco da Suíte, e a ponte `_precisa_bridge.py` está presente.
- Itens do changelog conferidos no código:
  - DXF padrão **R2000** (2.1.0);
  - `ExportarDXFDialog` com a capitalização correta (2.1.1);
  - combos de curvas projetadas e MDT na drenagem dos lotes (2.1.1/2.4.0);
  - "cobertura insuficiente" em `engine/lote.py` (2.4.0);
  - `_com_scroll` e `_garantir_dlg_terrapl` na DIP (2.3.0);
  - teste da IA com sinais Qt (2.0.2);
  - folga de recorte de 0,30 m nos cruzamentos e teto de alargamento de 1,5 (2.0.9);
  - lista de modelos Claude atualizada (2.0.7).
- As tags correspondem a funções existentes: terraplanagem, loteamento, estaqueamento, greide, seções, volumes, Brückner, DIP, declividade, bacias, hidrologia, watershed, GRASS, morfometria, TWI, IA, Ollama.

**Resultado:** há divergências (15 itens com divergência, 3 de gravidade alta; os itens 12 e 16 são apenas confirmação).
