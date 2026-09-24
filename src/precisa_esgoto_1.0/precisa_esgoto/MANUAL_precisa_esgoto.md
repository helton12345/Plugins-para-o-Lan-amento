# Manual de Instruções — Precisa Esgoto

**Pasta:** `precisa_esgoto` · versão declarada 1.0 (experimental)

---

## 1. Para que serve
Dimensiona uma rede coletora de esgoto por gravidade (loteamentos),
trecho a trecho, pelos critérios da NBR 9649 e da COPASA T-194, a partir de:
- uma **camada de linhas** com o traçado da rede;
- um **MDT** (raster de cotas).

Gera:
- camadas com a rede dimensionada e os PVs;
- tabela de resultados editável;
- CSV;
- perfil e planta em DXF;
- perfil interativo;
- memória de cálculo em DOCX.

Acesso: **Complementos → Precisa Agrimensura → Precisa Esgoto** (o botão da
barra não tem ícone) ou, pela Suíte, **📁 Projetos → Precisa Esgoto**.

## 2. Antes de começar
- A rede e o MDT têm de estar **no mesmo SRC projetado** (em metros).
  - O plugin recusa rede em SRC geográfico.
  - O plugin **não reprojeta** o MDT.
- O sentido do escoamento é decidido pelas **cotas do terreno** em cada trecho
  (montante = ponta mais alta).
  - Em ruas planas, confira o sentido e use **Inverter sentido**.

## 3. Tela principal (de cima para baixo)

### 3.1 Dados do Projeto (vão para a memória de cálculo)
- Empreendimento, proprietário, município/UF, data.
- RT, título, registro e ART. Já vêm com os dados do escritório.

### 3.2 Entradas
- **Rede (linha)** e **MDT terreno (raster)**.
- **Campo população/trecho (manual):** campo numérico da camada de rede.
  - Com a segmentação ligada, o valor da feição é repartido pelos
    sub-trechos proporcionalmente ao comprimento.

### 3.3 População pelos lotes (opcional)
Se preenchido, **tem prioridade** sobre o campo manual.
- **Lotes (polígono)** e **campo quadra**.
  - Lotes cuja quadra começa com `A.I, AI, A.V, AV` (editável) não contam.
- **Unidades/lote:** um campo da camada ou um valor único.
- **Habitantes por lote** (padrão 4).
- **Distância máxima lote→trecho** (padrão 250 m).
- Cada lote soma sua população ao trecho mais próximo do seu interior.

### 3.4 Segmentação automática (ligada por padrão)
- Cria um nó em cada cruzamento, em cada mudança de direção e por espaçamento.
- **Espaçamento máx. entre PV:** padrão 60 m.
- **Trecho mínimo:** padrão 12 m.
- **Tolerância de cruzamento:** padrão 0,5 m.
- Desligada, cada feição da camada vira **um** trecho, do 1º ao último ponto.

### 3.5 Parâmetros

| Parâmetro | Padrão |
|---|---|
| Consumo | 150 L/hab·dia |
| C (coeficiente de retorno) | 0,80 |
| k1 | 1,20 |
| k2 | 1,50 |
| n de Manning | 0,013 |
| Q mínima | 1,5 L/s |
| Tensão trativa mínima | 1,0 Pa |
| Recobrimento | 0,90 m |

- A infiltração é automática: menor valor entre 0,00033 L/s/m e 25 % da vazão média.
- Diâmetros fixos: DN 200 a 600.

## 4. Calcular e revisar
1. Clique em **Calcular**. Surgem as camadas `rede_esgoto_dim` e `PVs_esgoto`.
   - Cores das galerias:
     - vermelho: V > 5 m/s;
     - amarelo: τ abaixo do mínimo;
     - laranja tracejado: lâmina no limite.
   - PV com **X vermelho** = fundo acima do terreno.
   - A mensagem avisa se algum nó ficou **sem cota** no MDT. Esse nó é
     tratado como cota 0, e o resultado é inválido.
2. **Tabela:** Trecho, DN, S, Q, V, y/D, σt, profundidades, degrau no PV, alertas.
   - Dá para editar **DN(mm)** (só diâmetros da lista) e **Prof.mont**. O
     trecho e tudo a jusante são recalculados na hora; a célula fica amarela.
3. **Inverter sentido do trecho selecionado:** força o sentido oposto.
4. **Ver Perfil (interativo):** escolha o alinhamento e arraste um PV azul
   para cima ou para baixo para mudar a profundidade.
5. **Recálculo automático:**
   - editar a população (rede ou lotes) recalcula a tabela;
   - mover ou inserir vértice na rede refaz tudo (**e cria novas camadas no projeto**).

> As edições da tabela, do perfil, da inversão e da população **não
> atualizam as camadas do mapa**, só a tabela e as exportações. Clique em
> **Calcular** de novo para atualizar o mapa. Isso descarta as edições manuais.

## 5. Exportar
- **Exportar CSV…** (`;`, ponto decimal).
- **Exportar Perfil (DXF)…:** um perfil por caminho, lado a lado, exagero vertical 10×.
- **Exportar Planta (DXF)…:**
  - linhas por DN, PVs com profundidade;
  - quadro de trechos (até 60 linhas).
- **Exportar Memória (DOCX)…:** memória de cálculo completa, com
  normas, parâmetros, tabela de trechos, quadro de PVs, síntese e conclusões.
  - Os parâmetros são lidos da tela **no momento da exportação**. Não altere
    nada entre o cálculo e a exportação.

## 6. Conferência obrigatória
A síntese e as conclusões da memória **sempre** dizem “ATENDE” / “todos os
trechos atenderam”. Confira na tabela de trechos e nos alertas antes de assinar.
Ver o Descritivo, seção 6.
