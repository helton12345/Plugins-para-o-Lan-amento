# Manual de Instruções — Precisa Batimetria CAV

**Pasta:** `precisa_batimetria` · versão declarada 1.0

---

## 1. Para que serve

A partir das **curvas de nível e/ou pontos cotados do fundo** de uma lagoa ou
reservatório e do **polígono do espelho d'água**, o plugin:

1. gera o **MDT do fundo** por interpolação TIN (GeoTIFF);
2. calcula a **Curva Cota-Área-Volume (CAV)** em faixas de cota;
3. ajusta **equações de 2º grau** de área e volume (com R²);
4. desenha o **gráfico CAV**, o **perfil longitudinal** e **3 seções transversais**;
5. gera **curvas de nível** a partir do MDT;
6. monta a **planilha Excel** e o **laudo técnico DOCX** (com relatório fotográfico).

## 2. Antes de usar
- Bibliotecas Python necessárias no QGIS: `numpy`, `openpyxl`, `python-docx`
  e `matplotlib` (sem o matplotlib, os gráficos em PNG são pulados).
- Todas as camadas devem estar **no mesmo sistema de coordenadas projetado,
  em metros** (ex.: SIRGAS 2000 / UTM).
- Carregue no projeto: a camada de **curvas** (linhas) com campo de cota e/ou
  a de **pontos cotados**, e a camada de **polígono do nível d'água (NA)**.

## 3. Como abrir
Menu **Complementos → Precisa Agrimensura → Batimetria CAV**, ou pelo launcher
da Suíte Precisa (*📁 Projetos → Batimetria CAV*).

## 4. Passo a passo

### Aba 1 · Dados Espaciais
| Campo | O que informar |
|---|---|
| **Curvas de Nível – Camada / Campo de cota** | Camada de linhas e o campo com a cota |
| **Pontos Cotados – Camada / Campo de cota** | Opcional; melhora o TIN. Pode ser usado sozinho, sem curvas |
| **Polígono do NA – Camada** | Obrigatório. Define a área de cálculo (o MDT é recortado por ele) |
| **Cota do NA (m)** | Cota da lâmina d'água (topo da curva CAV) |
| **Cota mínima do fundo (m)** | Cota inicial da curva CAV — informe o fundo real da lagoa |
| **Resolução do raster (m/pixel)** | Padrão 1,0 m (0,5–1,0 m recomendado para lagoas médias) |
| **Número de intervalos de cota** | Padrão 10 (de 5 a 200). A tabela terá esse número + 1 linhas |
| **Adicionar MDT raster ao mapa** | Carrega o MDT no projeto ao final |
| **Gerar curvas de nível a partir do MDT / Intervalo** | Padrão 0,5 m; o shapefile é carregado no projeto |

> Os campos de cota do NA e do fundo vêm preenchidos com valores de exemplo
> (522,908 m e 517,891 m) — **substitua sempre** pelos do seu levantamento.

### Aba 2 · Dados do Projeto
Proprietário, fazenda/imóvel, município, área do imóvel (ha), latitude e
longitude do centroide (texto livre), data do levantamento, responsável técnico,
título profissional, CREA e ART. Esses dados vão para o cabeçalho, textos e
assinatura do laudo e para o título da planilha.

### Aba 3 · Saída
- **Pasta** e **Prefixo** dos arquivos (padrão `batimetria`).
- Marque o que gerar: **Planilha Excel**, **Laudo DOCX**, **Perfil e seções**.
- A opção “Exportar raster MDT (GeoTIFF)” **não tem efeito**: o MDT é sempre
  salvo, porque é a base de cálculo.

### Aba 4 · Fotos 📷
**➕ Adicionar fotos** (JPG, PNG, BMP, TIFF). A legenda inicial é o nome do
arquivo — **clique duplo** para editar. **▲ / ▼** mudam a ordem; **🗑 Remover**
tira a foto. No laudo, as fotos saem 2 por linha.

### Executar
Clique em **▶ Calcular e Gerar Laudo**. O log e a barra de progresso mostram
cada etapa. Ao final, uma janela lista todos os arquivos gerados.

## 5. Arquivos gerados (na pasta escolhida)
| Arquivo | Conteúdo |
|---|---|
| `<prefixo>_mdt.tif` | MDT do fundo (TIN) |
| `<prefixo>_curva_cav.png` | Gráfico CAV (área à esquerda; volume à direita, eixo invertido) |
| `<prefixo>_perfil_longitudinal.png` | Perfil do fundo com a linha do NA |
| `<prefixo>_secao_inicio/meio/fim.png` | Seções a 25 %, 50 % e 75 % do comprimento |
| `<prefixo>_cav.xlsx` | Tabela CAV, equações, gráfico nativo do Excel e aba com o PNG |
| `<prefixo>_laudo.docx` | Laudo técnico completo |
| `<prefixo>_curvas_mdt.shp` | Curvas de nível do MDT (campo `ELEV`) |

## 6. Revisar o laudo antes de assinar
O laudo DOCX traz **textos padrão fixos** que precisam ser conferidos e
ajustados ao seu caso:
- Metodologia descrita como **corda graduada com peso + GNSS RTK** e
  processamento no **GTopo** — altere se usou outro método (ecobatímetro etc.).
- Objeto descrito como **“lagoa artificial”** em imóvel rural de uso agropecuário.
- A seção **8. Anexos** só lista os anexos (Mapa, ART, planilha de campo) —
  eles **não são gerados**; anexe-os manualmente.
- As cotas aparecem como “Datum SIRGAS 2000” — informe o referencial
  altimétrico correto, se for o caso.
- A **profundidade máxima** é calculada como *Cota do NA − Cota mínima
  informada* (não é lida do MDT).

## 7. Problemas comuns
| Situação | Causa provável |
|---|---|
| “Selecione ao menos a camada de curvas OU de pontos” | Nenhuma camada de cota escolhida |
| Erro na regressão / ajuste | Menos de 3 faixas com área > 0 (cota mínima acima do fundo real ou polígono fora do MDT) |
| Áreas zeradas | Polígono do NA em outro sistema de coordenadas que as curvas |
| Erro ao abrir sem a Suíte Precisa instalada | Ver Descritivo, limitação 1 |
