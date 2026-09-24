# Descritivo Técnico-Funcional — Precisa Sedimentação

**Pasta:** `precisa_sedimentacao` · **Versão declarada:** 1.2 · **QGIS mínimo declarado:** 3.16
**Arquivos lidos:** `__init__.py`, `plugin.py`, `dialog.py`, `core/*.py`
(sedimentacao_calculator, interpolacao_mdt, validacao_srs, reamostragem,
diferenca_mdt, curva_cav, perfil, serie_temporal, graficos,
report_excel_sedimentacao, report_docx_sedimentacao), `metadata.txt`

---

## 1. Arquitetura
Derivado do Precisa Batimetria (mesmo padrão de worker, máscara e TIN, citado
nos comentários).

| Módulo | Função |
|---|---|
| `plugin.py` | Menu, registro no *launcher* da suíte, checagem de dependências (numpy, openpyxl, python-docx, gdal, matplotlib) quando a suíte existe; diálogo não modal |
| `dialog.py` | 6 abas, validação, `SedimentacaoWorker` (SafeWorker da suíte ou `QThread`) |
| `sedimentacao_calculator.py` | Orquestração: MDTs → SRC → grade → diferença → estatísticas → CAV/perfil/série → relatórios |
| `interpolacao_mdt.py` | TIN (`QgsTinInterpolator`) na extensão da poligonal |
| `validacao_srs.py` | Comparação de `authid` entre MDTs e poligonal; funções de reprojeção |
| `reamostragem.py` | `gdal.Warp` bilinear para a grade do MDT base |
| `diferenca_mdt.py` | ΔZ, volumes (líquido, positivo, negativo), espessuras, GeoTIFF de diferença |
| `curva_cav.py` | Curva CAV por MDT, comparação base × atual, volume/área numa cota |
| `perfil.py` | Amostragem de 201 pontos ao longo da 1ª feição da linha |
| `serie_temporal.py` | Regressão linear volume × ano decimal, projeção de vida útil |
| `graficos.py` | PNGs (matplotlib Agg, 200 dpi) |
| `report_*` | Excel (openpyxl) e DOCX (python-docx / helpers da suíte) |

## 2. Funcionalidades implementadas

### 2.1 Obtenção dos MDTs
Raster pronto (caminho da camada) ou TIN de curvas (linhas estruturais) e/ou
pontos; grade = extensão da poligonal / resolução (mín. 10 × 10).

### 2.2 SRC e grade
- Checagem de SRC entre os 2 MDTs e a poligonal (referência = 1º da lista);
  divergência → `SrsIncompativelError`.
- Grades diferentes (dimensão ou geotransform > 1 cm) → reamostragem bilinear
  do MDT atual para a grade do base.

### 2.3 Diferença (`calcular_diferenca`)
- Máscara pela poligonal (rasterização GDAL), NoData → NaN.
- ΔZ = Z_atual − Z_base; volume líquido = ΣΔZ × área do pixel; volumes
  positivo e negativo separados; espessuras média/máx./mín.; área = nº de
  células válidas × área do pixel.
- Saída `<prefixo>_diferenca.tif` (Float32, NoData −9999).

### 2.4 Estatísticas por modo
- **A:** perda % = ΔV / V₀ × 100; taxa = ΔV / anos.
- **B:** volume de rejeito = ΔV; resguardo operacional = crista − lâmina;
  borda livre de projeto = crista − nível máximo maximorum; taxa = ΔV/anos;
  lâmina residual: área e volume Σ(cota − Z) do MDT atual abaixo da cota da lâmina.
- **C:** volume extraível = soma de ΔZ > 0.

### 2.5 Curva CAV (Forma A)
Para cotas de `floor(Zmín/passo)·passo` a `ceil(Zmáx/passo)·passo`:
área(c) = nº células com Z ≤ c × área; volume(c) = Σ(c − Z) × área
(“Surface Volume”). Volume útil = V(NMO) − V(NMN), volume morto = V(NMN)
(interpolação linear). Comparação base × atual em cotas coincidentes, perda %.

### 2.6 Série temporal (Formas A e B)
Campanhas (data dd/mm/aaaa, volume) + a execução atual; regressão linear
(slope em m³/ano, R², erro-padrão do slope com ≥ 3 campanhas); vida útil =
(limite − volume atual)/slope, com intervalo pelo erro-padrão.

### 2.7 Relatórios
- **Excel:** aba de resumo por modo; “Curva CAV” completa; “Série Temporal”.
- **DOCX:** título por modo; dados de entrada e rastreabilidade; metodologia
  (texto padrão); resultados por modo; CAV (tabela amostrada ~15 linhas +
  gráfico); mapa de isopacas estilizado (rampa divergente, percentis 2–98);
  perfil comparativo; análise temporal; conclusão/parecer; assinatura.
  Numeração de seções automática.

## 3. Normas e referências citadas no código
| Referência | Onde | Uso |
|---|---|---|
| **Res. ANA/ANEEL 127/2022** | Rótulo da Forma A e docstring do relatório | Contexto do assoreamento hidrelétrico (nenhuma regra específica dela é implementada além do cálculo de volume/perda) |
| **PNSB / ANM** | Rótulo da Forma B, título do laudo | Citados genericamente (sem número de lei) |
| **ANM Resolução nº 220/2025** | Calculadora, diálogo e laudo | Distinção entre resguardo operacional e **borda livre** (definida pelo nível máximo maximorum) — implementada como dois indicadores separados |
| Relatório UHE Camargos (Rural Tech/CEMIG, 2016) e Camargos et al. (2019) — Maravilhas II | Docstring de `curva_cav.py` | Metodologia de curva CAV reproduzida |

Nenhuma NBR é citada.

## 4. Dependências
QGIS (TIN, GDAL/OGR), `numpy`, `openpyxl`, `python-docx`, `matplotlib`;
opcional: pacote `precisa_agrimensura_suite` (log, constantes, launcher,
deps, SafeWorker, helpers DOCX).

## 5. Limitações conhecidas e pontos de atenção
1. **Popup de reprojeção nunca é chamado:** `_resolver_srs_ou_cancelar` está
   implementado em `dialog.py`, mas nenhum fluxo o executa. Com SRC
   divergente, o cálculo falha na thread e a UI mostra só um erro genérico
   (o comentário do calculador diz que “a UI já deve ter tratado isso”).
2. **Falso positivo de SRC:** a comparação é por `authid`; um raster cujo
   WKT não resolve para EPSG é comparado pelo WKT completo e pode divergir
   de uma camada com o mesmo sistema identificado por EPSG.
3. **QGIS mínimo real é 3.20:** a máscara usa `writeAsVectorFormatV3`.
4. **Perda de capacidade “útil” usa o ΔV total** da poligonal (líquido,
   incluindo erosão), não o volume entre NMN e NMO.
5. **Curva CAV só na Forma A:** o docstring do calculador diz “disponível para
   Formas A e B”, mas a opção fica no grupo da Forma A e só é enviada nesse modo.
6. **Série temporal mistura grandezas:** o volume da execução atual é o ΔV
   entre os dois MDTs informados; se o MDT base não for o levantamento
   original, esse valor não é “acumulado” como os volumes históricos digitados.
7. **Forma C:** “Área da Poligonal de Licença” é, na verdade, a área de células
   válidas (interseção dos dois MDTs com a poligonal); a espessura média inclui
   células negativas, embora o volume extraível use só as positivas.
8. Cotas digitáveis de 0 a 9999 m; valor 0 significa “não informado”
   (NMN/NMO/crista/lâmina negativos ou zero não são aceitos).
9. Comparação de curvas CAV só em cotas **idênticas** (docstring fala em
   interpolação, não implementada).
10. Linha de perfil: só a 1ª feição; sem checagem de SRC.
11. Arquivos temporários da máscara não são removidos; máscara recalculada
    várias vezes.
12. Com a suíte presente, depende de `SafeWorker` (fora deste pacote).
13. Nenhum trecho marcado como “não testado” no código.
