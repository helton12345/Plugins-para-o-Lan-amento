# Manual de Instruções — Precisa Abastecimento

**Pasta:** `precisa_abastecimento` · versão declarada 0.5.0 (experimental)

---

## 1. Para que serve
Dimensiona a **rede de distribuição de água ramificada** de um loteamento:
- vazão de marcha;
- DN escolhido pela velocidade máxima;
- perda de carga por Hazen-Williams;
- pressão em cada nó.

Traz também uma calculadora de **linha de recalque** (Bresse).

Gera:
- camadas da rede e dos nós com pressão;
- tabela com DN editável;
- CSV;
- perfil piezométrico e planta em DXF;
- memória de cálculo em DOCX.

Acesso: **Complementos → Precisa Agrimensura → Precisa Abastecimento**
(botão sem ícone) ou, pela Suíte, **📁 Projetos → Precisa Abastecimento**.

## 2. Tela principal (de cima para baixo)

### 2.1 Dados do Projeto
Empreendimento, proprietário, município/UF, data, RT, título, registro e
ART. Vão para a memória de cálculo.

### 2.2 Entradas
- **Rede de distribuição (linha):** SRC projetado.
- **Fonte do terreno:**
  - **MDT (raster)**; ou
  - **Curvas de nível** com o campo de cota (TIN com linhas de quebra).

### 2.3 Segmentação automática
Ligada por padrão.
- Espaçamento máximo: 60 m.
- Trecho mínimo: 12 m.
- Tolerância de cruzamento: 0,5 m.

### 2.4 População pelos lotes (opcional)
Soma os habitantes dos lotes vendáveis:
- hab/lote × unidades;
- lotes de quadras com prefixo `A.I, AI, A.V, AV` ficam de fora.

Tem **prioridade** sobre o campo “População (hab)”.

### 2.5 Reservatório: base pelo greide da rua (opcional)
- **Lote do reservatório:** `QUADRA/NUM_LOTE` (ex.: `A.I./1`).
  - Escolha também o **campo do número do lote**.
- **Greide da rua** (linha) e o campo de cota de projeto.
- Ordem de busca da cota de base:
  1. greide mais próximo do lote;
  2. se não houver greide, **terreno** no ponto da rede mais próximo do lote
     (estimativa, com aviso);
  3. por último, a cota do **nó mais alto da rede**.
- **Altura automática:** procura a menor altura (1 a 25 m, passo 0,1 m) que
  deixa todos os nós entre a pressão mínima e a máxima.

> ⚠ A **rede sempre parte do nó de maior cota do terreno**. O lote do
> reservatório só muda a **cota** da água, não o **ponto** de onde a rede
> sai (ver Descritivo).

### 2.6 Demanda e Hidráulica
| Parâmetro | Padrão |
|---|---|
| População | 1000 hab |
| Consumo | 150 L/hab·dia |
| k1 | 1,20 |
| k2 | 1,50 |
| Reservatório acima do terreno | 15 m |
| C de Hazen-Williams | 140 |
| V mín | 0,60 m/s |
| V máx | 3,00 m/s |
| Pressão mínima | 10 mca |
| Pressão máxima | 50 mca |

Diâmetros: DN 50 a 400.

## 3. Calcular e revisar
1. Clique em **Calcular**. A barra mostra:
   - Q total;
   - o nó de origem;
   - a cota de base e a piezométrica.
   Avisa se há anel/malha e se há nós sem cota.
2. Surgem as camadas `agua_rede` e `agua_nos_pressao`.
   - Na rede: vermelho = V > máx; amarelo = V < mín.
   - Nos nós: vermelho = pressão fora dos limites.
3. **Tabela:** Trecho, DN, Q, V, L, hf, pressão a jusante, alertas.
   - **DN(mm)** é editável e recalcula tudo a jusante.
4. **Ver Perfil (interativo):** terreno × piezométrica × pressão.
   Só visualização.
5. **Recálculo automático:**
   - editar os lotes recalcula a tabela;
   - editar a geometria da rede refaz tudo (**e cria novas camadas**).

> Edições na tabela e recálculos por lotes **não atualizam as camadas do
> mapa**. Clique em **Calcular** de novo. Isso descarta os DN editados.

## 4. Recalque (Bresse)…
Janela independente:
- cotas de sucção e descarga;
- vazão, ou **Calcular Q** = P·q·K1/(h·3600);
- comprimentos da adutora e da sucção;
- horas de bombeamento;
- K de Bresse, C, perdas localizadas, rendimento e velocidades máximas.

Resultado em texto: DN de recalque e de sucção, AMT, potência da bomba e do motor.
**Não é gravado nem entra na memória de cálculo.**

## 5. Exportar
- **CSV** e **Planta (DXF)**.
- **Perfil (DXF)**, com exagero vertical de 2×.
- **Memória (DOCX)**. Leia antes de assinar:
  - a memória imprime **cota piezométrica na origem = 0,00 m**;
  - se a população veio dos lotes, a memória usa a **população do campo
    manual** (ver Descritivo).
