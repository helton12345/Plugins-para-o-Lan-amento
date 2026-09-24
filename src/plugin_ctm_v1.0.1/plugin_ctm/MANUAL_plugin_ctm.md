# Manual de Instruções — CTM/BCI (Cadastro Técnico Multifinalitário)

**Pasta:** `plugin_ctm` · versão declarada 1.0.1 (experimental) · QGIS 3.34+

---

## 1. Para que serve

Ferramenta de cadastro urbano para prefeituras e escritórios:
1. **Valida a topologia** da camada de lotes: sobreposições, vazios e
   geometrias inválidas.
2. **Calcula, para cada lote,** a inscrição imobiliária, área, perímetro,
   testada principal e secundárias, profundidade média e confrontantes.
3. **Emite o Boletim de Cadastro Imobiliário (BCI)** em HTML ou PDF, um por
   lote ou todos num só arquivo.

## 2. Preparação dos dados
- **Lotes:** polígonos com colunas de **distrito, setor, quadra e número do lote**.
- **Logradouros:** **linhas** (eixo ou meio-fio) com coluna de **nome da rua**.
- **Limite do setor/quadra:** polígono, só para a verificação de vazios.
- Tudo em **sistema projetado em metros** (UTM). O plugin não confere isso.
- Colunas opcionais nos lotes para o boletim completo: área construída, ano
  de construção, tipo de edificação, estado de conservação, estrutura,
  cobertura, pavimentação e tipo, redes de água/esgoto/elétrica, iluminação,
  coleta de lixo, meio-fio, tipo de domínio, matrícula no RI, zona fiscal,
  situação REURB (valores Sim/Não viram ✔/✘ no boletim).

## 3. Como abrir
Botão **Abrir CTM/BCI** (barra de ferramentas “CTM/BCI”), menu
**Complementos → CTM/BCI - Cadastro Técnico Multifinalitário**, ou menu da
Suíte (*📋 Peças Técnicas → CTM/BCI*).

## 4. Aba Topologia
1. **Camada de Lotes** e, para vazios, **Limite do Setor/Quadra**.
2. Botões (rodam em segundo plano; cada resultado vira uma camada nova no projeto):
   - **Verificar Overlaps** → camada *Overlaps*: área de cada sobreposição
     entre dois lotes (`lote_id_1`, `lote_id_2`, `area_sobreposta_m2`).
   - **Verificar Gaps** → camada *Gaps*: vazios dentro do limite não cobertos
     por lotes (ignora fragmentos < 0,01 m²).
   - **Verificar Geometrias Inválidas** → camada de pontos
     *Geometrias_Invalidas* com o motivo (autointerseção, anel aberto etc.).
   A identificação do lote usa a coluna `inscricao`, `lote_id`, `lote`,
   `id_lote` ou `numero`, se existir; senão, o ID interno.

## 5. Aba BCI
1. **Camadas:** Lotes, Quadras (lista obrigatória, mas não usada no cálculo)
   e Logradouros.
2. **Tolerância de Contato** (padrão 0,30 m): distância máxima entre a divisa
   do lote e a linha da rua para contar como testada. Rua desenhada no
   meio-fio pede ~1,50 m; rua na divisa, valores menores.
3. **Tolerância de Vizinhança** (padrão 0,02 m): distância máxima entre dois
   lotes para serem confrontantes.
4. **Mapeamento de Campos:** escolha a coluna real de Distrito, Setor, Quadra,
   Número do Lote e Nome do Logradouro.
5. **Configurações de inscrição:** nº de dígitos (zeros à esquerda) de
   distrito (2), setor (2), quadra (3) e lote (4). “Lote 01” com 4 dígitos vira
   `0001`. Máscara: `distrito.setor.quadra.lote`.
6. **Ignorar confrontantes:** palavras separadas por vírgula (ex.:
   `servidão, via, viela`); lotes cujo número contenha essas palavras não
   aparecem como vizinhos.
7. **Calcular BCI.**

> **Atenção (limitação atual):** o botão só calcula se **todos** os campos
> estiverem mapeados, inclusive os da aba *Atributos Complementares*. Se algum
> ficar em “— nenhum —”, aparece a lista de campos faltando. Ver Descritivo,
> limitação 1.

## 6. Aba Atributos Complementares
Mapeie as colunas de edificação (Bloco 4), infraestrutura (Bloco 5) e
situação jurídica (Bloco 6).

## 7. Aba Revisão / Exportação
1. Ao terminar o cálculo, a janela abre a aba **Atributos Complementares** —
   clique na aba **Revisão / Exportação**.
2. A tabela mostra inscrição, logradouro principal, testada, área, perímetro,
   profundidade média e confrontantes **Esquerda / Direita / Fundo**
   (conferência — não vão para o boletim).
3. **Contribuinte/possuidor:** selecione o lote na tabela, volte à aba BCI,
   preencha Nome, CPF/CNPJ e Endereço e clique **Aplicar ao Lote Selecionado**.
4. **Cabeçalho do boletim:** Município, Secretaria e brasão (PNG/JPG).
5. Exportar:
   - **HTML** ou **PDF (Lote Selecionado)**;
   - **Todos — PDF Único** (um lote por página);
   - **Todos — PDFs por Lote** (`BCI_<inscrição>.pdf` numa pasta).

## 8. Conteúdo do boletim
Cabeçalho (município, secretaria, brasão) · **Bloco 1** Identificação
(inscrição, logradouro principal, esquina) · **Bloco 2** Contribuinte ·
**Bloco 3** Dados físicos (área, perímetro, testada, profundidade) · Testadas
secundárias (se esquina) · **Bloco 4** Edificação · **Bloco 5**
Infraestrutura · **Bloco 6** Situação jurídica · assinatura do responsável
técnico · rodapé citando a Portaria MDR 3.242/2022.

## 9. Dicas
- Confira testadas em lotes de esquina e com ruas curvas.
- Um valor vazio de distrito/setor/quadra vira `?` na inscrição — corrija a
  tabela antes de exportar PDFs por lote (o `?` não é aceito em nomes de
  arquivo no Windows).
