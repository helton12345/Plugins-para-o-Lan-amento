# Descritivo Técnico-Funcional — Precisa Batimetria CAV

**Pasta:** `precisa_batimetria` · **Versão declarada:** 1.0 · **QGIS mínimo declarado:** 3.16
**Arquivos lidos:** `__init__.py`, `plugin.py`, `_plugin_orig.py`, `dialog.py`,
`core/calculator.py`, `core/report_excel.py`, `core/report_docx.py`, `metadata.txt`

---

## 1. Arquitetura

| Módulo | Responsabilidade |
|---|---|
| `__init__.py` | `classFactory` (QGIS) e `run(iface)` (chamado pela Suíte) com instância única |
| `plugin.py` | Ação no menu (usa `MENU_NAME` da suíte ou `&Precisa Agrimensura`), registro no *launcher* da suíte, checagem de dependências via `precisa_agrimensura_suite.core.deps` |
| `_plugin_orig.py` | Cópia antiga de `plugin.py` — **não é importada** (arquivo morto) |
| `dialog.py` | Diálogo em 4 abas; `BatimetriaWorker` (usa `SafeWorker` da suíte ou `QThread` próprio) |
| `core/calculator.py` | `BatimetriaCalculator`: MDT, CAV, regressão, estatísticas, gráficos, curvas, orquestração |
| `core/report_excel.py` | `gerar_excel` (openpyxl) |
| `core/report_docx.py` | `gerar_docx` (python-docx; usa `docx_helpers` da suíte se existir) |

Dependência opcional do pacote **`precisa_agrimensura_suite`** (log, constantes
AUTHOR/CREA/EMPRESA/MENU_NAME, launcher, deps, thread_utils, docx_helpers) —
com *fallback* local quando ausente (ver limitação 1).

## 2. Funcionalidades implementadas

### 2.1 MDT (`_create_mdt`)
- `QgsTinInterpolator` com curvas (`SourceStructureLines`) e/ou pontos (`SourcePoints`).
- Extensão = *bounding box* do polígono do NA; nº de colunas/linhas =
  extensão / resolução (mínimo 10 × 10).
- Gravado por `QgsGridFileWriter` em `<prefixo>_mdt.tif`.
- Campo de cota não encontrado → usa o **índice 0** (primeiro campo) sem aviso.

### 2.2 Curva CAV (`_calcular_cav`)
- Leitura do MDT com GDAL; NoData → NaN; máscara pelo polígono do NA
  (rasterização GDAL de um shapefile temporário).
- `n + 1` cotas igualmente espaçadas de `cota_minima` a `cota_na`.
- Área(cota) = nº de pixels com Z ≤ cota × área do pixel.
- Volume acumulado pelo **método das áreas médias (trapezoidal)**:
  V_i = V_{i-1} + (A_{i-1} + A_i)/2 · Δh; V = 0 na cota mínima.
- Saídas por linha: cota, área (m², ha), volume (m³, hm³).

### 2.3 Regressão e estatísticas
- `numpy.polyfit` grau 2 para área e volume (só faixas com área > 0), R² calculado.
- Profundidade máxima = `cota_na − cota_minima` (valores digitados).
- Profundidade média = V_total / A_NA.

### 2.4 Gráficos (matplotlib, backend Agg, 150 dpi)
- CAV: X = cota; Y esquerdo = área; Y direito = volume **invertido**.
- Perfil longitudinal: se a extensão do polígono é mais larga que alta, usa a
  **linha de pixels** com mais células válidas; caso contrário, a **coluna**
  com mais células válidas. Seções transversais perpendiculares em 25/50/75 %
  dos pixels válidos do perfil. Linha do NA e preenchimento da lâmina d'água.

### 2.5 Curvas de nível do MDT
- `processing.run('gdal:contour')` (campo `ELEV`, intervalo configurável);
  *fallback* `gdal.ContourGenerate`. Resultado carregado no projeto.

### 2.6 Planilha Excel
- Aba “Curva CAV”: cabeçalho com NA, volume, área, profundidades; tabela com
  nº, cota, área (m², ha), volume (m³, hm³), % de cota; linha “VOLUME TOTAL”
  com fórmula; equações e R²; gráfico de linhas nativo com eixo secundário de
  volume invertido (`orientation = maxMin`).
- Aba “Gráfico CAV” com o PNG do matplotlib (se existir).

### 2.7 Laudo DOCX
Seções: título; tabela de identificação; 1 Introdução; 2 Objetivo;
3 Localização; 4 Metodologia (4.1 Equipamentos, 4.2 Procedimento de campo,
4.3 Processamento); 5 Resultados (5.1 Mapa batimétrico, 5.2 CAV com Figura 1
e Tabela 1, 5.3 Perfil e seções, 5.4 Tabela 2 por percentual de cota
0–100 % via equações, com equações e R²); 6 Conclusão (fórmula
Pm = Va/Ala e Tabela 3 resumo); 7 Relatório fotográfico (2 fotos por linha
com legenda); 8 Anexos (lista A–E); bloco de assinatura (município/data,
RT, título, CREA, ART). Rodapé profissional se a suíte estiver presente.

## 3. Normas técnicas referenciadas
**Nenhuma norma (NBR, ANA etc.) é citada no código.** O laudo contém a frase
genérica “seguindo as melhores práticas e normas técnicas aplicáveis”, sem
norma identificada. Método de volume: áreas médias (trapezoidal), descrito no
texto do laudo como “integração numérica (método trapezoidal)”.

## 4. Dependências
| Dependência | Uso | Verificada pelo plugin? |
|---|---|---|
| QGIS (`qgis.analysis` TIN, `processing`, GDAL/OGR) | MDT, máscara, curvas | — |
| `numpy` | Cálculos | Sim (se a suíte existir) |
| `openpyxl` | Excel | Sim (se a suíte existir) |
| `python-docx` | Laudo | Sim (se a suíte existir) |
| `matplotlib` | PNGs | **Não** — ausência só pula os gráficos |
| `precisa_agrimensura_suite` | Log, launcher, constantes, thread, helpers DOCX | Opcional |

## 5. Limitações conhecidas e pontos de atenção

1. **Bug — não funciona sem a Suíte Precisa instalada.** Em `dialog.py`,
   `_log()` faz `from precisa_agrimensura_suite.core.log import info` e, no
   `except ImportError`, chama `error(...)`, que **não está definido** nesse
   módulo → `NameError`. Como `_run()` chama `_log('Iniciando processamento…')`
   antes de criar o worker, o cálculo não inicia quando a suíte está ausente.
2. **Opção “Exportar raster MDT (GeoTIFF)” sem efeito**: o parâmetro
   `gerar_raster` é enviado mas nunca lido; o MDT é sempre gravado.
3. **Sem reprojeção entre camadas:** a máscara grava o polígono com
   `QgsCoordinateTransformContext()` vazio e a rasteriza sobre o MDT; se o
   polígono estiver em CRS diferente das curvas/pontos, a máscara fica
   deslocada (áreas zero ou erradas). Resolução e intervalos assumem CRS em metros.
4. **Cota mínima e profundidade máxima vêm do usuário**, não do MDT. Se o MDT
   tiver pixels abaixo da cota mínima informada, a área já aparece na primeira
   linha com volume 0 (volume abaixo dela é descartado).
5. **Regressão exige ≥ 3 faixas com área > 0**; caso contrário `polyfit`
   falha e o processamento aborta.
6. **Perfil longitudinal alinhado aos eixos do raster** (linha ou coluna de
   pixels), não ao eixo real de maior comprimento da lagoa; lagoas diagonais
   têm perfil subestimado. A seção rotulada “Fim” fica a 75 %, não no fim.
7. **Textos fixos no laudo:** método por corda graduada + GNSS RTK, software
   GTopo, “lagoa artificial”, uso agropecuário, recomendação de 3–5 anos; cotas
   rotuladas “Datum SIRGAS 2000” (referencial altimétrico não é tratado).
   Anexos A–E apenas listados, não gerados.
8. Valores padrão de NA/fundo (522,908 / 517,891 m) herdados de um projeto real.
9. Arquivos temporários (`tempfile.mktemp`) do shapefile e da máscara não são
   apagados; a máscara é recalculada duas vezes (CAV e perfil).
10. Com a suíte presente, o caminho de *worker* depende de `SafeWorker`
    (sinais `erro`, `isRunning`, `start`), que **não faz parte deste pacote** e
    não pôde ser verificado.
11. `_plugin_orig.py` é código morto; difere de `plugin.py` só no tratamento de log.
12. Nenhum comentário no código marca trechos como “não testados”; os itens
    acima vêm da leitura do código.
