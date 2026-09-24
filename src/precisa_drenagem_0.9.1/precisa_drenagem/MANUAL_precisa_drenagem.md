# Manual de Instruções — Precisa Drenagem

**Pasta:** `precisa_drenagem` · versão declarada 0.9.1 (experimental)

---

## 1. Para que serve
Dimensiona a rede de **galerias de águas pluviais** de um loteamento,
trecho a trecho. A vazão vem do **Método Racional** com chuva IDF, e as galerias
são verificadas por Manning.

Também preenche as áreas de contribuição de cada trecho, por delineação de
bacias (MDT) ou por lotes.

Gera:
- camadas de galerias e PVs;
- CSV;
- perfil e planta em DXF;
- memória de cálculo em DOCX.

Acesso: **Complementos → Precisa Agrimensura → Precisa Drenagem** (botão sem
ícone) ou, pela Suíte, **📁 Projetos → Precisa Drenagem**.

## 2. Preparação da camada de rede
- Linhas em **SRC projetado** (metros).
- **Cada feição = um trecho entre dois PVs.** O plugin **não** quebra as
  linhas em cruzamentos nem por espaçamento.
  - Divida a rede antes, com um nó em cada PV.
  - As pontas devem coincidir (tolerância de 1 cm).
- Um campo numérico para a **área de contribuição (ha)** de cada trecho e,
  opcionalmente, um campo de **C**.
- **Todo trecho precisa de área > 0.** Trecho de cabeceira sem área
  gera declividade de 50 % e vala enorme (ver Descritivo).

## 3. Individualizar Sub-Bacias (opcional, preenche o campo de área)

### Método A — MDT + pontos de deságue (Whitebox)
1. Escolha o **MDT** (arquivo raster) e a camada de **pontos** (bocas de lobo),
   **no mesmo SRC**. Opcionalmente, o campo de ID dos pontos.
2. **Delinear Sub-Bacias →:**
   - gera a camada `bacias_contribuicao`;
   - **grava o campo `area_bacia_ha` na camada de pontos**, que é editada e salva.
3. Na seção Entradas, escolha a rede e o campo de área. Depois clique em
   **Vincular Áreas à Rede →**.
   - Cada trecho recebe a área do ponto a até **1 m** de uma de suas pontas.
   - **A camada de rede é editada e salva.**

> ⚠ Cada bacia é delineada **sozinha**, e inclui toda a área a montante do
> ponto. Somada à acumulação de vazão, isso pode **contar a mesma área
> várias vezes**. Revise antes de usar (Descritivo, item 3).

### Método B — Lotes (divisas + caimento)
1. Escolha a **camada de lotes** e, se quiser, um campo de área.
   - Sem esse campo, a área é calculada pela geometria (m² → ha).
2. **Distância máx. lote→rede** (padrão 15 m) e **desnível máx. para aterro**
   (padrão 0 = desligado).
3. Na seção Entradas, preencha a rede, o campo de área e a **fonte do terreno**.
4. Clique em **Individualizar por Lotes →**.
   - Para cada lote, a “frente” é o lado mais próximo da rede.
   - Se o lote drenar para a frente (ou até o limite de aterro), sua área
     soma no trecho mais próximo da frente.
   - O campo de área da rede é **sobrescrito e salvo**.
   - Surge a camada de auditoria `drenagem_saida_lotes`. **Confira antes de calcular.**

## 4. Entradas
- **Rede de galerias**.
- **Fonte do terreno:**
  - **Raster (MDT)**; ou
  - **Curvas de nível** com o campo de cota (interpolação TIN).
- **Campo área contrib. (ha)** (obrigatório) e **Campo coef. C** (opcional;
  sem ele, vale o C global).

## 5. Parâmetros

### Chuva (IDF)
- Fórmula: i = K·T^a/(t+b)^c.
- Padrões: K = 3260; a = 0,16; b = 26; c = 0,90; TR = 10 anos; tc = 15 min.
- **Troque K, a, b e c pelos da sua localidade** (Pluvio/UFV).

### Hidráulica
| Parâmetro | Padrão |
|---|---|
| C global | 0,70 |
| n de Manning | 0,013 |
| V mín | 0,75 m/s |
| V máx | 5,0 m/s |
| Lâmina máx | 0,80 |
| Recobrimento | **1,00 m** |

- Diâmetros: DN 400, 500, 600, 800, 1000, 1200 e 1500.
- Profundidade máxima: 3,00 m (fixa).

## 6. Calcular
1. Clique em **Calcular**.
   - A barra mostra a intensidade *i* e o número de galerias e PVs.
   - Avisa se algum nó ficou **sem cota** (vira 0).
2. Surgem as camadas `drenagem_galerias` e `drenagem_PVs`, e a tabela:
   Galeria, DN, S, Q, V, y/D, Área, C, Prof.mont, Alerta.
   - A tabela **não é editável** nesta versão.
3. Alertas possíveis:
   - V > 5 m/s (prever dissipador);
   - trecho > 80 m;
   - lâmina no limite;
   - vala > 3 m.

## 7. Exportar
- **CSV**, **Perfil (DXF)** e **Planta (DXF)**.
- **Memória (DOCX)**. Leia antes de assinar:
  - sai com empreendimento **em branco** e município **“Guiricema-MG”**;
  - o RT é fixo;
  - a coluna “Q própria” está errada (ver Descritivo).
- Os parâmetros da memória são lidos da tela no momento da exportação.
