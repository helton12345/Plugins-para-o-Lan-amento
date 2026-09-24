# Manual de Instruções — Conversor de Memoriais Pro

**Pasta:** `gerador_de_memoriais` · versão declarada 1.4 · plugin gratuito (GPL-3.0)

---

## 1. Para que serve

Três ferramentas de memorial descritivo numa só janela:

| Aba | Entrada | Saída |
|---|---|---|
| **📝 Texto** | Memorial narrativo pronto (colado ou .txt/.docx/.pdf) | Camadas de **pontos e polígono** no QGIS (e shapefiles, opcional) |
| **🗺️ Camada QGIS** | Polígono desenhado no QGIS (em UTM) | **Memorial descritivo .docx** + shapefile de vértices numerados (opcional) |
| **📄 SIGEF** | PDF oficial do SIGEF/INCRA | **Memorial narrativo .docx** + camadas de pontos e polígono (opcional) |

Nenhum modo preenche confrontantes automaticamente: eles saem como
`[PREENCHER CONFRONTANTE]` para completar no Word.

## 2. Instalação das dependências
- Aba Texto com .txt: nenhuma.
- Aba Texto com .docx e abas Camada/SIGEF: `python-docx`.
- Aba Texto com .pdf e aba SIGEF: `pdfplumber`.

No OSGeo4W Shell (Windows): `python -m pip install python-docx pdfplumber`.
Reinicie o QGIS.

## 3. Como abrir
Menu **Complementos → Precisa Agrimensura → Gerador de Memoriais** (nome atual no menu; o novo nome, Conversor de Memoriais Pro, ainda não foi aplicado ao código), ou ícone
na barra de ferramentas.

## 4. Aba 📝 Texto — memorial → geometria

1. **Cole o texto** ou clique em **Carregar arquivo (.txt / .docx / .pdf)**.
2. Formatos reconhecidos:
   - UTM: `vértice P-01, de coordenadas N 7.671.998,640 m e E 562.498,640 m`
     (aceita também separadores no padrão inglês e texto entre o código e
     “de coordenadas”);
   - Geográfico: `vértice XXXX-P-0001 de coordenadas, Longitude -42°40'29,793" Latitude -20°59'39,563"`.
   Memoriais em forma de tabela **não** são reconhecidos.
3. **CRS:** padrão SIRGAS 2000 / UTM 23S (EPSG 31983).
   - Se o texto declarar o fuso (ex.: “Fuso 24S”, “fuso 23 sul”), o CRS é
     ajustado sozinho.
   - Se não declarar, o plugin pergunta se o CRS selecionado está correto —
     **confira o fuso antes de continuar**.
   - Texto em latitude/longitude → EPSG 4674 automaticamente.
4. **Salvar shapefiles em disco** (opcional): marque e escolha a pasta —
   serão gravados `memorial_vertices.shp` e `memorial_perimetro.shp`
   (substituem arquivos com o mesmo nome).
5. **Gerar pontos e polígono.** Surgem as camadas *Memorial - Vértices*
   (vértice, coordenadas, confrontante, azimute, distância) e
   *Memorial - Perímetro* (área, perímetro, nº de vértices).

Avisos úteis:
- “Vértices insuficientes” → menos de 3 vértices reconhecidos.
- “Vértices não reconhecidos: …” → códigos citados no texto sem coordenadas
  lidas (geralmente quebra de página/timbre no meio da frase). Corrija o
  trecho e gere de novo.
- Linhas repetidas idênticas (timbre, rodapé, “--- Página N ---”) são
  removidas automaticamente antes da leitura.

## 5. Aba 🗺️ Camada QGIS — geometria → memorial

1. **Camada de entrada:** escolha a camada de polígono (lista atualizada a
   cada abertura da janela). **Ela deve estar em UTM.** Em CRS geográfico o
   plugin avisa e pede confirmação (as medidas sairiam erradas).
2. **Shapefile de vértices (opcional):** informe o **prefixo** (ex.: `V-`) e o
   caminho do `.shp`. Os vértices recebem códigos `V-00001`, `V-00002`…
   (vértices repetidos entre feições são numerados uma só vez). Deixe o
   prefixo vazio para não gerar.
3. **Responsável técnico:** cidade, data (vazio = hoje), nome e registro
   profissional. Campos vazios saem como `[PREENCHER …]`.
4. **Pasta de saída** → **Selecionar…** (também sugere o caminho do shapefile).
5. **✔ Gerar Memorial (.docx)** → `Memorial_Georreferenciado_<data_hora>.docx`.

O memorial traz: cabeçalho com o responsável; campos a preencher (endereço,
cartório, proprietário); declaração do **método de cálculo** (UTM + fuso +
datum, lidos do CRS); para cada feição, área e perímetro e a descrição
vértice a vértice a partir do **vértice mais ao norte**, com azimute (grau,
minuto, segundo) e distância, ou trecho em **curva** (raio médio,
desenvolvimento, ângulo central) quando o desenho tem arco densamente
segmentado; fecho com cidade/data e assinatura.

## 6. Aba 📄 SIGEF — PDF do INCRA → memorial

1. **PDF SIGEF:** selecione o PDF oficial (com a tabela de vértices).
2. **Dados complementares (opcionais):** cidade e data para o fecho.
3. **Memorial (.docx):** onde salvar.
4. **Gerar também pontos e polígono** (marcado por padrão): cria camadas em
   SIRGAS 2000 geográfico (EPSG 4674) para cada parcela.
5. **Gerar Memorial Descritivo.** O log mostra quantas parcelas e vértices
   foram lidos.

O documento traz, por parcela: imóvel, proprietário, município, comarca
(cartório), matrícula, área e perímetro lidos do PDF; narrativa com longitude,
latitude, altitude, azimute e distância de cada vértice (e trechos em curva,
quando detectados); texto padrão sobre o SIRGAS 2000; assinaturas do
proprietário e do responsável técnico (nome, formação, CREA, código INCRA e ART
lidos do PDF).

## 7. Conferências obrigatórias antes de usar o documento
- Preencher todos os `[PREENCHER …]`.
- No modo Camada, os números saem com **ponto decimal e vírgula de milhar**
  (ex.: `N 7,671,998.640 m`, `12.35 m`) — ajuste para o padrão brasileiro se necessário.
- No modo SIGEF, o título é sempre “MEMORIAL DESCRITIVO – DIVISÃO DE IMÓVEL
  RURAL” — altere se não for divisão.
- No modo SIGEF, confira se **todos os vértices** do PDF entraram (ver
  Descritivo, limitação 1).
- Rodapé com propaganda da versão paga aparece em todos os .docx.
