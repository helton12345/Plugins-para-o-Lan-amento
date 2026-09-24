# Prompts de revisão — um por plugin

Cole cada bloco em uma sessão nova do Claude e anexe o ZIP do plugin correspondente.
Ordem sugerida por risco: GEO_RURAL → Topografia → Terraplanagem → Drenagem → Esgoto → Abastecimento → demais.

---

## 1. GEO_RURAL

````text
Você vai revisar e corrigir o plugin QGIS **GEO_RURAL** (versão atual 3.199).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/GEO_RURAL_v3_199/plugin_geo_rural_3_59/DESCRITIVO_plugin_geo_rural_3_59.md` e `src/GEO_RURAL_v3_199/plugin_geo_rural_3_59/METADATA_DIVERGENCIA_plugin_geo_rural_3_59.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Tabelas de método PG inconsistentes entre o leitor TSV e o exportador ODS; FLUTUANTE vira PG3 e CONTROLE vira PG6.
- Rota "polígono sem GNSS" aplica σ e método inventados (combo começa em PG1); marcos novos da divisão/unificação saem com σ = 0.
- Altitude SRTM (ortométrica) gravada como elipsoidal, automaticamente no fluxo multiparcela.
- RG/CPF do autor fixos em anuências, requerimentos e rodapés; textos afirmam fatos automaticamente (sem sobreposição, anuências obtidas, "não sou PEP").
- Numeração M duplicada ao converter vários vértices de uma vez (proximo_numero só é gravado ao fechar).
- Unificação com credencial "VQUE" fixa.
- Kit PPK/RBMC: baixar_efemerides() e baixar_ionex() dão TypeError (os.path.basename(dict)).
- Conversão assume hemisfério Sul; SHP importado sem reprojeção.
- Sessão não restaura RG/CPF do RT nem os condôminos; autosave grava uma vez só.
- Conflito de self.tbl_parcelas entre a aba Gestão e a Fase 5.
- fase5_divisao/test_divisao.py TEST 14 falha (teste desatualizado).

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 3.199 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 2. Topografia

````text
Você vai revisar e corrigir o plugin QGIS **Topografia** (versão atual 1.0.4).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/precisa_topografia_v1.0.4/precisa_topografia/DESCRITIVO_precisa_topografia.md` e `src/precisa_topografia_v1.0.4/precisa_topografia/METADATA_DIVERGENCIA_precisa_topografia.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Poligonal calcula com os ângulos deslocados de um vértice (bug confirmado).
- Tipo "Aberta" sem efeito (_on_tipo_poligonal vazio).
- Convenção fixa de ângulos internos horários, sem opção.
- Irradiação pressupõe círculo zerado na ré.
- DXF declara ET_ESTACOES mas não grava estações; relatório imprime "Não informado" indevidamente; título sempre "POLIGONAL FECHADA".

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 1.0.4 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 3. Terraplanagem

````text
Você vai revisar e corrigir o plugin QGIS **Terraplanagem** (versão atual 2.4.0).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/precisa_terraplanagem_suite_2.4.0/precisa_terraplanagem_suite/DESCRITIVO_precisa_terraplanagem_suite.md` e `src/precisa_terraplanagem_suite_2.4.0/precisa_terraplanagem_suite/METADATA_DIVERGENCIA_precisa_terraplanagem_suite.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- M3 grava secoes_json sem 'secao_confiavel' e o diálogo do M4 descarta o campo: a exclusão de seções não confiáveis nunca ocorre no fluxo manual.
- M4 grava volumes_json com chaves diferentes das que o M5 lê: áreas, volume homogeneizado, saldo acumulado e Brückner por via saem zerados no relatório.
- platos_resumo_json nunca é gravado: "Resumo de platôs" do M5 sempre vazio.
- M3 acumula alertas_json a cada execução (alertas duplicados).
- M3 manual tem taludes fixos 1,5/2,0 na tela e não lê a Configuração (1:1).
- Orquestrador de IA sobrescreve a Configuração do projeto com o perfil, sem aviso; exige chave Claude mesmo com OpenAI.
- DIP: unidade padrão Graus com intervalos em %; ações da IA na DIP não afetam o processamento; np.trapz quebra no NumPy 2.4.
- Diálogo de CRS aceita CRS geográfico sem aviso.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 2.4.0 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 4. Drenagem

````text
Você vai revisar e corrigir o plugin QGIS **Drenagem** (versão atual 0.9.1).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/precisa_drenagem_0.9.1/precisa_drenagem/DESCRITIVO_precisa_drenagem.md` e `src/precisa_drenagem_0.9.1/precisa_drenagem/METADATA_DIVERGENCIA_precisa_drenagem.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Trecho sem área gera declividade de 50% (confirmado por teste).
- Memória de cálculo: coluna "Q própria (L/s)" errada.
- Delineação Whitebox pode contar área em dobro (provável).
- Memória sem dados do projeto; síntese "ATENDE" enganosa; texto não corresponde ao método.
- Recobrimento padrão divergente; prof_max 3,00 m não editável; sem segmentação (cruzamentos no meio da linha não viram PV).

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 0.9.1 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 5. Esgoto

````text
Você vai revisar e corrigir o plugin QGIS **Esgoto** (versão atual 1.0).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/precisa_esgoto_1.0/precisa_esgoto/DESCRITIVO_precisa_esgoto.md` e `src/precisa_esgoto_1.0/precisa_esgoto/METADATA_DIVERGENCIA_precisa_esgoto.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Memória afirma conformidade de forma fixa.
- Capacidade excedida é escondida (só alerta de lâmina 0,75).
- Tensão trativa calculada com a vazão de fim de plano.
- Sentido do escoamento só por cota de terreno (diálogo nunca passa no_descarga).
- Recobrimento único (servidão 0,20 m nunca passado); nó fora do MDT recebe cota 0.
- Camadas do mapa desatualizadas; parâmetros da memória relidos na exportação.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 1.0 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 6. Abastecimento

````text
Você vai revisar e corrigir o plugin QGIS **Abastecimento** (versão atual 0.5.0).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/precisa_abastecimento_0.5.0/precisa_abastecimento/DESCRITIVO_precisa_abastecimento.md` e `src/precisa_abastecimento_0.5.0/precisa_abastecimento/METADATA_DIVERGENCIA_precisa_abastecimento.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- A origem da rede não é o lote do reservatório (orientar_arvore_de_fonte sem no_fonte).
- Memória de cálculo com parâmetros diferentes dos calculados.
- Recalque fora da memória e usando a população do campo manual.
- Malhas: arestas de fechamento descartadas, sem Hardy-Cross.
- Pressão estática não verificada; DN escolhido só pela V máx; C de Hazen-Williams diverge entre núcleo e tela.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 0.5.0 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 7. Conversor de Memoriais Pro (gerador_de_memoriais)

````text
Você vai revisar e corrigir o plugin QGIS **Conversor de Memoriais Pro (gerador_de_memoriais)** (versão atual 1.4).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/Gerador_de_Memoriais_v1.4/gerador_de_memoriais/DESCRITIVO_gerador_de_memoriais.md` e `src/Gerador_de_Memoriais_v1.4/gerador_de_memoriais/METADATA_DIVERGENCIA_gerador_de_memoriais.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Vértices tipo "M" ignorados no SIGEF (regex).
- Área/perímetro errados em camadas geográficas (camadas.py).
- Formato numérico não brasileiro no modo Camada.
- Modo Camada descreve só o 1º anel do 1º polígono.
- Falso positivo de curva no SIGEF; textos fixos ("DIVISÃO DE IMÓVEL RURAL").
- Obs.: o nome comercial passou a ser "Conversor de Memoriais Pro" só nos documentos; NÃO renomear código/pasta sem pedir.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 1.4 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 8. DANI

````text
Você vai revisar e corrigir o plugin QGIS **DANI** (versão atual 6.6).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/DANI_v6.6/DANI_v6_0/DESCRITIVO_DANI_v6_0.md` e `src/DANI_v6.6/DANI_v6_0/METADATA_DIVERGENCIA_DANI_v6_0.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Reabrir "Configurar Camadas" apaga os Dados do Loteamento.
- Autosave grava uma só vez por sessão.
- Scripts GEO/TABULAR/PLANILHA sobrescrevem o campo AREA da camada de lotes e gravam campos DANI_* (commitChanges).
- Filtro de ângulo dos confrontantes não filtra; detectores de curva diferem entre scripts.
- Segmentos sem lado somem da descrição; TOTAL GERAL em dobro no XLSX; área muda de fonte entre saídas.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 6.6 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 9. GEO URBANO

````text
Você vai revisar e corrigir o plugin QGIS **GEO URBANO** (versão atual 1.6.0).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/geo_urbano_v1_6_0/geo_urbano/DESCRITIVO_geo_urbano.md` e `src/geo_urbano_v1_6_0/geo_urbano/METADATA_DIVERGENCIA_geo_urbano.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Fuso e meridiano central errados nas plantas (planta.py).
- CPF dos proprietários não sai nos memoriais.
- "Parte menor/maior → Desmembrado" não garantido; SRC não validado.
- Curvas quase nunca detectadas; segmentos sem lado não entram na descrição.
- Trava de fechamento nunca dispara; lotes com vértice em "T" falham.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 1.6.0 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 10. CTM (BCI)

````text
Você vai revisar e corrigir o plugin QGIS **CTM (BCI)** (versão atual 1.0.1).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/plugin_ctm_v1.0.1/plugin_ctm/DESCRITIVO_plugin_ctm.md` e `src/plugin_ctm_v1.0.1/plugin_ctm/METADATA_DIVERGENCIA_plugin_ctm.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Campos opcionais bloqueiam o cálculo.
- Aba errada após o cálculo (setCurrentIndex(2)).
- Camada de Quadras recebida mas não usada.
- Testada superestimada; orientação da rua simplificada.
- Sem verificação de CRS; acesso à camada dentro da QgsTask; conexões layersAdded/Removed nunca desconectadas.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 1.0.1 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 11. Batimetria

````text
Você vai revisar e corrigir o plugin QGIS **Batimetria** (versão atual 1.0).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/precisa_batimetria/precisa_batimetria/DESCRITIVO_precisa_batimetria.md` e `src/precisa_batimetria/precisa_batimetria/METADATA_DIVERGENCIA_precisa_batimetria.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Não funciona sem a Suíte Precisa instalada (dialog.py).
- Opção "Exportar raster MDT (GeoTIFF)" sem efeito.
- Sem reprojeção entre camadas (máscara).
- Cota mínima e profundidade máxima vêm do usuário, não do MDT.
- Regressão exige ≥3 faixas; perfil alinhado aos eixos do raster; temporários não removidos; NA/fundo padrão herdados de projeto real.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 1.0 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 12. Sedimentação

````text
Você vai revisar e corrigir o plugin QGIS **Sedimentação** (versão atual 1.2).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/precisa_sedimentacao_pro_v1.2/precisa_sedimentacao/DESCRITIVO_precisa_sedimentacao.md` e `src/precisa_sedimentacao_pro_v1.2/precisa_sedimentacao/METADATA_DIVERGENCIA_precisa_sedimentacao.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Popup de reprojeção nunca é chamado; comparação de SRC por authid dá falso positivo.
- QGIS mínimo real é 3.20 (writeAsVectorFormatV3), metadata diz outro.
- Perda de capacidade "útil" usa o ΔV total; série temporal mistura grandezas.
- Curva CAV só na Forma A; Forma C usa área de células como área da poligonal.
- Temporários da máscara não removidos.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 1.2 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 13. HidroQGIS

````text
Você vai revisar e corrigir o plugin QGIS **HidroQGIS** (versão atual 1.1.0).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/HidroQGIS_pro/hidroqgis/DESCRITIVO_hidroqgis.md` e `src/HidroQGIS_pro/hidroqgis/METADATA_DIVERGENCIA_hidroqgis.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Ícone ausente (resources/icon.png não existe).
- Publicação Netlify sem _headers.
- JavaScript do visualizador quebra sem bacias (ReferenceError).
- Fallback da drenagem gera polígonos.
- "Corpos d'água" = limiar de fluxo acumulado (nomenclatura enganosa); textos fixos; linha TOTAL do perímetro.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 1.1.0 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 14. Earth Engine

````text
Você vai revisar e corrigir o plugin QGIS **Earth Engine** (versão atual 1.0.0).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/precisa_earth_engine/precisa_earth_engine/DESCRITIVO_precisa_earth_engine.md` e `src/precisa_earth_engine/precisa_earth_engine/METADATA_DIVERGENCIA_precisa_earth_engine.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Limite de download do getDownloadURL corta imagens grandes.
- filterDate: data final exclusiva.
- Sentinel-2 sem máscara de nuvem por pixel.
- Links de streaming expiram; busca STAC sem paginação (limit 100).
- Asset escolhido por heurística; cena inteira baixada antes do recorte; não integrado à Suíte (sem chaves precisa_*).

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 1.0.0 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 15. GEO Visualizador

````text
Você vai revisar e corrigir o plugin QGIS **GEO Visualizador** (versão atual 0.15).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/geo_visualizador_v0.15_corrigido/geo_visualizador/DESCRITIVO_geo_visualizador.md` e `src/geo_visualizador_v0.15_corrigido/geo_visualizador/METADATA_DIVERGENCIA_geo_visualizador.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- Simplificação 0,5 na unidade do CRS (em graus deforma).
- Modo imagem marcado como não testado.
- Cada publicação cria um site novo (sem atualização).
- Publicação roda na thread da interface.
- Lote ausente no modo venda; fundo Google sem atribuição; API key só em base64.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 0.15 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````

---

## 16. Assistente IA Code Runner (restante)

````text
Você vai revisar e corrigir o plugin QGIS **Assistente IA Code Runner (restante)** (versão atual 1.21.2).

## Arquivos
- Vou anexar o ZIP do plugin nesta conversa. Se não houver anexo, PARE e peça o ZIP.
- Diagnóstico já feito (leia antes de tudo): `src/assistente_ia_code_runner_v1.21.1/assistente_ia_code_runner/DESCRITIVO_assistente_ia_code_runner.md` e `src/assistente_ia_code_runner_v1.21.1/assistente_ia_code_runner/METADATA_DIVERGENCIA_assistente_ia_code_runner.md` no repositório `helton12345/Plugins-para-o-Lan-amento` (branch `claude/new-session-zdxlyf`). Não suba o código do plugin para o repositório (contém dados pessoais).

## Problemas já levantados (confirme cada um no código antes de corrigir)
- JÁ FEITO na 1.21.2: senha (Bearer) no MCP e recusa de Origin. Não refazer.
- Ferramentas GitHub (enviar_arquivo_repositorio, atualizar_memoria_permanente) sem confirmação.
- Restrição de pasta parcial durante o exec; modo Automático pula confirmação de limpeza.
- Erros de rede das ferramentas GitHub quebram o turno (400 no Claude).
- Nova sessão não encerra o Claude Code local; pip install automático sem perguntar; chaves em texto simples.

## Regras (obrigatórias)
1. Antes de alterar qualquer arquivo, me diga o que pode quebrar e por quê. NÃO edite até eu confirmar.
2. Diagnostique e explique a causa raiz de forma sucinta antes de propor a correção. Nada de tentativa e erro.
3. Menor mudança possível: só o que eu aprovar. Não refatore, não "melhore" nem reorganize outros módulos.
4. Depois de editar, liste exatamente arquivos, funções e linhas alteradas.
5. Se algo for arriscado ou você estiver em dúvida, pare e pergunte.
6. Respostas curtas e diretas.
7. Não altere textos jurídicos/normativos nem cálculos que dependam de norma sem me mostrar a norma e o antes/depois.

## Fluxo
1. Leia o DESCRITIVO e todo o código envolvido.
2. Me entregue uma tabela: problema | causa raiz (arquivo:linha) | correção proposta | o que pode quebrar | prioridade (alta/média/baixa). Espere minha escolha.
3. Corrija só os itens aprovados. Para cálculo, mostre um exemplo numérico antes/depois.
4. Rode testes que existirem (ou um teste mínimo sem QGIS, simulando o que for preciso) e mostre o resultado.
5. No final: suba a versão em `metadata.txt` (patch, ex.: 1.21.2 → próxima) e acrescente uma linha no changelog, se existir.
6. Entregue o ZIP do plugin INTEIRO pronto para "Instalar a partir de ZIP" (sem __pycache__ e sem os arquivos MANUAL_/DESCRITIVO_/METADATA_DIVERGENCIA_) e a lista final de alterações.
````
