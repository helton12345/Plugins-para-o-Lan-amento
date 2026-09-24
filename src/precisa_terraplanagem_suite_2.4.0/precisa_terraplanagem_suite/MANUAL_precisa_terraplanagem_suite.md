# Manual de uso — Precisa Terraplanagem Suite v2.4.0

Guia prático, na ordem das telas. Descreve o que o código faz hoje. Os problemas conhecidos estão no `DESCRITIVO_precisa_terraplanagem_suite.md`.

## 0. Antes de começar

- QGIS 3.16 ou superior. Para a aba **Bacias**, o QGIS precisa ter o **GRASS**.
- Bibliotecas Python usadas conforme a função: numpy, scipy, GDAL (vêm com o QGIS); **matplotlib** (gráficos), **python-docx** (DOCX), **reportlab** (PDF), **openpyxl** (Excel), **ezdxf** (DXF, com gravação manual como alternativa), **shapely**, **osqp** (só o método "Declividade Máxima" da DIP) e **Pillow**.
- Trabalhe em um **CRS projetado em metros** (ex.: SIRGAS 2000 / UTM). O plugin **não impede** o uso de CRS geográfico, mas as distâncias sairiam em graus.
- Dados típicos:
  - eixos das vias (linhas), com um campo de nome;
  - curvas de nível (linhas), com um campo de cota;
  - limite do levantamento (polígono, opcional);
  - lotes e limite da gleba (para a IA e para a drenagem dos lotes);
  - MDT raster (para a DIP e para as Bacias).
- Salve o projeto QGIS: os parâmetros e os resultados intermediários ficam gravados dentro do `.qgz`.

## 1. Abrir o plugin

- Pelo menu **Precisa Agrimensura → 📁 Projetos → Terraplanagem Suite** (menu da Suíte) ou pelo ícone da barra de ferramentas.
- A janela principal tem quatro abas: **🏗️ Loteamentos**, **⛰️ DIP Adaptativa**, **💧 Bacias** e **⚙️ Configurações**.
- No rodapé ficam **🤖 Configurar IA** e **Fechar**.
- **CRS de trabalho:** ao clicar em qualquer módulo pela primeira vez (exceto **💾 Sessão**), abre **Definir CRS de Trabalho**. Você pode:
  - usar o CRS do projeto;
  - usar o CRS de uma camada;
  - escolher o CRS manualmente.

  A escolha fica salva no projeto.

## 2. Aba ⚙️ Configurações (recomendado primeiro)

1. **📋 Checklist de Execução → Abrir Checklist**: lista os arquivos de entrada e mostra quais parâmetros já foram configurados e quais estão no padrão.
2. **📋 Parâmetros do Projeto → Configurar Parâmetros**: abas Identificação, M1, M2, M3 e M4. Ali você define:
   - intervalo de estacas;
   - declividades mínima e máxima (padrão da configuração: 1% a **27%**);
   - gabarito, taludes (padrão da configuração: **1:1** em corte e aterro) e fatores de homogeneização;
   - método de volume (áreas médias ou prismoidal);
   - velocidade de projeto (40 km/h);
   - espessura do pavimento (0,30 m) e demais parâmetros.
3. **🤖 Co-piloto IA → Configurar IA**:
   - **Online:** Claude (chave da Anthropic) ou OpenAI (chave própria).
   - **Offline:** Ollama, com URL (padrão `http://localhost:11434`) e modelo; o botão **🔍 Detectar modelos instalados** lista os modelos.
   - **Modelo "fino"** (para as decisões) e **modelo "rápido"**.
   - **Pasta de dados da IA** (opcional; por exemplo, uma pasta do Google Drive). Guarda o histórico de ações e o consumo de tokens.
   - **Testar conexão Claude** envia uma mensagem de teste.
4. **📖 Documentação → Abrir Manual DOCX**: procura `manual.docx` na pasta do plugin. Esse arquivo **não vem no pacote**, então aparece o aviso "manual.docx não encontrado".

> ⚠️ Os módulos M3 e o relatório do M5 têm padrões próprios na tela (talude **1,5 em corte e 2,0 em aterro**) quando não há Seção Tipo aplicada. Aplique uma **📐 Seção Tipo** (item 3.6) para que todos usem o mesmo gabarito.

## 3. Aba 🏗️ Loteamentos — fluxo manual M1 → M5

A ordem obrigatória é **M1 → M2 → M3 → M4 → M5**. A partir do M2, o plugin verifica se a etapa anterior foi feita e avisa se faltar.

### 3.1 M1 — Alinhamento e Estaqueamento

Abre uma janela modal com três abas.

**Aba ⚙️ Configuração**

- **Camadas de Entrada:**
  - eixos das vias e o campo com o nome da rua (em branco = automático);
  - curvas de nível e o campo de cota;
  - limite do levantamento (opcional). Com limite, o MDT é recortado pela propriedade.
- **🔄 Atualizar camadas** recarrega as listas.
- **Parâmetros:**
  - intervalo (padrão 20 m);
  - resolução do MDT (1 m/pixel);
  - tolerância de interseção (0,5 m);
  - estacas intermediárias.
- **Pasta de Saída:** CSV de estacas e CSV de resumo por via.

Clique em **▶ Processar Estaqueamento (todas as ruas)**. O plugin então:

1. valida as curvas de nível. Se houver erro grave, para e lista os problemas;
2. gera o MDT por TIN (reaproveitado do cache quando nada mudou);
3. cria a camada **Estacas** com os campos `id_global`, `id_via`, `nome`, `numero`, `dist_acum`, `cota_tn`, `cota_proj`, `cota_verm`, `rua`, `azimute`, `tipo` e `intersecao`;
4. grava os CSVs, se marcados.

A numeração recomeça em `0+000` em cada via (formato `N+DDD`).

> Se o MDT não puder ser gerado, o M1 **continua** e as estacas ficam sem cota do terreno. Confira o log.

**Aba 📋 Por Via**: tabela com o número de estacas e o comprimento de cada via.

**Aba 🗺️ Terreno Projetado**: gera o MTP (modelo do terreno projetado) pelo método antigo, depois do M2. Pede:

- MDT natural;
- largura da plataforma;
- taludes;
- resolução.

Clique em **Gerar Terreno Projetado**.

### 3.2 M2 — Greide Interativo 🤖

Janela **não modal**: você pode usar o mapa ao mesmo tempo.

1. Em **Camada de Estacas (M1)**, escolha a camada e clique em **▶ Carregar Vias**.
2. Em **Via Ativa**, escolha a via.
3. Ajuste a declividade mínima e máxima.
4. Edite os **PVIs** na tabela ou **arrastando no gráfico**:
   - **+ PVI** / **− PVI** / **↕ Ordenar**;
   - roda do mouse para zoom e arraste em área vazia para mover a vista.
5. Clique em **✔ Calcular Greide de Todas as Vias**.
   - Vias com menos de 2 PVIs são ignoradas, depois de uma confirmação.
   - As cotas de projeto são gravadas na camada de estacas e os PVIs ficam salvos no projeto.
   - As curvas verticais usam o K mínimo pela velocidade de projeto (AASHTO), com comprimento arredondado para múltiplos de 20 m.
6. Exportação:
   - **🖼️ Perfil (PNG)**;
   - **📄 Notas de Serviço (CSV)**;
   - **📐 Perfil Longitudinal (DXF)**, com exagero vertical ajustável.
7. **Painel IA:** a IA pode sugerir ajustar, adicionar ou remover PVIs. Cada sugestão tem o botão **Aplicar**.

### 3.3 M3 — Seções, Offsets, Curvas de Nível e Cul-de-sac 🤖

**Aba 📐 Seções e Offsets**

- **Entradas:**
  - camada de estacas (com o greide do M2);
  - MDT natural (se ficar em branco, usa o último MDT do cache);
  - limite da pista (polígonos, opcional).
- **Seção Tipo:** plataforma, faixa, calçada, taludes, altura máxima sem banqueta e largura da seção. Se houver gabarito aplicado, ele é usado.
- **Saída:** a pasta é **obrigatória** (as camadas são salvas em `.gpkg`). Opções:
  - pontos de offset;
  - linhas de offset;
  - polígonos de talude;
  - caderno de seções (DXF + PDF, uma seção por folha), com exagero vertical.
- **🌐 Superfície Projetada:** gera o MTP e as curvas de nível automaticamente. Opções:
  - intervalo das curvas e das curvas mestras;
  - pixel;
  - simplificação;
  - curvas do terreno natural;
  - raster de diferença.
- Clique em **▶ Calcular Seções e Offsets**.

**Aba 📏 Curvas de Nível**

- Entradas: MDT natural, MTP projetado, intervalos e simplificação.
- Escolha curvas do terreno natural e/ou do projetado e se quer aplicar o estilo.
- Clique em **📏 Gerar Curvas de Nível**.

**Aba 🔄 Cul-de-sac**

- Parâmetros:
  - camada de eixos;
  - tipo (circular simples, circular com ilha, hammerhead ou bulbo elíptico);
  - raio externo e raio da ilha;
  - tolerância;
  - detecção automática das extremidades livres.
- Clique em **🔄 Gerar Cul-de-sacs**.

**Painel IA:** sugere talude de corte e de aterro e banqueta.

### 3.4 M4 — Volumes e Diagrama de Brückner 🤖

- **Aba ⚙️ Parâmetros:**
  - fatores de homogeneização por categoria;
  - categoria do material (1ª, 2ª ou 3ª);
  - distância limite de transporte.
- **▶ Calcular Volumes**: lê as seções gravadas pelo M3 e monta o quadro trecho a trecho (áreas médias, ou prismoidal se configurado).
- **Aba 📊 Quadro de Volumes**: totais e tabela. O saldo acumulado aparece em verde (sobra) ou vermelho (falta).
- **Aba 📈 Diagrama de Brückner**: o gráfico.
- **Exportação:** **📊 Excel** e **🖼️ Brückner PNG**.
- **Painel IA:** zonas de empréstimo e de bota-fora, distância de transporte, categoria e fator de homogeneização.

### 3.5 M5 — Relatório Técnico Final

1. Preencha os **Dados do Empreendimento**: nome, município/UF, responsável técnico, CREA/CAU/RRT e data. Clique em **💾 Salvar dados no projeto**.
2. Marque as seções que vão no relatório:
   - parâmetros;
   - notas de serviço;
   - volumes + Brückner;
   - platôs;
   - alertas;
   - referências.
3. Escolha o **formato** (PDF, DOCX ou ambos) e a pasta de saída.
4. Clique em **▶ Gerar Relatório**.

> Veja no Descritivo §5.2 os campos que podem sair zerados quando o M4 é feito pela tela manual.

### 3.6 Ferramentas auxiliares (grade abaixo dos módulos)

| Botão | O que faz |
|---|---|
| 📐 **Seção Tipo** | Editor do gabarito da via: elementos do eixo para fora, simetria, taludes e banqueta, pré-visualização com terreno simulado, **Salvar/Carregar JSON** e **✔ Aplicar Seção Tipo** (grava no projeto). |
| 🟡 **Platôs** | Platôs de lotes. **Cota:** média, mínima ou máxima do terreno natural, ou manual. **⚖ Ajustar Plano (mínima terra)** por mínimos quadrados. **Rampas de acesso.** **Camadas geradas:** perímetro com volumes, taludes, manchas de corte e aterro, rampas e pontos de locação. **MTP unificado** (ruas + platôs), **empolamento/caçambas** e **PDF resumo**. |
| 📍 **Locação** | Gera os pontos de campo (plataforma e taludes) e exporta em **TXT** (estação total), **DXF** (GNSS) e **XLSX**, com opção de um arquivo por via ou por tipo. Use **▶ Gerar Pontos** e depois **💾 Exportar**. |
| 🔄 **Cul-de-sac** | Abre o M3 direto na aba Cul-de-sac. |
| ✕ **Interseções** | Detecta cruzamentos na camada de estacas, mostra alertas e exporta CSV. |
| ↩ **Superelevação** | Detecta curvas horizontais pelo azimute das estacas, calcula superelevação e superlargura e exporta CSV. |
| ⚠️ **Alertas** | Abre um painel lateral com os alertas, com filtro por módulo. |
| 📤 **Exportar DXF** | Exporta as camadas escolhidas do projeto para um DXF (padrão AutoCAD R2000), com texto opcional. |
| 📐 **Produtos Finais** | **Prancha** planialtimétrica no Compositor (folha automática, A3 a A0, escala e hachura), com exportação em PDF. **Dados de campo:** notas de serviço (CSV/TXT) e **LandXML** (superfície + eixo). **Verificar drenagem dos lotes:** pede lotes, ruas, curvas de nível projetadas e, opcionalmente, o MDT do terreno natural. |
| 🔺 **Editor de Malha** | **📤 Exportar malha para edição:** pontos, arestas coloridas pela declividade e breaklines. **Edição ao vivo:** ✂️ excluir vértice, 📐 desenhar breakline e ➕ incluir ponto, com a cota sugerida pelo greide. Depois, reconstrói a malha, o MTP, os volumes e o relatório. A janela não é modal. |
| 💾 **Sessão** | Pasta de saída do projeto, informações do `.qgz`, **JSON externo** (para levar o projeto a outra máquina) e o estado atual (o que já foi executado). Não exige CRS definido. |

## 4. Terraplanagem IA (botão roxo no topo da aba Loteamentos)

1. **Camadas:**
   - eixo da rua, lotes, limite e curvas de nível;
   - opcionais: servidão, meio-fio e limite da calçada.
2. **Perfil de Projeto → ✏️ Editar perfil…** Padrão:
   - pista de 7 m, calçada de 1,5 m;
   - declividade longitudinal de 2% a 25%;
   - talude 1:1, com 5 m de altura máxima;
   - lote: 45% proibido, alvo 30%;
   - talude entre lotes de 1 m;
   - cruzamentos, camadas de borda e recursos (cul-de-sac, superelevação, pontos baixos).

   O perfil pode ser salvo e carregado em `.json`.
3. **Modo:**
   - **Assistido:** confirma cada ação.
   - **Autônomo:** só pergunta quando a IA não tem certeza ou quando há conflito.
4. **Pasta de saída:** checkpoints, cópias versionadas e relatórios de bug.
5. **Instrução** (opcional, em português).
6. Clique em **▶ Executar**. Use **⏸ Parar** para interromper.

O fluxo roda M1, interseções, greide, seções, superelevação, volumes, platôs, cul-de-sac, MTP e curvas, mapa de corte e aterro, perfil em PDF, seções em PDF, DXF e relatório. No fim, adiciona as camadas ao projeto e mostra o custo estimado da IA.

> ⚠️ Ao iniciar, os valores do perfil **sobrescrevem** os parâmetros do projeto (por exemplo, a declividade máxima passa de 27% para 25%). Revise a Configuração depois.

## 5. Aba ⛰️ DIP Adaptativa (terraplanagem por classe de declividade)

### 5.1 📊 Classificar Declividade

1. Escolha o **MDT (raster)**.
2. Escolha a **unidade**. ⚠️ O padrão é **Graus**, mas os intervalos padrão estão em **%**: **mude para "Porcentagem (%)"** antes de processar.
3. Ajuste os intervalos (De, Até, Rótulo, Cor), com **＋ Adicionar**, **－ Remover** e **⟳ Padrão**. O padrão tem 6 classes, de 0–8% (plano) a mais de 100% (escarpado).
4. Defina o **shapefile de saída** e as opções:
   - simplificar (tolerância 2 m);
   - suavizar (Chaikin, 5 iterações);
   - área mínima de ilha (500 m²).
5. Clique em **▶ Processar**.
6. O painel IA (abaixo) pode sugerir uma estratégia por classe. Isso apenas **colore a tabela**: não altera o processamento.

### 5.2 ⛰️ Terraplanagem por Classe

1. **Tipo de entrada:** MDT raster, curvas de nível ou pontos cotados. Para vetor, informe também o campo Z e a resolução.
2. Em **Camada de classes**, use o shapefile da etapa 5.1 e clique em **Carregar classes da camada** (exige o campo `classe_id`).
3. **Método:**
   - **Plano Inclinado:** declividade exata, curvas retas.
   - **Declividade Máxima:** preserva a curvatura; requer `osqp`.

   Opcional: **arredondar os limites** (raio).
4. Na tabela, marque **Terraplanar?** e informe a **Decl. alvo (%)** de cada classe.
5. **Taludes de transição:** corte 1:1, aterro 1,5:1 (H:V), suavização por cosseno.
6. Defina a **pasta de saída** e clique em **Processar Terraplanagem**. O resultado é um MDT parcial por classe, com os volumes de cada uma.

### 5.3 🔧 Finalização / Mescla

1. Confira a lista de **MDTs parciais**. **+ Adicionar MDT externo** só funciona com um GeoTIFF da **mesma grade** do MDT original; os outros são ignorados.
2. Defina a **equidistância das curvas** (1 m).
3. Relatório DOCX (opcional):
   - obra;
   - eixo (linha);
   - espaçamento e semi-largura das seções;
   - perfil e seções no DOCX e/ou como camadas;
   - layout e exagero vertical;
   - Brückner.
4. Clique em **Mesclar + Volumes + Curvas**.

Saídas na pasta:

- `mdt_final.tif`;
- `curvas_nivel_final.gpkg`;
- `relatorio_terraplanagem.docx`;
- PNGs;
- `perfis.gpkg`.

## 6. Aba 💧 Bacias

1. **🗺️ Entrada:**
   - MDE raster;
   - threshold de canal (1000 células);
   - resolução (apenas informativa).

   Clique em **▶ Processar Bacias** (botão abaixo das abas). O plugin usa o GRASS: remoção de depressões, r.watershed, declividade, TWI e TPI.
2. **📊 Morfometria:** área, perímetro, comprimento da drenagem, número de canais e índices.
3. **💧 Greide vs Drenagem:** compara o greide com a drenagem natural (declividade mínima de 0,5%).
4. **🌧️ Hidrologia:**
   - **Tc** por Kirpich ou Giandotti;
   - **vazão pelo Método Racional**, Q = C·I·A/3,6. A intensidade **I** é digitada (não há curva IDF).
5. **📋 Relatório:** texto, que pode ser salvo.

Use **➕ Adicionar ao Projeto** para carregar as camadas.

## 7. Dicas rápidas

- Salve o projeto `.qgz` depois de cada módulo. O M2 salva os PVIs automaticamente.
- Se o M4 disser "Execute o Módulo 3 primeiro", recalcule o M3 no mesmo projeto.
- Para levar o projeto a outra máquina, use **💾 Sessão → JSON externo** e copie também a pasta de saída (com os `.gpkg`).
- A IA online envia o contexto do projeto (parâmetros, cotas, volumes) ao provedor escolhido. Use o modo offline (Ollama) se isso for um problema.
