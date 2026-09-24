# Manual de Instruções — Precisa Geo Visualizador

**Pasta:** `geo_visualizador` · versão declarada 0.15

---

## 1. Para que serve

Gera um **site (página web com mapa)** do loteamento para ser aberto no celular
em campo ou enviado a clientes e corretores. A página mostra os lotes sobre
imagem de satélite, com a **posição GPS do celular** em tempo real. Três modos:

| Modo | Para quê | Produto |
|---|---|---|
| **Vetorial** | Lotes a partir de shapefile/GeoJSON, com rótulos, popup de medidas, rua e cotas opcionais | Produto 1 |
| **Imagem** | Planta (imagem) já georreferenciada no QGIS sobreposta ao satélite | Produto 1 |
| **Venda** | Lotes pintados por status (disponível/reservado/vendido) e valor, lidos de uma planilha Google publicada | Produto 2 / 3 |

O site pode ser **publicado direto no Netlify** (com API key) ou salvo como
**arquivo HTML** para você hospedar onde quiser.

## 2. Como abrir
Ícone **Geo Visualizador – Gerar Site** na barra de ferramentas, ou pelo menu
da Suíte Precisa (*🔧 Ferramentas → Geo Visualizador*). Abre a janela
**Projetos publicados**.

## 3. Janela “Projetos publicados”
- **Escolher/trocar pasta do histórico:** pasta onde fica o arquivo
  `geo_visualizador_links.csv` com todos os sites já publicados. Configure
  isso **antes** da primeira publicação — sem pasta, o link não é registrado.
- A tabela mostra **Título, Link (clicável), Status, Data e Produto**.
- **Atualizar status:** consulta o Netlify e muda de *pendente* para *pronto*
  quando o site termina de publicar; também troca links provisórios pelo
  endereço definitivo `https://nome.netlify.app`. (Precisa da API key salva.)
- **+ Novo site:** abre a tela de geração.

## 4. Gerar um novo site

Informe o **Título do loteamento** (aparece no topo do navegador, no rodapé do
site e vira o nome do endereço no Netlify) e escolha o **Modo**.

### 4.1 Modo Vetorial
1. **Camada de lotes** (polígonos).
2. Confira os **nomes dos campos**: lote (`LOTE`), quadra (`QUADRA`), área
   (`AREA_M2`), perímetro (`PERIMETRO`), frente/fundos/esquerda/direita
   (`DIM_FRENTE`, `DIM_FUNDO`, `DIM_ESQ`, `DIM_DIR`). Campos que não existirem
   são simplesmente omitidos no site.
3. **Camada de rua (opcional):** polígono da via, desenhado em cinza tracejado.
4. **Camada de cotas (opcional):** pontos com os campos `texto` (medida) e
   `rotacao` (graus) — o formato gerado pelo plugin de cotas da suíte.
5. **Continuar.**

No site: quadras “A.I.”/“Área Institucional” aparecem em cinza tracejado e
“A.V.”/“Área Verde” em verde tracejado; os demais lotes em laranja. Os rótulos
(lote, quadra, área, perímetro) aparecem sozinhos quando o zoom deixa espaço
suficiente; clicar no lote abre as medidas.

> Use a camada em **sistema projetado em metros (UTM)** — ver Descritivo,
> limitação 1.

### 4.2 Modo Imagem
1. Carregue antes a planta georreferenciada no QGIS (ex.: via Georreferenciador)
   como **imagem PNG/JPEG/TIFF comum**.
2. Escolha a **Planta georreferenciada**, a **Opacidade** (0 a 1, padrão 0,85)
   e o **Limiar de branco** (0–255, padrão 240 — pixels mais claros que isso
   ficam transparentes).
3. **Continuar.**
Precisa de `numpy` e `Pillow` (fornecidos pela Suíte Precisa).

### 4.3 Modo Venda (Produto 2 e 3)
Pré-requisito: uma planilha Google com Apps Script publicado (URL terminando em
`/exec`) que devolva `{ "ID_LOTE": {"status": "...", "valor": ...} }`.

1. **Camada de lotes.**
2. Se a camada ainda não tem `ID_LOTE`, `STATUS` e `VALOR`, clique em
   **Preparar camada**: cria uma **camada nova temporária** (`<nome>_venda`) com
   `ID_LOTE = QUADRA-LOTE`, `STATUS = disponivel` e `VALOR` vazio. O arquivo
   original não é alterado. **Salve essa camada** (botão direito → Exportar)
   se quiser mantê-la — ela é temporária.
   - O plugin recusa se houver QUADRA+LOTE repetido ou se os campos já existirem.
3. Confira os nomes dos campos, cole a **URL do Apps Script** e o **intervalo
   de consulta** (padrão 10 s).
4. Marque **Produto 3** se quiser as colunas administrativas extras no CSV
   (forma de pagamento, nº de parcelas, vencimento, adimplência — em branco).
5. **Continuar** → o plugin pede onde salvar o **CSV da planilha**
   (`id_lote, quadra, lote, status, valor` [+ 4 colunas]); importe esse CSV
   na planilha Google.
No site, cada lote é pintado: verde = disponível, amarelo = reservado,
vermelho = vendido; o valor aparece só para lotes disponíveis. A página
reconsulta a planilha sozinha no intervalo definido.

## 5. Publicar ou salvar
Abre a janela **Publicar Visualizador**:
- **API key do Netlify** (opcional) + **Salvar key neste computador**.
  **Remover key salva** apaga do computador.
- **Publicar no Netlify:** cria um **site novo** e registra o link no
  histórico como *pendente*. Depois, na lista de projetos, clique em
  **Atualizar status**.
- **Gerar HTML:** salva o `index.html` para upload manual.
- **Pasta do histórico de links:** o mesmo ajuste da janela principal.

## 6. Usando o site no celular
- Permita o acesso à localização. O indicador mostra *GPS OK* (< 10 m),
  *Melhorando sinal* (< 50 m) ou *Sinal fraco*.
- **🛰️ Satélite** liga/desliga o fundo; **📍 Minha posição** centraliza em você.
- O site precisa de internet (mapa, satélite e biblioteca vêm da web).

## 7. Cuidados
- Cada publicação cria um site novo. Publicar de novo com o **mesmo título**
  pode falhar (nome já usado no Netlify) — mude o título.
- Lotes que **não estiverem na planilha** aparecem como **disponíveis** no modo Venda.
- A janela congela alguns segundos durante a publicação — aguarde.
