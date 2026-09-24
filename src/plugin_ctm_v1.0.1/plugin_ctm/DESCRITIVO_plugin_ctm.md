# Descritivo Técnico-Funcional — CTM/BCI

**Pasta:** `plugin_ctm` · **Versão declarada:** 1.0.1 (experimental) · **QGIS:** 3.34 – 3.99
**Arquivos lidos:** `__init__.py`, `plugin_ctm.py`, `ui/dialog_ctm.py`,
`core/topologia.py`, `core/bci.py`, `core/exportador.py`, `metadata.txt`

---

## 1. Arquitetura
| Módulo | Responsabilidade |
|---|---|
| `plugin_ctm.py` | Barra de ferramentas própria “CTM/BCI”, menu, ciclo de vida, disparo das tarefas de topologia e referência forte às `QgsTask` |
| `ui/dialog_ctm.py` | Diálogo com 4 abas (Topologia, BCI, Atributos Complementares, Revisão/Exportação) |
| `core/topologia.py` | `ValidadorTopologico(QgsTask)` — um modo por tarefa |
| `core/bci.py` | `MotorBCI(QgsTask)` e dataclasses `DadosBCI`, `DadosEdificacao`, `DadosInfraestrutura`, `DadosSituacaoJuridica` |
| `core/exportador.py` | `ExportadorBCI` — HTML e PDF (QTextDocument + QPrinter, A4, margens 18 mm) |

Tradução: procura `i18n/plugin_ctm_<locale>.qm` (pasta **não existe** no pacote).

## 2. Funcionalidades implementadas

### 2.1 Topologia
- **Overlaps:** índice espacial; para cada par candidato, `intersects` +
  `intersection`; só conta interseção **poligonal** com área > 0.
- **Gaps:** `unaryUnion(limite) − unaryUnion(lotes)`; partes < 0,01 m² descartadas.
- **Geometrias inválidas:** `validateGeometry(ValidatorGeos)`; ponto do erro
  ou `pointOnSurface` quando o GEOS não informa posição; geometria vazia reportada.

### 2.2 Cálculo do BCI (por lote)
- **Testadas:** `boundary(lote).buffer(tolerância_contato)` ∩ linha do
  logradouro → comprimento. Maior comprimento = **testada principal**
  (critério “clássico”, declarado no código como distinto do DANI); demais =
  secundárias; lote com mais de uma = esquina.
- **Dados físicos:** área e perímetro da geometria; profundidade média =
  área / testada principal.
- **Confrontantes:** lotes a distância ≤ tolerância de vizinhança (não
  `touches()`), filtrados pela lista “ignorar”. Classificação pelo azimute
  centroide→vizinho vs. azimute da rua (1º e último vértice da linha):
  diferença ≥ 135° = fundo; senão esquerda/direita pelo produto vetorial;
  laterais extras vão para “fundo”. Sem testada → todos em “fundo”.
- **Inscrição:** máscara `{distrito}.{setor}.{quadra}.{lote}` com
  *zero-padding* por componente (extrai só os dígitos); vazio → `?`
  (lote vazio → ID interno).
- **Complementares:** leitura direta das colunas mapeadas (texto).

### 2.3 Exportação
- HTML com CSS próprio; blocos 1–6; testadas secundárias; linha de
  assinatura; rodapé “Portaria MDR nº 3.242/2022 … não substitui certidão de matrícula”.
- PDF individual, PDF único (conteúdo de `<body>` concatenado com
  `page-break-after`) ou um PDF por lote (`BCI_<inscrição com _>.pdf`).
- Confrontantes **não** são impressos no boletim (decisão documentada no código).

## 3. Normas e referências citadas no código
| Referência | Onde | Uso |
|---|---|---|
| **Portaria MDR nº 3.242/2022** (diretrizes do CTM) | `bci.py`, `exportador.py`, rodapé do boletim | Referência normativa declarada; art. 24 citado para o bloco de situação jurídica (intercâmbio CTM ↔ RI) |
| Portaria Min. Cidades 511/2009 | `bci.py`, `exportador.py` | Citada **como revogada** pela 3.242/2022 |
| Lei 6.015/1973 | `exportador.py` | Citada para justificar **não** incluir confrontantes no BCI |

A **Lei 13.465/2017 (REURB) não é citada no código** — o plugin só tem o campo
opcional “Situação REURB”. Não há implementação de regra específica da REURB.
A máscara de inscrição é declarada como “prática consolidada”, sujeita ao
Código Tributário de cada município.

## 4. Dependências
Somente QGIS/PyQt (QtPrintSupport para PDF). Nenhuma biblioteca externa.

## 5. Limitações conhecidas e pontos de atenção
1. **Bug — campos opcionais bloqueiam o cálculo.** Em
   `_disparar_calculo_bci`, a checagem percorre **todas** as chaves do
   mapeamento (23), inclusive edificação, infraestrutura e situação jurídica.
   Qualquer campo complementar em “— nenhum —” impede o cálculo, com mensagem
   chamando-os de “obrigatórios”. Contradiz o modo “BCI Simplificado”
   documentado no próprio código e na interface.
2. **Aba errada após o cálculo:** `setCurrentIndex(2)` abre *Atributos
   Complementares*; a *Revisão/Exportação* é o índice 3.
3. **Camada de Quadras não é usada:** recebida pelo `MotorBCI`, mas nenhum
   cálculo a utiliza (a docstring diz que o motor “cruza lotes, quadras e logradouros”).
4. **Extensão de testada superestimada:** o comprimento da rua dentro do
   *buffer* do contorno inclui até 2 × tolerância além dos cantos, e ruas
   transversais que tocam o *buffer* no canto geram testadas secundárias
   curtas (falsas esquinas) — tanto maior quanto maior a tolerância.
5. **Orientação da rua simplificada:** azimute da reta 1º→último vértice da
   linha (ou da parte cujo 1º vértice está mais perto); em ruas curvas ou
   longas, a classificação esquerda/direita/fundo pode errar.
6. **Sem verificação de CRS:** área, perímetro, tolerâncias e testadas estão
   na unidade do CRS; em CRS geográfico, todos os números saem em graus.
7. **Acesso à camada dentro da `QgsTask`:** `getFeatures()`/`getFeature()`
   são chamados na thread secundária diretamente sobre a camada (não sobre
   uma cópia/`QgsVectorLayerFeatureSource`), prática que o QGIS não garante
   ser *thread-safe* — risco com edição simultânea.
8. **PDF com CSS limitado:** `QTextDocument` não suporta `flex`, `@page`,
   `border-radius` e parte do CSS usado — o PDF não fica igual ao HTML
   (cabeçalho/brasão).
9. Inscrição com `?` (campo vazio) gera nome de arquivo inválido no Windows
   na exportação por lote; inscrições repetidas sobrescrevem o PDF.
10. `DadosPossuidor.uso_do_solo` e `padrao_construtivo` existem, mas a
    interface não permite preenchê-los.
11. Conexões `layersAdded/layersRemoved` do diálogo nunca são desconectadas;
    no `unload` a barra de ferramentas só tem a referência apagada (`del`).
12. Nenhum comentário marca trecho como “não testado”; o metadata marca o
    plugin como experimental.
