# Correções aplicadas — DANI v6.6.1 a v6.6.5

Registro das correções feitas em cima do diagnóstico de `DESCRITIVO_DANI_v6_0.md`
e `METADATA_DIVERGENCIA_DANI_v6_0.md`, e da validação de desempenho feita depois,
com um loteamento real (144 lotes) cedido só para teste. Nenhum dado do cliente
(nome, CNPJ, endereço, coordenadas) está registrado aqui — só métricas.

## 1. Correções por versão

| Versão | Item do DESCRITIVO (§5) | Resumo |
|---|---|---|
| 6.6.1 | 1 | "Configurar Camadas" não apaga mais os Dados do Loteamento/RT |
| 6.6.1 | 2 | Autosave grava a cada fechamento, não só na 1ª vez |
| 6.6.1 | METADATA #6 | Tabela de Áreas XLSX: TOTAL GERAL não soma mais em dobro |
| 6.6.2 | 6 (parcial) | GEO/TABULAR/PLANILHA/Plantas passam a usar o mesmo detector de curva; início do memorial sempre no vértice mais ao norte, sentido horário; curva no início do anel não fica mais partida em duas nem some da frente |
| 6.6.2 | — | Relatório de Observações passa a trazer todos os casos especiais (antes descartava qualquer linha com "VERIFICAR") |
| 6.6.3 | 9, 13, 15, 17, 18, 19 | Perímetro de Plantas/Cotas e da Planta Geral passa a bater com o memorial; Declividade funciona com e sem camada de ruas; "Lote específico" aceita lote com letra (ex. "12A"); fuso por EPSG corrigido; trecho sem lado passa a gerar aviso em vez de sumir |
| 6.6.4 | 6, 9 (residual) | TABULAR/PLANILHA/Plantas passam a usar as mesmas distâncias impressas pelo memorial GEORREFERENCIADO por trecho (não só o perímetro total) — elimina divergência de ~1 cm entre documentos do mesmo lote |
| 6.6.5 | — | Correção de desempenho (ver seção 2) |

Itens do DESCRITIVO ainda abertos: 5 (filtro de ângulo dos confrontantes não
filtra — não modificado, é o núcleo marcado "NÃO MODIFICAR"), 11 (ABNT
12721), 12, 14, 16, 20–27 (interface, IA, empacotamento).

## 2. Correção de desempenho (v6.6.5)

**Causa raiz.** Desde a 6.6.2 (unificação do detector de curvas dos 4
scripts), a verificação periódica interna de `detectar_curva_multicriterio`
recalculava o raio médio sobre **todos** os pontos já acumulados da curva a
cada 20 pontos novos, em `Decimal` — custo O(n²) por curva, onde n é o número
de vértices da curva. Loteamentos com curva digitalizada em alta densidade
(comum em importação de CAD) ficavam lentos o suficiente para parecer
travados.

**Medido** num loteamento real de 144 lotes (mediana de 5 vértices por lote,
mas 9 lotes entre 2.798 e 6.736 vértices na curva mais densa): geração do
memorial GEORREFERENCIADO caiu de 88,5 s para 9,2 s.

**Correção.** A verificação periódica passou a usar `float` em vez de
`Decimal` nesse cálculo interno. As tolerâncias de decisão (20% no raio, 5°
no ângulo) são muitas ordens de grandeza maiores que a diferença entre
`float` e `Decimal` na escala de coordenada UTM usada — a decisão não muda.
O raio, o desenvolvimento e os demais valores impressos continuam calculados
como antes, com `Decimal`, no mesmo lugar.

**Validação.** Byte a byte, nos 179 lotes de um loteamento fictício de teste
e nos 144 lotes do loteamento real, GEO/TABULAR/PLANILHA/Plantas saíram
idênticos antes e depois da correção.

## 3. Validação independente contra geometria bruta

Além de comparar a saída do plugin contra si mesma, os números foram
conferidos contra a geometria bruta do shapefile do loteamento real, com
código que não compartilha nenhuma linha com o DANI (`shapely` para
área/perímetro; circuncentro de 3 pontos, calculado à mão, para ajuste de
curva):

- Perímetro: bate (±0,05 m) em 144 de 144 lotes.
- Área: bate (±0,05 m²) em 143 de 144; o único fora (diferença de 0,051 m²,
  lote de 530 m de perímetro) é compatível com o arredondamento de
  coordenadas a 3 casas antes do cálculo, comportamento documentado no
  DESCRITIVO (item 4).
- Curva de teste (raio nominal 16,00 m, desenvolvimento 12,30 m, 2.204
  pontos digitalizados): ajuste independente deu raio 15,9972 m,
  desenvolvimento 12,3007 m, desvio máximo de 0,15 mm entre os 2.204 pontos
  e o círculo ajustado.

Também foi conferido, e refutado com evidência linha a linha do memorial,
um relatório de auditoria de terceiros que apontava 9 supostas divergências
nesse mesmo loteamento (4 casos de "curva perdida" no TABULAR e 5 de
diferença de lado reto). Em todos os 9 casos, a soma dos trechos do
GEORREFERENCIADO bate exatamente com o valor do TABULAR; as divergências
apontadas vinham do script de auditoria (heurística própria de detecção de
curva por comprimento de segmento, não a do DANI), não do plugin.
