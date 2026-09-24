# Changelog — Gerador de Memoriais

## [1.4]

### Corrigido — modo Texto
- **Vértice perdido em quebra de página**: .txt exportado de PDF costuma
  repetir timbre/rodapé a cada página; quando esse texto caía no meio da
  declaração de um vértice (ex.: entre "N" e o valor da coordenada), o
  regex não casava e o vértice sumia do resultado sem aviso. Passa a
  remover automaticamente linhas repetidas (timbre, rodapé, marcador
  "--- Página N ---") antes de interpretar o texto.
- Novo aviso: se algum vértice citado no texto não teve as coordenadas
  reconhecidas (quebra de página/timbre fora do padrão comum), mostra
  quais códigos revisar manualmente em vez de descartar em silêncio.

### Adicionado — modo Texto
- Carregar memorial direto de um arquivo **.pdf** (além de .txt/.docx).
- Confrontante passa a reconhecer também a frase "segue confrontando com
  X" (sem "neste trecho"), e a valer para todos os vértices do mesmo
  trecho até a próxima declaração — antes só reconhecia declaração
  repetida a cada vértice.

## [1.3]

### Adicionado — modo Texto
- Passa a procurar a declaração do fuso UTM (ex.: "Fuso 23S") no próprio
  texto colado do memorial e ajusta automaticamente o CRS antes de gerar
  pontos/polígono, em vez de depender só do padrão fixo do seletor de CRS
  (EPSG:31983 — fuso 23S). Coordenadas N/E sozinhas não indicam a qual dos
  60 fusos UTM pertencem; usar o fuso errado desloca o polígono no mapa.
- Se o texto não declarar o fuso, mostra um aviso pedindo para confirmar
  manualmente o CRS selecionado antes de continuar, em vez de gerar
  silenciosamente com o padrão.

## [1.2]

### Corrigido — modo Camada QGIS
- **Dados pessoais do autor vazando para o memorial de terceiros**: cidade
  ("Guiricema"), nome e registro profissional do autor do plugin estavam
  fixos no código e apareciam automaticamente no cabeçalho e na assinatura
  de qualquer memorial gerado por qualquer usuário, independente de quem
  realmente o gerou. Adicionados campos "Cidade", "Data", "Nome" e
  "Registro profissional" na aba Camada QGIS; se deixados em branco, o
  memorial usa `[PREENCHER ...]` (mesmo padrão já usado para proprietário/
  confrontante) em vez de assumir a identidade do autor do plugin.
- **Fuso UTM incorreto**: o fuso era estimado por uma faixa fixa de códigos
  EPSG que só cobria parte dos casos e produzia número de fuso errado (ex.:
  camadas em EPSG:31984 saíam com "Fuso 12" em vez de "Fuso 24S"). Passa a
  ler o fuso e o hemisfério diretamente dos parâmetros da projeção da CRS
  da camada (`+proj=utm +zone=NN [+south]`), válido para qualquer datum.

## [1.0] — versão inicial pública

Fusão de três plugins anteriores do autor, descontinuados nesta versão:
- **Restituição** (v1.2) — modo Texto, sem alteração de lógica.
- **Memorial Rural Gerador** (v1.1) — modo Camada QGIS. Removida a geração
  de memorial tabular com classificação automática de lado (frente/fundos/
  laterais); mantida a geração georreferenciada com detecção de curva.
- **Memorial SIGEF** (v1.0) — modo SIGEF. Mantido integralmente (leitura de
  PDF, cruzamento de confrontantes, narrativa com curva); adicionada geração
  de pontos e polígono (novidade desta versão).

Novidade: declaração automática do sistema de cálculo (UTM, Geográfico ou
Plano Topográfico Local) no texto do memorial, nos modos Camada QGIS e SIGEF.

## [1.1]

- Modo Camada QGIS: aviso obrigatório se a camada não estiver em CRS UTM
  (bloqueia geração com confirmação, evitando memorial com medidas erradas).
- Modo SIGEF: removida a extração/cruzamento de confrontantes (não pede mais
  um segundo PDF). Confrontante fica como [PREENCHER CONFRONTANTE], igual ao
  modo Camada QGIS. Classificação/extração de confrontante passa a ser
  exclusividade dos produtos pagos (GEO RURAL / DANI).
