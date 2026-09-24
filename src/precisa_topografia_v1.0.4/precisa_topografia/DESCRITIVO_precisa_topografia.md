# Descritivo Técnico-Funcional — Precisa Topografia

**Pasta:** `precisa_topografia` · **Versão declarada:** 1.0.4 · **QGIS mínimo:** 3.16
**Arquivos lidos:** `__init__.py`, `plugin.py`, `core/*.py`, os três módulos em
`modulos/` (todos os `.py`) e o texto dos três `manual.docx` embarcados.

---

## 1. Arquitetura
`PrecisaTopografiaPlugin` instancia três sub-plugins independentes, cada um
com ação na barra, item no menu `&Precisa Agrimensura` e ação “📖 Manual”
(abre `manual.docx` com o aplicativo do sistema). `run()` (Suíte) mostra um
`QMenu` no cursor. `core/` traz *stubs* locais (log, constantes, deps,
compatibilidade QVariant/QMetaType).

| Módulo | Arquivos principais |
|---|---|
| Caderneta de Estação Total | `parser_topograph.py`, `calculador.py`, `dialogo_caderneta.py`, `dialogo_importar.py`, `exportar.py` |
| Poligonal e Irradiação | `calculadora_poligonal.py`, `dialogo_poligonal.py`, `dialogo_cabecalho.py`, `relatorio_poligonal.py` |
| Relatório de Levantamento | `dialogo_relatorio.py`, `gerador_relatorio.py` |

## 2. Funcionalidades implementadas

### 2.1 Caderneta
- **Parser Topograph** de largura fixa (285 col., latin-1; linhas < 200
  caracteres ignoradas). Posições: flag [9], data [26:32], hora [32:38],
  estação [48:63], ré [63:78] (zeros à esquerda removidos), Hz ré [83:90],
  ZN ré [93:101], HI mm [127:132], dist. ré [160:168], código [168:183],
  ponto [183:198], Hz [198:207], ZN [208:218], HP mm [243:248], dist. [276:285].
  Ângulos DDDMMSS; flag `0` inicia estação; observação exige código e distância > 0.
- **Cálculo:** Az = Az_ré + (Hz − Hz_ré); Dh = |Di·sen ZN|; ΔZ = Di·cos ZN + HI − HP;
  E/N/Z por coordenadas polares; Az_ré pelas coordenadas da ré ou digitado.
- **Importação de coordenadas** de camada de pontos por nome exato
  (detecção automática de campos nome/E/N/Z).
- **Saídas:** camada memória `Caderneta_ET` PointZ (SRC escolhido ou
  indefinido), CSV `;` com números BR e ângulos GMS/decimais (UTF-8 BOM), DXF R12
  escrito à mão (POINT + TEXT em 3 camadas; texto 0,5 m).

### 2.2 Poligonal (`calcular_poligonal_fechada`)
- Soma teórica (n−2)·180°; erro angular; tolerância **10"·√n**.
- Correção angular: erro distribuído igualmente.
- Propagação: Az_{i+1} = Az_i + 180° − ângulo (sentido horário, ângulos internos).
- Coordenadas provisórias a partir de E/N iniciais; ΔE, ΔN, erro linear,
  precisão 1:(Σd/erro); tolerância **1:5000 (“INCRA classe B”)**.
- Ajuste de Bowditch proporcional à distância acumulada.

### 2.3 Irradiação (`calcular_irradiacao`)
Az = Az_ré + Hz (círculo zerado na ré); Dh = |Di·sen ZN|; ΔZ = Di·cos ZN + HI − HP.

### 2.4 Relatórios
- **Poligonal/Irradiação:** DOCX (ou HTML) com identificação, fechamento
  angular e linear, coordenadas ajustadas, irradiação, declaração (Lei 6.496/1977)
  e assinatura. Cabeçalho salvo em `QSettings` (`precisa_topografia/cabecalho`).
- **Levantamento:** DOCX com 8 seções (identificação, referenciais,
  equipamentos, PPK, coordenadas, poligonal, observações, declaração); HTML reduzido sem python-docx.

## 3. Normas e referências citadas no código
| Referência | Onde | Uso |
|---|---|---|
| **INCRA — Classe B, 1:5000** | `calculadora_poligonal.py`, relatórios | Tolerância linear da poligonal |
| Tolerância angular 10"√n | `calculadora_poligonal.py` (o manual embarcado a atribui ao INCRA Classe B) | Tolerância angular |
| **Lei nº 6.496/1977** | Textos de declaração dos dois relatórios | ART — texto declaratório |
| SIRGAS 2000, hgeoHNOR2020 (IBGE) | Valores padrão do relatório | Só texto pré-preenchido |

## 4. Dependências
QGIS ≥ 3.16; `python-docx` opcional (DOCX); sem outras bibliotecas.

## 5. Limitações conhecidas e pontos de atenção

1. **Bug confirmado — poligonal com ângulos deslocados de um vértice.** Em
   `calcular_poligonal_fechada`, o azimute do lado i+1 é calculado com
   `angulos[i]` (ângulo do vértice i), quando deveria usar o ângulo do vértice
   i+1 (onde o lado i+1 começa). A interface e o manual definem cada linha
   como “ângulo interno do vértice + distância ao próximo”. **Teste
   reproduzível** (quadrilátero P1(0,0)→P2(0,30)→P3(40,40)→P4(50,0),
   sentido horário, ângulos e distâncias exatos): o plugin reporta erro
   linear de **22,29 m (1:7)** e desloca os vértices; com os ângulos
   deslocados uma linha para cima, o erro cai a **0,000 m**. Afeta
   coordenadas, azimutes, o relatório e as camadas geradas. Só é invisível
   em figuras com todos os ângulos iguais (ex.: retângulos).
2. **Tipo “Aberta” sem efeito:** `_on_tipo_poligonal` é vazio e o cálculo é
   sempre o de poligonal fechada (o manual embarcado diz que a aberta é suportada).
3. Convenção **fixa**: ângulos internos com percurso horário; não há opção
   para anti-horário ou ângulos externos; Z inicial e a coluna “E/N (1º pto)”
   da tabela são ignorados.
4. **Irradiação** pressupõe círculo zerado na ré (Az = Az_ré + Hz) e recebe
   ângulos em graus **decimais**, diferente do restante (GMS); linhas com valor
   inválido são descartadas em silêncio.
5. **Parser da caderneta:** posições fixas “validadas nos arquivos reais do
   usuário” (comentário no código) — outras versões do TransferCPE/Topograph
   podem desalinhar; ângulo inválido vira **Hz = 0° / ZN = 90° silenciosamente**,
   gerando ponto errado sem aviso.
6. DXF: camada `ET_ESTACOES` é declarada, mas nenhum ponto de estação é
   escrito; gravação em latin-1 falha com caracteres fora dessa codificação.
   O `manual.docx` do módulo ainda cita DXF “AC1015”, mas o código gera R12.
7. **Relatório de Levantamento:** a seção 6 imprime “Não informado” em soma
   dos ângulos, soma teórica e tolerância angular (campos nunca coletados);
   “Estação RBMC utilizada”, “Época de referência” sempre “Não informado”;
   “Modo de processamento: PPK” fixo, mesmo em levantamento só com estação
   total; declaração fixa “Engenheiro Agrimensor / Engenheiro Cartógrafo”.
   Valores não numéricos em sd3d/E/N na camada abortam a geração.
8. HP global padrão = HI do arquivo (valor pouco intuitivo).
9. `modulos/relatorio_levantamento/__init__.run()` exige python-docx, mas esse
   caminho não é usado pelo `plugin.py` (o diálogo tem *fallback* HTML).
10. O título do relatório de poligonal é sempre “POLIGONAL FECHADA”.
