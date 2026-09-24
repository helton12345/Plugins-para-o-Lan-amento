# Descritivo técnico — Precisa Terraplanagem Suite v2.4.0

Levantamento feito pela leitura do código da pasta `precisa_terraplanagem_suite/`. O código não foi alterado.

Marcas usadas:

- **[CONFIRMADO leitura]**: comportamento visto no código;
- **[CONFIRMADO teste]**: verificado executando o código;
- **[PROVÁVEL]**: dedução que depende do ambiente;
- **[A CONFIRMAR]**: depende de norma ou de serviço externo.

## 1. Arquitetura

| Pasta | Papel |
|---|---|
| `plugin.py`, `__init__.py`, `_precisa_bridge.py` | Entrada no QGIS. Cria uma ação "Precisa Terraplanagem Suite" que abre o `SuiteDialog`. A "ponte" da Suíte suprime o item do menu padrão, que passa a vir do menu unificado **Precisa Agrimensura**. |
| `dialogs/` | Todas as telas: hub `suite_dialog.py`, M1–M6, ferramentas auxiliares, IA, sessão, CRS e configuração. `main_dialog.py` é uma interface antiga que nada chama (código morto). |
| `modules/` | Cálculo com QGIS: estaqueamento, greide, seções, volumes, relatório, platôs, locação, superfícies (TIN/MTP), curvas, cul-de-sac, interseções, superelevação, DXF/PDF, sessão e bacias. |
| `engine/` | Motor em Python puro, testável sem QGIS: TIN (`surface`), alinhamento vertical, corredor, cruzamento, volumes, lote/drenagem, LandXML, prancha, layout QPT, hachura e materiais de referência. `engine/adapters/qgis_io.py` faz a ponte com o QGIS. |
| `adaptativa/` | Terraplanagem adaptativa por classe de declividade (DIP): classificador (GDAL/Processing), terraplanagem por classe (numpy/scipy/OSQP), mescla, volumes, curvas e relatório DOCX próprio. |
| `ai/` | Co-piloto e orquestrador por IA: provedores Claude, OpenAI e Ollama, perfil de projeto, memória, checkpoints e custo. |
| `core/` | Utilitários: compatibilidade Qt5/Qt6, CRS, validadores (curvas, geometria, entradas), cache do TIN, gravação de DXF, wrapper do GRASS, dependências, log e encadeamento de camadas entre módulos. |
| `project_config.py` | Parâmetros do projeto, gravados no `.qgz` (escopo "Terraplanagem"), com os valores padrão comentados. |

**Encadeamento dos dados entre módulos:**

- **M1:** cria a camada Estacas e registra MDT, eixos e estacas (`core/encadeamento.py`).
- **M2:** grava `cota_proj` na camada e os PVIs em `greide_pvis_json`.
- **M3:** grava `secoes_json` e `alertas_json` e salva os `.gpkg`.
- **M4:** grava `volumes_json`.
- **M5:** lê tudo isso e gera PDF e/ou DOCX.

## 2. Funcionalidades reais

### 2.1 M1 — Estaqueamento (`modules/m1_estaqueamento.py`)

- **Várias vias:** a numeração reinicia em `0+000` em cada via e há um ID global.
- **Estacas:** intermediárias opcionais e estacas extras nas interseções geométricas entre vias.
- **MDT:** por `qgis:tininterpolation`, com o GDAL como alternativa. Com limite de levantamento, usa um TIN com restrições e recorta pela propriedade. Um raster todo NODATA é rejeitado (`_tem_pixel_valido`). Há cache do TIN por camada, campo, pixel e limite (`core/tin_cache.py`).
- **Validação prévia das curvas** (`core/curvas_validator.py`): cota, CRS e cobertura. Erros bloqueiam a execução.
- **Nome da estaca:** `N+DDD` ou `N+DDD.d`, com o resto dentro da estaca. O erro histórico da v2.0.4 está corrigido.
- **Saídas:** camada Estacas (memória) e CSVs de estacas e de resumo por via.

### 2.2 M2 — Greide (`modules/m2_greide.py`)

- PVIs por via.
- Concordância vertical parabólica.
- **K mínimo** calculado pela distância de visibilidade de parada na velocidade de projeto (AASHTO 2011 via `engine/alignment.py`), ou um K imposto na configuração. O antigo K = 20 fica só como último recurso.
- Comprimento L ≥ 0,6·V, arredondado para cima em múltiplos de 20 m.
- Alertas de declividade mínima e máxima e de concordância.
- Cota vermelha = TN − projeto.
- **Tela:** gráfico interativo (arrastar PVI, zoom e pan) e exportações PNG, CSV (notas de serviço) e DXF (perfil).

### 2.3 M3 — Seções (`modules/m3_secoes.py`, `secao_tipo.py`, `superficie_*`)

- Seção em cada estaca pelo gabarito (Seção Tipo) ou pelos parâmetros da tela.
- Busca do pé e da crista do talude no terreno natural: a janela de busca aumenta até 640 m. A seção é marcada `secao_confiavel=False` quando o terreno natural termina antes.
- Largura pela camada de meio-fio, limitada a 1,5 × a meia-plataforma de projeto.
- **Saídas:** pontos, linhas e polígonos de offset em `.gpkg` e caderno de seções DXF + PDF.
- **Superfície projetada (MTP):** o TIN é a fonte única desde a 2.0.0, com aresta máxima de 30 m e hierarquia nos cruzamentos. O método por pixel fica só como alternativa.
- Curvas de nível do terreno natural e do projetado.
- **Cul-de-sac:** circular, com ilha, hammerhead e elíptico.

### 2.4 M4 — Volumes (`modules/m4_volumes.py`)

- **Métodos:** áreas médias ou prismoidal (Simpson sobre trios de seções, rateado por trecho).
- **Homogeneização:** Vhom = Vcorte / Fh. Os valores padrão de Fh vêm de `engine/materiais_ref.py`, e um Fh fora da faixa publicada gera alerta.
- Pula trechos entre vias diferentes e trechos com seção não confiável (**somente se o campo chegar**; ver §5.2).
- Diagrama de Brückner e exportação Excel (openpyxl) e PNG.
- `engine/volumes.py` tem a conferência de volume por cruzamento de TINs, usada pelo Editor de Malha.

### 2.5 M5 — Relatório (`modules/relatorio.py`, `m5_relatorio.py`)

- **PDF** (ReportLab) e/ou **DOCX** (python-docx).
- **Conteúdo:**
  - identificação;
  - parâmetros técnicos com a norma de cada um;
  - notas de serviço;
  - quadro de volumes;
  - Brückner por via;
  - resumo executivo;
  - platôs;
  - alertas;
  - referências normativas.
- O texto do método de volume e do K vertical é montado conforme a configuração.

### 2.6 DIP Adaptativa (`adaptativa/`)

- **Classificador:** gdal:slope (Horn), reclassificação, polygonize com 8 vizinhos, dissolve, remoção de furos, snap, eliminação de ilhas, Douglas-Peucker, Chaikin e correção de geometria. Saída em shapefile UTF-8 com os campos `classe_id`, `rotulo`, `de_val`, `ate_val`, `unidade` e `cor_hex`.
- **Terraplanagem por classe:**
  - **Plano inclinado:** aspecto médio ponderado e cota de passagem por mínimos quadrados, com corte = aterro.
  - **Declividade máxima:** OSQP minimiza Σ(z−z₀)² com |Δz| ≤ s·res·0,92 entre os 8 vizinhos.
  - Parâmetros internos sem controle na tela (usados pela IA/orquestrador): `modo_massa` (compensado, só corte, só aterro) e cotas impostas por classe.
  - Taludes H:V por distância euclidiana. O talude pode entrar nas classes vizinhas.
- **Mescla:** suaviza as fronteiras entre classes. Volumes por diferença de rasters. Curvas por `gdal.ContourGenerate`.
- **Relatório DOCX próprio:** seções transversais (41 pontos, bilinear), áreas médias, Brückner com Fh da 1ª categoria, perfil e gráficos.

### 2.7 Bacias (`modules/m6_bacias.py`, `core/grass_wrapper.py`)

- GRASS: r.fill.dir, r.watershed, declividade e aspecto, canais e bacias vetorizados, TWI e TPI.
- **Morfometria:** a ordem de Horton é uma estimativa simplificada, como o próprio código diz.
- **Hidrologia:** Tc por Kirpich ou Giandotti; Método Racional Q = C·I·A/3,6.

### 2.8 Ferramentas auxiliares

- **Platôs:**
  - cota por média, mínima ou máxima do terreno natural, manual, ou plano ajustado por mínimos quadrados (`ajuste_plano_mq.py`);
  - rampas, taludes, manchas e locação;
  - empolamento e número de caçambas (`empolamento_terraplanagem.py`);
  - PDF resumo.
- **Locação:** pontos de plataforma e talude exportados em TXT, DXF e XLSX.
- **Interseções:** a cota nos cruzamentos segue a hierarquia das vias, em cascata.
- **Superelevação e superlargura:** detecção de curvas pelo azimute.
- **Exportar DXF:** gravador próprio, R2000 por padrão, com ezdxf ou gravação manual.
- **Produtos Finais:**
  - prancha no Compositor/PDF (`engine/prancha.py`, `layout_qpt.py`, `hachura.py`);
  - notas de serviço;
  - LandXML 1.2;
  - drenagem dos lotes (`engine/lote.py`), com o novo motivo "cobertura insuficiente" e o terreno natural como complemento.
- **Editor de Malha:** exporta pontos, arestas e breaklines. No modo ao vivo, exclui vértice, desenha breakline e inclui ponto com a cota interpolada no greide. Reconstrói em cadeia a malha, o MTP, os volumes e o relatório.
- **Sessão:** grava e restaura PVIs, gabarito, configuração, parâmetros dos platôs, camadas e pasta, no `.qgz` ou em JSON externo.

### 2.9 IA (`ai/`)

- **Provedores:**
  - Claude (API Anthropic por `urllib`, streaming, cache de prompt, até 3 tentativas);
  - OpenAI (`gpt-4o`);
  - Ollama local.
- **Painel co-piloto** nos M2, M3, M4 e DIP: a IA responde em JSON com ações, e cada ação tem o botão Aplicar.
- **Orquestrador** (`ai/orquestrador.py`):
  - cadeia completa M1 → relatório;
  - modos assistido e autônomo (confiança mínima de 0,70);
  - checkpoints, cópia versionada e relatório de bug em caso de erro;
  - custo estimado em US$ e R$, com câmbio consultado em `economia.awesomeapi.com.br`.
- **Memória opcional** em uma pasta escolhida: histórico de ações, tokens e análises.

## 3. Normas e referências citadas no código

As normas abaixo são **citadas** no código ou nos textos gerados. A adequação de cada uma **não foi verificada** [A CONFIRMAR].

- **Na tela e nos relatórios:**
  - NBR 13133:2021 e NBR 9732:1987;
  - Lei 6.766/1979 e Lei 9.785/1999;
  - GRAPROHAB/SP e IP-03 SIURB;
  - NBR 14166:2022, NBR 11682:2009 e NBR 5681:1980;
  - NBR 15777:2009, NBR 16752:2020 e NBR 17047:2022;
  - DNIT 108/2009-ES e DNIT IPR-706;
  - AASHTO Green Book 2011;
  - CAPUTO (2015) e SICRO/DNIT.
- **Na DIP:**
  - Lei 12.651/2012 (Código Florestal), NBR 6502:1995, NBR 8044:1983 e NBR 11682:2009;
  - DNIT/IPR-719 e CONFEA Res. 1.048/2013;
  - Borges (1994), Veiga (UFPR), Horn (1981) e Stellato et al. (OSQP).
- **Nas Bacias:** Kirpich (1940), Giandotti (1934) e Método Racional.
- **Observações sobre os valores padrão:**
  - o próprio código avisa que **27%** de declividade máxima e talude **1:1 em aterro** vão além do usual. Pede para conferir o plano diretor e fazer verificação geotécnica ("não é o 1:1,5/1:2 usual da NBR 5681");
  - a faixa de 12% "confortável" é atribuída a DER-SP/GRAPROHAB.

## 4. Dependências

| Pacote | Uso | Se faltar |
|---|---|---|
| numpy, GDAL/osgeo, processing | Todo o cálculo | Vêm com o QGIS |
| scipy | DIP (`ndimage`), amostragem bilinear | A DIP não carrega (`ImportError` na importação do módulo) |
| osqp | DIP, método "Declividade Máxima" | Erro claro ao processar |
| matplotlib | Gráficos: M2, M4, Seção Tipo, DIP, PNG | Os gráficos são desativados com aviso |
| python-docx | Relatório DOCX (M5, DIP) | Aviso/erro ao gerar |
| reportlab | Relatório PDF, caderno de seções, PDF de platôs | O PDF falha; o caderno exige DXF **e** PDF |
| openpyxl | Excel do M4 e da locação | A exportação falha |
| ezdxf | DXF (opcional; há gravação manual) | Usa a gravação manual |
| shapely, Pillow | Geometria auxiliar, imagens | Depende da função |
| GRASS (provedor do QGIS) | Aba Bacias | A aba mostra erro |
| Rede | IA online e câmbio | A IA online não funciona; o câmbio cai no valor em cache ou padrão |

`core/deps.py` tem um instalador via pip, mas **nenhuma tela o chama**.

## 5. Limitações, bugs e trechos não testados

### 5.1 Testes

- **[CONFIRMADO teste]** `engine/tests` (motor puro, sem QGIS): **380 passed** (pytest, numpy 2.4 e scipy).
- A pasta **`testes/`** citada no changelog (540 testes, `conftest.py`, integração com o QGIS) **não está no pacote**. Não foi possível reproduzir essa contagem.
- As telas, a DIP, as Bacias e a IA **não foram executadas** nesta revisão (exigem QGIS e GRASS).

### 5.2 Fluxo de dados M3 → M4 → M5 (manual)

- **[CONFIRMADO leitura] Exclusão de seções não confiáveis não acontece no M4 manual.**
  - O M3 grava `secoes_json` **sem** o campo `secao_confiavel` (`m3_secoes_dialog.py`).
  - O M4 manual monta as seções só com áreas e estaca (`m4_volumes_dialog.py::_calcular`).
  - Resultado: a exclusão anunciada na 2.0.0 nunca ocorre por esse caminho, e as seções com talude "no limite da busca" entram no volume.
- **[CONFIRMADO leitura] O relatório M5 recebe o quadro incompleto do M4 manual.**
  - O M4 grava em `volumes_json` só estas chaves: `est_ini`, `est_fim`, `dist_ini`, `area_corte`, `area_aterro`, `vol_corte`, `vol_aterro` e `saldo_acum`.
  - O relatório lê `area_*_med/ini`, `vol_homogeneizado`, `saldo_acumulado`, `id_via` e `dist_acum_fim`.
  - Resultado: no PDF e no DOCX, as **áreas, o volume homogeneizado e o saldo acumulado saem 0**, e o **Brückner por via** fica plano ou agrupado numa única via.
  - O Editor de Malha grava o quadro completo, portanto por esse caminho o relatório sai correto.
- **[CONFIRMADO grep] O "Resumo de platôs" do M5 fica sempre vazio.** A chave `platos_resumo_json` é lida pelo M5 e pelo Editor de Malha, mas nenhum módulo a grava.
- **[CONFIRMADO leitura] Alertas duplicados.** Cada execução do M3 soma os alertas novos aos anteriores em `alertas_json`, sem limpar. Assim, o M5 lista alertas repetidos.
- **[CONFIRMADO leitura] O M4 soma o saldo acumulado de todas as vias** em sequência. O Brückner da tela não reinicia por via; o relatório tenta separar pelo `id_via`, que não chega (item acima).

### 5.3 Parâmetros divergentes entre caminhos

- **[CONFIRMADO leitura] O M3 manual tem padrões fixos na tela** (plataforma 12, calçada 2,5, **talude 1,5 corte / 2,0 aterro**) e **não lê** a Configuração do projeto, cujo padrão é 1:1.
  - O relatório M5 também cai em 1,5 e 2,0 quando não há gabarito.
  - O orquestrador de IA usa o perfil (1:1).
  - Sem Seção Tipo aplicada, o mesmo projeto pode ter taludes diferentes conforme o caminho.
- **[CONFIRMADO leitura] O orquestrador sobrescreve a Configuração.** Ao iniciar, `PerfilProjeto.aplicar_no_project_config()` grava os valores do perfil (por exemplo, declividade máxima 25%, calçada 1,5) sobre o que o usuário configurou, sem aviso.
- O rótulo "Padrão (7m/1,5m · 25%/2% …)" na tela da IA não bate com a Configuração padrão do projeto (27%, calçada 2,5).

### 5.4 DIP Adaptativa

- **[CONFIRMADO leitura]** A unidade padrão do classificador é **Graus**, mas os intervalos padrão são em **%**. Processar sem trocar a unidade classifica de forma errada (por exemplo, "0–8" passa a ser 0–8°, cerca de 14%).
- **[CONFIRMADO leitura]** As ações da IA "estratégia por classe" e "talude por classe" só gravam `_estrategias_ia` e `_taludes_ia`, que **nenhum código lê**. "Excluir classe" apenas pinta a linha. Nenhuma dessas ações muda o processamento.
- **[CONFIRMADO leitura]** Na mescla, um MDT parcial com dimensões diferentes do MDT original é **ignorado sem aviso** (`mesclar_mdts`). Isso afeta o "Adicionar MDT externo".
- **[CONFIRMADO leitura]** `converter_vetor_para_mdt` ignora o tipo (curvas ou pontos): as curvas entram como pontos no TIN.
- **[CONFIRMADO teste]** `extrair_secoes_transversais` usa `np.trapz`, que **não existe mais no NumPy 2.4** (`hasattr(numpy, 'trapz') == False`). Com NumPy novo, as seções e o Brückner do relatório DIP falham; o erro aparece como "Aviso seções" e o relatório sai sem esses elementos.
  - Com o NumPy do QGIS atual (1.x ou 2.0–2.3) ainda funciona, com aviso de função obsoleta [PROVÁVEL].
- **[CONFIRMADO leitura]** O texto do relatório DOCX diverge do código:
  - diz "4 vizinhos"; o código usa 8;
  - diz fator de segurança "padrão 1,0"; o código usa 0,92;
  - dá a fórmula do Brückner sem Fh; o código aplica Fh ao corte;
  - diz que o plano "minimiza o volume"; o código minimiza a soma dos quadrados;
  - cita a versão "v3.5.2".
- **[CONFIRMADO leitura]** O relatório DOCX da DIP tem **responsável técnico, CNPJ, CREA e cidade do autor fixos** no cabeçalho e na assinatura. A tela não tem campo de RT.
- **[PROVÁVEL]** As classes são rasterizadas pelo GDAL sem reprojeção: com camada de classes em CRS diferente do MDT, a máscara sai vazia.
- **[PROVÁVEL]** O método OSQP monta as restrições em laço Python, pixel a pixel: é lento e usa muita memória em MDTs grandes.
- **[PROVÁVEL]** O classificador roda o `processing.run` numa QThread com uma camada do projeto. Isso é instável em algumas versões do QGIS.
- Rótulos com apóstrofo quebram as expressões `CASE` do classificador [CONFIRMADO leitura].

### 5.5 IA

- **[PROVÁVEL]** O orquestrador roda **todo o pipeline em uma QThread**: cria camadas, chama processing e acessa o projeto fora da thread principal. É uma fonte possível de travamentos ou falhas aleatórias no QGIS.
- **[CONFIRMADO leitura]** A tela da IA exige `claude_api_key`, mesmo com o provedor OpenAI escolhido: quem só tem chave da OpenAI fica bloqueado.
- **[CONFIRMADO leitura]** Chaves de API:
  - a do Claude vai para o gerenciador de autenticação do QGIS **só se houver senha-mestra**; sem ela, fica em texto claro no QSettings;
  - a da OpenAI fica **sempre** em texto claro.
- **[A CONFIRMAR]** Os identificadores de modelo `claude-opus-5`, `claude-opus-4-8`, `claude-opus-4-7` e similares são fixos no código e precisam existir na API. Um ID inválido gera erro na chamada.
- O modo online envia ao provedor o contexto do projeto (vias, cotas, volumes, instrução do usuário).
- O custo usa o câmbio de um serviço externo, com cache na pasta de memória.

### 5.6 Interface e compatibilidade

- **[CONFIRMADO leitura]**
  - O botão **Abrir Manual DOCX** procura `manual.docx`, que não vem no pacote.
  - O `plugin.py` procura `icon.svg`, que também não existe (o ícone da barra fica vazio). O `metadata.txt` aponta `icon.png`.
  - Títulos de janela com versões antigas: "Suite v1.0", "Precisa Terraplanagem v3.5.2" e o `main_dialog` "v1.0".
- **[CONFIRMADO leitura]** O **diálogo de CRS aceita CRS geográfico** sem aviso. Distâncias, estacas e volumes perderiam o sentido.
- **[CONFIRMADO leitura]** O M1 continua sem MDT quando a interpolação falha: as estacas ficam sem `cota_tn` e o aviso aparece só no log. Cada execução adiciona **outra** camada "Estacas" ao projeto [PROVÁVEL duplicação].
- **[CONFIRMADO leitura]** `format_estaca`: um resto muito próximo do intervalo (por exemplo, 19,97 m com estaca de 20 m) sai "N+020.0" em vez de "(N+1)+000". É um caso de borda.
- **Qt6/QGIS 4:** ainda há 2 chamadas `exec_()` (tela da IA), 9 usos diretos de `QVariant.*` e importação de `backend_qt5agg` (com alternativa). A execução no QGIS 4 não foi testada [PROVÁVEL problemas pontuais].
- `core/__init__.py` lista os módulos `docx_helpers`, `thread_utils` e `ui_helpers`, que não existem (só documentação).
- **[CONFIRMADO leitura]** O rodapé da janela e alguns relatórios trazem "CREA-MG 141.370/D · Guiricema-MG" fixos.
- **Métodos hidrológicos:** o Racional não tem limite de área nem curva IDF (I digitado). Na morfometria, a ordem de Horton é estimada.

### 5.7 Não testado nesta revisão

- Todas as telas, o Editor de Malha ao vivo, a prancha/Compositor e o LandXML em software de campo.
- GRASS, IA online e offline, exportação DXF em CAD.
- Resultados numéricos contra projetos reais (o changelog cita medições que não puderam ser reproduzidas).
