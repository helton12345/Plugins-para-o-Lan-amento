# Manual de Instruções — Precisa Topografia

**Pasta:** `precisa_topografia` · versão declarada 1.0.4

---

## 1. Para que serve

Três ferramentas de topografia de campo num só plugin:

| Ferramenta | O que faz |
|---|---|
| **Caderneta de Estação Total** | Lê a caderneta Topograph (.B/.B00…) exportada pelo TransferCPE, calcula E/N/Z de todos os pontos e gera camada, CSV e DXF |
| **Poligonal e Irradiação** | Calcula poligonal fechada com fechamento angular/linear e ajuste por Bowditch, e irradiação simples a partir de uma estação |
| **Relatório de Levantamento** | Monta o relatório técnico (DOCX ou HTML) com identificação, referenciais, equipamentos, estatísticas GNSS/PPK, tabela de coordenadas, resumo da poligonal e declaração de responsabilidade |

Cada ferramenta tem botão próprio na barra, item no menu **Complementos →
Precisa Agrimensura** e um item **📖 Manual** que abre o `manual.docx` do módulo.
Pela Suíte (*📐 Topografia → Topografia*) aparece um menu rápido para escolher a ferramenta.

## 2. Caderneta de Estação Total

1. **📂 Selecionar** o arquivo `.B`, `.B00` … `.B09` (TransferCPE › Exportar ›
   Caderneta de Campo › Topograph) e clicar em **▶ Ler Arquivo**.
2. **Aba Arquivo:** confira estações, ré, data, leitura Hz à ré e nº de observações.
3. **Aba Estações:** para cada estação da lista:
   - **E, N, Z** da estação;
   - **HI** (vem do arquivo; **↺** restaura o valor original);
   - **Altura do prisma:** marque *Sobrescrever HP* para usar um único HP em
     todas as observações da estação;
   - **Orientação:** *por coordenadas da ré* (informe E/N da ré; o plugin
     calcula o azimute) **ou** *azimute à ré digitado* (G° M' S").
     A leitura do círculo Hz na ré (do arquivo) é descontada automaticamente.
   - **📥 Importar de Camada QGIS:** escolha uma camada de pontos e os campos
     Nome, E, N (e Z); o plugin casa estações e rés pelo nome.
   - **🗑 Limpar Configurações** zera tudo.
4. **⚙ Calcular Coordenadas.** Estações com problema (ex.: ré igual à
   estação) são listadas num aviso e as demais são calculadas.
5. **Aba Resultados:** tabela com estação, ponto, código, E, N, Z, Hz, ZN,
   distâncias e HP (ordenável por coluna).
6. Saídas:
   - **🗺 Criar Camada QGIS** — pergunta se as coordenadas são
     arbitrárias/locais (camada sem SRC) ou reais (você escolhe o SRC);
     cria a camada `Caderneta_ET` (PointZ).
   - **📄 Exportar CSV** — separador `;`, números com vírgula decimal, ângulos em GMS e decimal.
   - **📐 Exportar DXF** — DXF R12 com pontos 3D (`ET_PONTOS`), número
     (`ET_NUMEROS`) e código (`ET_CODIGOS`).

> Todas as estações começam com E=N=Z=0. Informe as coordenadas antes de
> calcular (com E/N da ré também em 0, a estação dá erro de “ré idêntica”).

## 3. Poligonal e Irradiação

### 3.1 Aba Poligonal
1. **Azimute inicial (1º lado)** em G° M' S"; **E/N inicial** do 1º vértice
   (o Z inicial não é usado no cálculo).
2. Tabela de **vértices**: nome, ângulo interno (G, M, S) e distância ao
   próximo vértice. **➕/➖** adicionam/removem linhas; **📥 Importar de CSV**
   aceita colunas `nome,angulo_g,angulo_m,angulo_s,dist`.
3. **⚙ Calcular** (com a aba Poligonal ativa). Aparecem: soma medida e
   teórica, erro e tolerância angular (10"√n), ΔE, ΔN, erro linear e precisão
   (verde = dentro; laranja = fora de 1:5000), e as coordenadas ajustadas.
4. **🗺 Camada Poligonal** (pontos + linha fechada) e **📄 CSV Poligonal**.

> ⚠ **Atenção — ver Descritivo, limitação 1.** Nesta versão o cálculo desloca
> os ângulos de uma linha: com dados sem erro, a poligonal não fecha. **Não use
> os resultados da poligonal sem conferência independente.** A opção
> “Aberta” do campo *Tipo* não muda o cálculo.

### 3.2 Aba Irradiação
1. **Ponto de apoio:** nome, E, N, Z, HI e **azimute à ré**.
2. Observações: nome, código, **Hz e ZN em graus decimais**, distância
   inclinada, HP e (opcional) HI específico. O plugin considera
   Az = Az_ré + Hz, ou seja, **o círculo deve ter sido zerado na ré**.
3. **⚙ Calcular** (com a aba Irradiação ativa) → **🗺 Camada Irradiação**
   (PointZ) e **📄 CSV Irradiação**.

### 3.3 Relatório
**📋 Relatório** abre a *Identificação do Relatório* (RT, CREA, ART,
proprietário, imóvel, município, data; **💾 Salvar como padrão** guarda para
a próxima vez) e gera DOCX (ou HTML sem python-docx) com os resultados da
poligonal e/ou da irradiação e a declaração de responsabilidade técnica.

## 4. Relatório de Levantamento

1. **Identificação:** RT, CREA, ART/RRT, proprietário, imóvel, município,
   matrícula/código INCRA, tipo de serviço e data.
2. **Levantamento:** SGR (padrão SIRGAS 2000), projeção, zona UTM (23S),
   modelo geoidal (hgeoHNOR2020), equipamentos, software, constelações e a
   **camada de pontos** (campos reconhecidos: nome/id, e/x, n/y, alt/z,
   qualidade/q, sd3d/sigma3d).
3. **Dados GNSS/PPK:** estatísticas do processamento (só entram se *Total de
   épocas* for preenchido) e resumo da poligonal (só entra se *Nº vértices*
   for preenchido).
4. **Observações:** texto livre (seção 7).
5. Escolha o **arquivo de saída** e clique em **📄 Gerar Relatório**.

Sem python-docx, o relatório sai em HTML resumido (só identificação e tabela de coordenadas).

## 5. Dependências
- Caderneta e cálculos: nenhuma além do QGIS.
- Relatórios DOCX: `python-docx` (OSGeo4W Shell: `pip install python-docx`).
