# Descritivo técnico — DANI (Memorial Descritivo) v6.6

Pasta: `src/DANI_v6.6/DANI_v6_0/`. O código tem cerca de 24 mil linhas: o plugin, 26 módulos e 4 scripts.

---

## 1. Arquitetura

- **`memorial_descritivo_plugin.py`**
  - `MemorialDescritivoPlugin` cria o menu e o ícone, e `MemorialDialog` é a janela com as abas Projeto, Gerar e Ferramentas.
  - Insere a pasta do plugin no início do `sys.path` e importa `modules.*` como pacote de topo.
- **Execução dos memoriais:** os scripts em `scripts/` rodam com `exec()`, depois que o plugin **injeta um bloco de código antes do script (PREPEND)**. Esse bloco traz:
  - constantes de camada e campo (`CAMADA_LOTES`, `CAMPO_IMOVEL`, …, `USAR_AREA_TABELA`);
  - o filtro de escopo (`__DANI_FILTRO_*`);
  - o modo de confrontante (0/1/2) e o modo silencioso;
  - o cache de lados (`__DANI_LADOS_CACHE__`);
  - os dados de RT (`__DANI_RT_*`);
  - as funções da fonte única `geo_consistencia` (`arredondar_anel_qgis`, `area_gauss`, `perimetro_anel`, `validar_fecho`);
  - um `open()` substituto, que:
    - descarta HTML e CSV;
    - descarta o TXT quando o preview está desligado;
    - redireciona para a pasta de saída o que iria para a Área de Trabalho;
  - um patch em `QInputDialog.getItem/getText`, restaurado depois do `exec`.
- **Pós-processamento (`_pos_exec`):** converte o `saida[]` do script em .docx (`memorial_docx`), em XLSX (`planilha_memorial`, `relatorio_observacoes`) e em atributos DANI_* na camada.
- **Sessão:** `SessionManagerV2`, que substitui o `SessionManager`, grava JSON na pasta do projeto e mantém `dani_backups/` com 5 versões.
- **IA:** `ia_manager` usa a API da Anthropic (Claude) ou do Google (Gemini), com uma base local de correções.

## 2. Funcionalidades reais

### 2.1 Memorial GEO (`scripts/memorial_GEO_final.py`)

1. Lê o primeiro anel do primeiro polígono. Em multipolígono, as demais partes são ignoradas.
2. Remove segmentos de comprimento zero, começa pelo **vértice mais ao norte** (desempate pelo maior E) e recua até o início da curva, se esse vértice estiver dentro de uma.
3. Arredonda as coordenadas a 3 casas (mm). A área é calculada por Gauss no plano, e o perímetro é a soma dos lados impressos, com 2 casas cada.
4. Detecta curvas na precisão original, com quatro detectores:
   - *multicritério*: ≥ 100 micro-segmentos uniformes;
   - *pequena*: ≥ 10 cordas menores que 0,50 m;
   - *suave*: ≥ 3 giros do mesmo sinal maiores que 0,30°;
   - *override* manual `FORCAR_CURVA_SUAVE`, informado por coordenada.
5. Curva: D = R × ângulo central. Se D divergir mais de 3 cm da soma bruta dos pontos, gera uma observação.
6. Retas colineares são fundidas (Δaz ≤ 0,30° e afastamento ≤ 10 mm, com o mesmo confrontante). Lados menores que 5 mm são absorvidos pelo vizinho.
7. Confrontante: buffers de 0,01 a 0,20 m no ponto central do segmento, testando lotes, ruas e a camada externa.
8. Rodapé com o sistema, fuso e datum tirados do SRC da camada, e data do dia.

### 2.2 Memorial TABULAR (`memorial_tabular_final.py`) e PLANILHA (`memorial_Tabela_e_texto.py`)

- Os dois scripts têm código quase idêntico (o `diff` confirma). A PLANILHA acrescenta o quadro CSV/HTML e o memorial HTML, e reaproveita o cache de lados do TABULAR.
- **Frente:**
  - lado que toca uma única rua, com ≥ 60 % dos 5 pontos centrais do segmento dentro de um buffer de 1 cm da rua;
  - em lote de esquina, a rua de **menor testada**; o "voto" dos vizinhos só desempata;
  - com 2 frentes opostas (±30°), o usuário escolhe, e a outra rua vira fundo;
  - curvas soltas vão para a frente quando o lote tem uma só rua.
- **Fundo:** segmentos que não são adjacentes à frente nem ligados a ela por deflexão menor que 30°.
- **Laterais:** o sinal da diferença de azimute em relação às retas da frente separa direita de esquerda. Casos ambíguos geram a janela "Validar lados" e, se o usuário rejeitar, a classificação manual.
- O perímetro é a soma dos 4 lados ou, quando disponível, o perímetro declarado pelo GEO (`segmentos_parser.parsear_perimetros_geo`).
- Um lado vazio sai como "0,00 m por se tratar de um lote triangular" ou "lote de esquina com frente estendida…".

### 2.3 Plantas técnicas (`scripts/gerador_plantas.py`)

- Folha A4 retrato em matplotlib, com o mesmo algoritmo de segmentos do memorial.
- Desenha os vértices V1…, a dimensão e o azimute por dentro do lote, os confrontantes por fora (agrupados), a área e o perímetro no centróide, o norte e a escala gráfica.
- Gera PDF e SVG (texto editável).

### 2.4 Ferramentas (`modules/funcionalidades_dialog.py`)

- **Planta Geral** (`planta_geral_qpt.py`): usa `MODELO_GEO.qpt`, com textos injetados por templateUuid.
  - Folha A4 a A0, escala automática, grade, escala gráfica e rosa dos ventos.
  - Declinação magnética por pygeomag ou pela NOAA, convergência meridiana e fator K.
  - Croqui ESRI (World Imagery) ou croqui local, e camada de cotas opcional com estilo .qml.
- **Planta do Lote** (`planta_individual_qpt.py`): folha A4 ou A3 por lote, com o lote destacado em amarelo, cotas filtradas por QUADRA/LOTE, croqui do loteamento e carimbo do lote. Saída em PDF e/ou SVG.
- **Declividade** (`declividade_engine.py`): a frente é o lado cujo centro fica mais perto de uma rua. A declividade é |Δcota| / distância entre frente e fundo, **sem sinal**, e é gravada na camada.
- **Cotas** (`cotas_engine.py`, `cotas_planta.py`): camada em memória ou em disco, com comprimento, azimute, V1…, raio, área e perímetro. Cotas repetidas a menos de 0,05 m são deduplicadas.
- **Tabelas de Áreas** (`tabelas_engine.py`):
  - Tabela 1 (lotes por quadra) e Tabela 2 (resumo por categoria).
  - A categoria é deduzida do texto do campo QUADRA por regex: "institu", "verde", "app", "servid", etc.
  - Gera XLSX com fórmulas `=SUM`.
- **ABNT NBR 12721** (`abnt_nbr12721_engine.py`): planilha de áreas e coeficientes. A norma é citada no código.
- **ONR** (`onr_engine.py`): cria um shapefile novo com os 33 campos de `CAMPOS_ONR`. A camada de entrada não é alterada.

### 2.5 Outros módulos

- `preview_dialog`: editor por lote, IA e "propagação" para o vizinho.
- `relatorio_observacoes`: XLSX com os avisos, observações e casos especiais.
- `memorial_docx`: TXT convertido em DOCX em Arial, com as linhas "CONFERIR EM PLANTA" e "triangular" em amarelo.
- `planta_loteamento.py`: planta geral em matplotlib ou QPT antigo. Nenhuma chamada a ela foi encontrada na interface atual (a Planta Geral usa `planta_geral_qpt`).

## 3. Normas citadas no código

- **ABNT NBR 12721:** aba e engine próprias.
- **INCRA:** citado apenas como justificativa do arredondamento de coordenadas a 3 casas. A linha INCRA/SNCR é **removida** do carimbo da planta (é cadastro rural).
- **ONR (Operador Nacional do Registro):** estrutura de campos do shapefile.

Nenhuma outra norma aparece no código.

## 4. Dependências

- **QGIS**, com PyQt5 e `qgis.core`/`gui`/`utils`.
- **matplotlib e numpy:** plantas técnicas e `planta_loteamento`.
- **python-docx e openpyxl:** Word e Excel. Nenhuma das duas é empacotada nem declarada no metadata.
- **pygeomag**, opcional: declinação magnética. Sem ele, o plugin consulta a NOAA pela internet.
- **Internet**, opcional: croqui ESRI, NOAA, Claude e Gemini.
- `_precisa_bridge.py`: integração com a suíte.

## 5. Limitações, bugs e trechos não testados

Legenda: **[CONFIRMADO]** = comprovado pela leitura linha a linha do fluxo; **[PROVÁVEL]** = depende de dado ou ambiente e não foi executado.

### 5.1 Dados e sessão

1. **[CONFIRMADO] Reabrir "Configurar Camadas" apaga os Dados do Loteamento.**
   - `layer_setup_dialog._validar_e_salvar` cria um `lc = {}` novo e chama `session.set_layer_config(lc)`, que substitui o dicionário inteiro.
   - Perdem-se: proprietário, CNPJ, endereço, município, comarca, RT, a posição da assinatura e o `campo_area` (quando "Usar Área da tabela" está desmarcado).
2. **[CONFIRMADO] O autosave grava uma só vez por sessão do QGIS.**
   - `SessionManagerV2.auto_save` marca `_autosave_feito = True`, e só `salvar_com_backup` volta a zerar a marca.
   - Mudanças feitas depois do primeiro fechamento da janela só são gravadas se o usuário clicar em "Salvar Sessão".
   - `save()` está desativado de propósito.
3. **[CONFIRMADO] Os scripts GEO, TABULAR e PLANILHA editam a camada de LOTES do usuário.**
   - Criam os campos AREA, PERIMETRO e AREA_CALC via `dataProvider().addAttributes` (a mudança é permanente).
   - **Sobrescrevem o campo AREA** com a área calculada sempre que "Usar Área da tabela" está desligado, e fazem `commitChanges()`. Um valor oficial digitado no campo AREA é perdido.
   - Se o usuário cancelar a escolha de escopo (fluxo sem filtro injetado), a camada fica em modo de edição, porque `startEditing` acontece antes da pergunta.
4. **[CONFIRMADO]** A declividade e os campos DANI_* também são gravados na camada (`addAttributes` + `commitChanges`).

### 5.2 Cálculo e texto

5. **[CONFIRMADO] O filtro de ângulo dos confrontantes não filtra nada.**
   - Em `buscar_confrontantes_por_poligonos` (GEO, TABULAR, PLANILHA e plantas), os dois ramos (`>45° e <135°` e o complemento) aceitam o candidato.
   - Na prática, qualquer lote que toque o buffer entra, apesar da docstring dizer "só aceita confrontantes alinhados".
6. **[CONFIRMADO] Os detectores de curva diferem entre GEO e TABULAR/PLANILHA.**
   - Só o GEO tem os detectores "pequena" (no laço), "suave" e o override. `detectar_curva_multicriterio` também tem implementações diferentes nos dois grupos.
   - Os documentos podem descrever o mesmo lote com lados diferentes. O perímetro só é igualado quando o GEO roda antes, na mesma execução.
7. **[CONFIRMADO] A área muda de fonte entre as saídas da PLANILHA.**
   - O TXT usa a área da tabela (se `USAR_AREA_TABELA`), mas os dados de exportação (`dados_lote['area']`) usam sempre `area_plano`.
8. **[CONFIRMADO] A mensagem de área contradiz o comportamento.**
   - No GEO, quando a área da tabela diverge mais de 5 %, a observação diz "usando a da tabela", mas o comentário do código diz que deveria cair na calculada.
   - O código usa a da tabela.
9. **[CONFIRMADO] Segmentos que ficam sem lado somem da descrição.**
   - No TABULAR e na PLANILHA, a regra que forçava sobras para FUNDO foi revertida (comentário "v6.1").
   - Um segmento com `lado=None` não aparece em nenhum dos 4 lados. Sem o GEO de referência, o perímetro também não o soma.
10. **[CONFIRMADO] Tabela de Áreas XLSX: o "TOTAL GERAL DO LOTEAMENTO" sai em dobro.**
    - A fórmula `=SUM(C3:C{lin-2})` soma os lotes **e** as linhas de subtotal (`=SUM`) de cada quadra.
    - A tabela em texto na tela está correta.
11. **[CONFIRMADO] ABNT 12721:**
    - `dados.get('nome_empreendimento')` nunca é preenchido, então o item 3.1 fica em branco.
    - O arredondamento é distribuído com ±0,01 aleatório (seed 42), em células amarelas.
    - O resíduo do coeficiente vai para o maior lote.
12. **[CONFIRMADO]** `memorial_docx.extrair_observacoes` grava `'lote': s`, a linha inteira da mensagem, na coluna Lote do XLSX de observações.
13. **[CONFIRMADO] A escolha de "Lote específico" usa `int(float(lote))`.**
    - Nos scripts (fluxo sem filtro injetado) e no `gerador_plantas`, um lote com letra ("5 B", "12A") gera ValueError e interrompe a execução.
    - [PROVÁVEL] O mesmo acontece no filtro do plugin.
14. **[CONFIRMADO] Modo de confrontante "Automático" (padrão 0):** com vários candidatos, usa o primeiro em ordem alfabética, o que pode estar errado; sem candidato, escreve "CONFERIR EM PLANTA".
15. **[PROVÁVEL]** `extrair_info_crs`: se a descrição do SRC não contiver "zone"/"fuso", o fallback por EPSG calcula `fuso = epsg − 31971` (31983 → 12, em vez de 23S). Com a descrição padrão do QGIS ("SIRGAS 2000 / UTM zone 23S") o fuso sai correto.
16. **[CONFIRMADO]** TXT, HTML e CSV dos scripts têm assinatura fixa "Helton José Carmanini Lourenço – CREA-MG 141.370/D" e rodapé "Guiricema".
    - O `__DANI_RT_*` injetado não é lido pelos scripts.
    - O .docx usa o RT da sessão.
    - O HTML e o CSV são descartados pelo `open()` injetado; o TXT só é mantido com o preview ligado.
17. **[CONFIRMADO] Planta geral matplotlib (`planta_loteamento`):** rodapé fixo "Precisa Agrimensura LTDA — CREA-MG 141.370/D". A escala impressa ("E: 1:x") é estimada e não corresponde necessariamente ao desenho, que usa `bbox_inches='tight'`.
18. **[PROVÁVEL] Plantas técnicas (`gerador_plantas`):**
    - O texto "Escala 1:x" pode não corresponder à escala real impressa: o eixo é salvo com `tight` e não fica amarrado ao tamanho em mm.
    - O teste de "segmento pequeno" usa `transData` sobre coordenadas UTM, e não sobre as coordenadas do desenho.
19. **[CONFIRMADO] Declividade:** a frente é o lado mais próximo de qualquer rua, e o valor não tem sinal (não diz se o terreno sobe ou desce).

### 5.3 Interface

20. **[CONFIRMADO] A "propagação para o vizinho" no preview não altera nada útil.**
    - `_propagar_para_vizinho` substitui a referência ao lote atual pelo mesmo texto.
    - Mesmo assim, a mensagem diz "Lotes vizinhos atualizados automaticamente".
21. **[PROVÁVEL]** Os diálogos "com canvas acessível" (`getItem_canvas`/`getText_canvas`) usam `exec_()`, então são modais na prática.
    - O patch de `QInputDialog` vale globalmente para a classe durante o `exec`.
    - As janelas "Validar lados" e "Lote de esquina" usam `QEventLoop` e não são modais.
22. **[CONFIRMADO]** `MemorialHighlighter` é recriado a cada troca de lote no preview, acumulando highlighters.

### 5.4 Segurança e privacidade

23. **[CONFIRMADO] IA:**
    - As chaves ficam em QSettings, em texto puro.
    - A chave do Gemini vai na URL.
    - "Testar" grava a chave antes de salvar.
    - O texto do memorial é enviado a serviços externos.
    - Os modelos Gemini configurados são antigos.

### 5.5 Instalação e empacotamento

24. **[PROVÁVEL]** Os imports `from modules...` com `sys.path.insert(0, …)` podem colidir com outro plugin que também tenha um pacote chamado `modules`.
25. **[CONFIRMADO] Versões inconsistentes:**
    - metadata 6.6 / "DANI v6.1";
    - janela "v5.03", statusTip "v5.01";
    - docstrings de v3.0 a v5.07;
    - README_SHAPEFILE v5.13;
    - PreviewDialog com o título "DANI v3.0".
26. **[CONFIRMADO]** `CAMPOS_ONR` tem 33 chaves (a docstring diz 32).
27. A planta geral põe "[Zona NNS]" fixo em parte do carimbo e inclui no mapa todas as camadas visíveis do projeto.

### 5.6 Não testado nesta revisão

- Nenhum dos fluxos foi executado no QGIS: os itens vêm de leitura de código.
- Geração de .docx e XLSX com dados reais.
- Croqui ESRI e NOAA (dependem de rede).
- Chamadas reais às APIs de IA.
- Comportamento em SRC geográfico (as medidas sairiam em graus).
