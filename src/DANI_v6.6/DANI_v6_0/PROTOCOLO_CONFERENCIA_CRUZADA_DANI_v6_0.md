# Protocolo de conferência cruzada dos produtos do DANI

Documento de metodologia para quem for auditar/conferir os produtos gerados
pelo DANI (Memorial GEORREFERENCIADO, TABULAR, PLANILHA, Plantas, Cotas)
contra a camada (shapefile) de origem. Escrito depois de duas rodadas de
auditoria externa no mesmo loteamento real terem apontado, juntas, 68
"divergências" (9 + 59) que na conferência linha a linha se mostraram falsos
positivos — não por má-fé, mas por 4 causas metodológicas que se repetem.
Este documento existe para não repeti-las.

Nenhum dado de cliente (nome, CNPJ, endereço, coordenadas) está aqui — só
metodologia.

## 1. Duas perguntas diferentes — não confundir

Há duas perguntas de auditoria possíveis, com padrões de tolerância
diferentes:

**Pergunta A — "os produtos do DANI concordam entre si?"**
GEO, TABULAR, PLANILHA, Plantas e Cotas devem bater **exatamente**, trecho a
trecho. Desde a v6.6.4 isso é garantido por construção: TABULAR, PLANILHA,
Plantas e Cotas não recalculam distância nenhuma — copiam o valor de cada
trecho já impresso pelo GEO (ver `modules/segmentos_parser.py`,
`aplicar_distancias_geo`). Se essa pergunta falhar, é bug de verdade e deve
ser reportado com o par de valores + qual lote/trecho.

**Pergunta B — "os produtos do DANI batem com a geometria crua do
shapefile?"**
Aqui **não existe tolerância zero possível**, em nenhum software. O motivo
está na seção 2. Essa pergunta só serve para achar erro grosseiro (metros,
não milímetros) — não para cobrar concordância exata.

Tratar a pergunta B com o padrão da pergunta A é a causa raiz de quase todos
os falsos positivos já vistos.

## 2. Por que a pergunta B nunca bate na última casa

O DANI arredonda cada vértice a 0,001 m (3 casas) antes de calcular área
(Gauss/shoelace) e perímetro — comportamento documentado, item 4 do
`DESCRITIVO_DANI_v6_0.md`. Geometria crua do shapefile usa os vértices sem
arredondar. São duas fórmulas válidas aplicadas a dois conjuntos de números
ligeiramente diferentes; a diferença é estrutural, não um defeito.

**Banda esperada, medida em 144 lotes reais + 59 lotes de uma segunda
rodada** (checagem completa, não amostra): até 0,051 m² de área e 0,017 m de
perímetro por lote. Qualquer diferença dentro dessa ordem de grandeza é
arredondamento, não erro. Diferença de metros, ou de dezenas de m², é que
indica problema real.

## 3. Lado com curva: como ler o comprimento sem gerar falso positivo

Dois erros já causaram falsos positivos aqui:

1. **Assumir que todo lado com curva precisa de uma frase de
   quebra "reto + curvo"**: quando o lado é 100% curva, o memorial imprime
   direto o desenvolvimento do arco como comprimento do lado — não existe
   frase de quebra porque não há trecho reto para quebrar. Um parser que só
   lê comprimento de curva dentro dessa frase vai ler 0,00 m onde na verdade
   o valor já está no total do lado.
2. **Somar duas curvas separadas do mesmo lote como se fossem uma**: quando
   um lado tem duas curvas com um reto entre elas, cada uma aparece com seu
   próprio desenvolvimento. Somar tudo e comparar com um único "raio
   nominal" produz um número que não corresponde a nenhuma curva real do
   lote.

**Regra prática**: para conferir o comprimento total de um lado, some o
valor impresso de cada trecho do lado (reto ou curvo) direto do memorial —
não reclassifique por conta própria.

## 4. Reto vs. curva: não usar limiar fixo de comprimento

Um heurístico do tipo "segmento ≥ 0,50 m = reto, abaixo disso descarta" já
gerou subcontagem de lado reto: existem segmentos retos genuínos, medidos em
campo, entre 0,12 m e 0,44 m (junção de poligonal, ajuste de alinhamento).
Descartá-los porque são curtos produz "diferença" onde o memorial está
certo.

**Regra prática**: a classificação reto/curva correta é a que o próprio
memorial já fez (a lista de trechos, cada um marcado). Não recriar o
detector de curva do zero com um critério diferente do DANI — isso é
auditar sua própria heurística, não o DANI.

## 5. Cotas do mapa — o que checar (item novo, ainda não coberto pelas
   rodadas anteriores)

O DANI tem dois caminhos para gerar as cotas (`modules/cotas_engine.py`,
`generate_cotas`):

- **Fonte única** (o caminho correto): usa `lados_prontos`, os mesmos
  trechos impressos pelo GEO (salvos em `session['_segmentos_cotas']`
  quando o GEO roda). Cota de lado curvo mostra o arco, igual ao memorial.
- **Cálculo legado** (fallback): só roda quando a sessão do QGIS não tem
  segmentos do GEO disponíveis (GEO nunca rodou nesta sessão, ou rodou
  depois das cotas). Usa `math.hypot` direto nos vértices do polígono — para
  lado reto dá o mesmo número, mas para lado com curva calcula a **corda**
  (distância reta entre os extremos), não o arco. Diverge do memorial por
  construção, não por erro.

O plugin mostra um aviso (`QMessageBox`) antes de usar o caminho legado, mas
quem gerou o mapa pode ter clicado "Sim" sem perceber a diferença.

**Protocolo de conferência das cotas**:

1. Antes de comparar qualquer valor de cota com o memorial, confirme com
   quem gerou o mapa se o Memorial GEORREFERENCIADO foi gerado **antes**
   das cotas, na mesma sessão do QGIS. Se não foi, a cota pode estar no
   caminho legado — divergência em lado curvo aí é esperada, não é bug.
2. Para cada lote, pegue o texto de cota de cada lado no mapa e o valor do
   mesmo lado no memorial GEO, por trecho (mesma ordem, mesmos extremos).
   Devem bater exatamente (pergunta A) se a fonte única foi usada.
3. Se houver divergência **e** a fonte única foi confirmada como usada,
   isso é bug real — reporte o par de valores, quadra/lote e o trecho
   específico (não só "a cota está errada").
4. Perímetro e área no centroide da cota seguem a mesma lógica: com
   `imovel_dict` disponível vêm do `saida[]` do GEO; sem ele, caem no
   fallback Gauss/`perimetro_anel` sobre o anel arredondado — mesma fonte
   usada no memorial, então também deve bater exatamente.

## 6. Roteiro de auditoria recomendado

1. **Primeiro** confira a pergunta A (produtos entre si — GEO × TABULAR ×
   PLANILHA × Plantas × Cotas), trecho a trecho, tolerância zero. É o que
   importa para o cartório e é o que a arquitetura do DANI garante desde a
   v6.6.4.
2. **Depois**, se quiser validar contra a geometria crua (pergunta B), use
   tolerância compatível com a seção 2 (ordem de 0,05 m² / 0,02 m por
   lote). Só sinalize como suspeito o que estourar essa faixa.
3. Para comprimento de lado com curva, siga a seção 3: leia o total
   impresso do lado, não recalcule.
4. Para reto vs. curva, siga a seção 4: use a classificação do próprio
   memorial.
5. Para cotas do mapa, siga o protocolo da seção 5 — primeiro confirme qual
   caminho (fonte única ou legado) foi usado antes de apontar divergência.
6. Todo achado deve vir com: quadra/lote, trecho específico, valor A, valor
   B, e qual das duas perguntas (A ou B) está sendo respondida. Um achado
   sem essas quatro informações não é verificável.
