# Manual de Instruções — Precisa Sedimentação

**Pasta:** `precisa_sedimentacao` · versão declarada 1.2

---

## 1. Para que serve

Compara **dois modelos do terreno (MDTs)** da mesma área — um de referência
(“base”) e um atual (“comparação”) — dentro de uma poligonal, e calcula o
volume de material acumulado ou disponível. Gera mapa de diferença, planilha
Excel e laudo DOCX. Três modos:

| Modo | Uso | MDT base | MDT comparação | Resultado principal |
|---|---|---|---|---|
| **Forma A — Assoreamento** | Reservatório de hidrelétrica/lagoa | Levantamento anterior ou projeto | Batimetria atual | Volume sedimentado, % de perda de capacidade, taxa m³/ano |
| **Forma B — Barragem de rejeito** | Monitoramento de barragem de mineração | Terreno natural/anterior | Levantamento atual | Volume de rejeito, resguardo operacional, borda livre de projeto, lâmina d'água residual |
| **Forma C — Pacote sedimentar** | Cubagem de areia/cascalho | Rocha-mãe (*bedrock*) | Batimetria atual | Volume extraível (só espessuras positivas) |

## 2. Antes de usar
- Bibliotecas: `numpy`, `openpyxl`, `python-docx`, `matplotlib` (GDAL já vem com o QGIS).
- **MDTs e poligonal no mesmo sistema de coordenadas projetado, em metros.**
  Se forem diferentes, o processamento falha — reprojete antes no QGIS.
- Carregue no projeto os rasters prontos **ou** as curvas de nível/pontos cotados
  de cada levantamento, a poligonal e, se quiser, uma linha de perfil.

## 3. Como abrir
Menu **Complementos → Precisa Agrimensura → Sedimentação**, ou pelo launcher
da Suíte (*📁 Projetos → Sedimentação*). A janela não bloqueia o QGIS.

## 4. Passo a passo

### Modo de cálculo
Escolha Forma A, B ou C no topo — os campos específicos aparecem na aba 3.

### Aba 1 · MDT Base e Aba 2 · MDT Comparação
Para cada levantamento:
- **Raster MDT já pronto** (escolha na lista), **ou**
- **Curvas de nível + campo de cota** e/ou **pontos cotados + campo de cota**
  (o plugin gera o MDT por TIN).
- **Data do levantamento** (dd/mm/aaaa) — vai para o laudo e para a série temporal.

### Aba 3 · Poligonal e Parâmetros
- **Poligonal de referência** (obrigatória): reservatório, bacia de contenção ou
  limite da licença.
- **Resolução do raster** (padrão 1,0 m) — usada só quando o MDT é gerado por TIN.
- **Formas A e B — Período entre levantamentos (anos)**: opcional; calcula a
  taxa média em m³/ano.
- **Forma A:**
  - **Capacidade original V₀** (opcional) → % de perda de capacidade.
  - **Gerar curva Cota × Área × Volume comparativa** + **passo** (padrão 0,25 m).
  - **Cotas NMN e NMO** (opcionais) → volume útil e volume morto da curva base.
- **Forma B:**
  - **Cota da crista** e **cota da lâmina d'água atual** (obrigatórias).
  - **Cota do nível máximo maximorum** (opcional, vem do estudo hidrológico)
    → borda livre de projeto.
  - **Calcular lâmina d'água residual** (marcado): área e volume de água acima
    do rejeito na cota da lâmina.
- **Perfil longitudinal** (opcional, todas as formas): escolha uma camada de
  linha — é usada a primeira linha da camada.

### Aba 4 · Série Temporal (Formas A e B)
Marque **Habilitar**, adicione as campanhas anteriores (**data** e **volume
acumulado em m³**) com **+ Adicionar campanha**. O resultado desta execução
entra como a campanha mais recente. Com 2+ campanhas: taxa por regressão
linear e, se informar **capacidade/volume-limite** (ou V₀ na Forma A),
projeção de vida útil em anos.

### Aba 5 · Identificação
Proprietário/contratante, empreendimento, município, responsável técnico,
título, CREA e ART (vão para o laudo).

### Aba 6 · Saída
Pasta, prefixo (padrão `sedimentacao`), Excel, DOCX e **Adicionar raster de
diferença ao mapa**.

### Executar
**▶ Calcular e Gerar Laudo.** Acompanhe o log. Ao final, a janela lista os
arquivos gerados.

## 5. Arquivos gerados
| Arquivo | Conteúdo |
|---|---|
| `<prefixo>_base_mdt.tif`, `<prefixo>_comparacao_mdt.tif` | MDTs gerados por TIN (se for o caso) |
| `<prefixo>_comparacao_reamostrado.tif` | MDT atual alinhado à grade do base (se as grades diferirem) |
| `<prefixo>_diferenca.tif` | Mapa de ΔZ = atual − base (isopacas) |
| `<prefixo>_sedimentacao.xlsx` | Resumo; abas “Curva CAV” e “Série Temporal” quando habilitadas |
| `<prefixo>_laudo.docx` | Relatório técnico |
| `<prefixo>_laudo_*.png` | Figuras do laudo (isopacas, CAV, perfil, série) |

## 6. Leitura dos resultados
- **ΔZ positivo** = a superfície atual está mais alta (sedimento/rejeito
  depositado ou espessura acima do *bedrock*).
- Na Forma A e B o volume é a **soma líquida** (áreas com erosão descontam).
- Na Forma C só entram as células com ΔZ > 0.
- **Resguardo operacional** (crista − lâmina do dia) **não é** a borda livre
  regulatória; a borda livre de projeto só é calculada com a cota do nível
  máximo maximorum.

## 7. Problemas comuns
| Situação | O que fazer |
|---|---|
| Erro de compatibilidade de SRC | Reprojete MDTs e poligonal para o mesmo sistema |
| Forma B pede crista e lâmina | Preencha as duas cotas na aba 3 |
| Campanha inválida | Preencha data e volume na tabela, ou remova a linha |
| Nenhuma célula válida na poligonal | Poligonal fora dos MDTs ou em outro sistema |
