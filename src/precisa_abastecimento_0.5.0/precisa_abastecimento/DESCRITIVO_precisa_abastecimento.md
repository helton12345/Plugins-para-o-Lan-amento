# Descritivo Técnico-Funcional — Precisa Abastecimento

**Pasta:** `precisa_abastecimento`
**Versão declarada:** 0.5.0 (experimental)
**QGIS mínimo:** 3.16

**Arquivos lidos:**
- `__init__.py`, `plugin.py`, `abastecimento_dialog.py`;
- `recalque_dialog.py`, `perfil_interativo_dialog.py`;
- todo o `core/`;
- `metadata.txt`.

**Compartilhado com o Precisa Esgoto (arquivos idênticos):**
- `segmentacao.py` e `docx_template.py`;
- `vendor/` (python-docx 1.2.0).

`terreno.py` é uma variante do da drenagem: usa `SourceBreakLines` em vez de `SourcePoints`.

---

## 1. Arquitetura
- **Núcleo em Python puro:**
  - `hidraulica.py`: Hazen-Williams e velocidade em seção plena;
  - `dimensionamento.py`;
  - `recalque.py`;
  - `topologia.py`: `orientar_arvore_de_fonte`, BFS a partir da fonte.
- **Com QGIS:** `terreno.py` e `segmentacao.py`.
- **Saídas:** `perfil_dxf.py` (perfil piezométrico), `planta_dxf.py`, `memoria_docx.py`.
- **Interface:** `abastecimento_dialog.py`, `recalque_dialog.py`, `perfil_interativo_dialog.py`.

## 2. Cálculo
1. **Árvore:**
   - BFS a partir do **nó de maior cota do terreno**;
   - arestas que fecham ciclo são descartadas e marcadas como “malha”;
   - trechos não alcançados ficam fora.
2. **Vazões:**
   - Q_total = P·q·k1·k2/86400 (L/s);
   - qm = Q_total/L_total;
   - Q do trecho = qm × comprimento da sub-árvore a jusante (incluindo o próprio trecho).
3. **DN:** menor DN da lista com V ≤ V_máx, ou o DN forçado na tabela.
4. **Perda de carga:** hf = 10,67·Q^1,852·L/(C^1,852·D^4,87).
5. **Pressões:**
   - piezométrica propagada da origem (cota de base + altura do reservatório);
   - pressão = piezométrica − cota do terreno;
   - status fora de [p_mín, p_máx].
6. **Alertas:**
   - V < V_mín;
   - V > V_máx;
   - J > 8 m/km (COPASA T-104);
   - pressão a jusante baixa ou alta.

**Recalque:**
- Q = P·q·K1/(h·3600);
- D = K·X^0,25·√Q, com X = h/24;
- DN comercial ≥ D; sucção um DN acima;
- AMT = Hg + hf (+10 % de perdas localizadas);
- P = 1000·Q·AMT/(75·η);
- folga do motor pela tabela de Azevedo Netto.

## 3. Normas e referências citadas no código
| Referência | Uso |
|---|---|
| **ABNT NBR 12218:2017** | Velocidades 0,6–3,0 m/s, pressão 10–50 mca, DN mín. (texto e padrões) |
| **COPASA T-104** | k1, k2, C, alerta J > 8 m/km |
| ABNT NBR 12214:2020; Tsutiya; Azevedo Netto | Recalque (Bresse, vazão, folga do motor) |
| NBR 12211:1992, Lei 11.445/2007, Lei 14.026/2020, Porto (2006) | Só citadas na memória |

## 4. Dependências
QGIS ≥ 3.16 (`qgis.analysis` para as curvas) e python-docx embarcado. Ver o
descritivo do esgoto sobre `typing_extensions`.

## 5. Limitações conhecidas e pontos de atenção
1. **A origem da rede não é o lote do reservatório.**
   - `orientar_arvore_de_fonte` é chamado sem `no_fonte`: a raiz é sempre o
     **nó de maior cota do terreno**.
   - A busca por lote e greide só troca a **cota de base**, que é aplicada a
     esse nó mais alto.
   - Se o reservatório estiver em outro ponto, o sentido do escoamento, as
     vazões, as perdas e as pressões são calculados a partir do lugar errado.
2. **Memória de cálculo com parâmetros diferentes dos calculados** (bug
   confirmado na leitura do código).
   - `exportar_memoria` monta um `ParametrosAgua` novo com `_ler_parametros()`,
     que **não define `piezo_origem`** (fica 0,0) e usa a **população do campo
     manual**.
   - Resultado: a memória imprime “Cota piezométrica na origem = 0,00 m”.
   - Quando a população veio dos lotes, Q_total, qm e o texto “atender a N
     habitantes” saem com o valor do campo (padrão 1000 hab), divergindo da
     tabela de trechos no mesmo documento.
3. **Recalque fora da memória.** A seção 9 existe no gerador, mas o diálogo
   nunca passa `recalque`. O resultado do recalque não é gravado.
4. **Malhas.**
   - As arestas de fechamento são **descartadas**: não recebem DN e não entram
     em L_total nem nas camadas.
   - Aparece um aviso de “rede desconexa ou com anéis”.
   - Não há Hardy-Cross.
5. **Pressão estática não verificada.**
   - Só a pressão dinâmica (com vazão máxima) é comparada a 10–50 mca.
   - Na NBR 12218:2017 a **pressão estática máxima é de 400 kPa (40 mca)**,
     segundo o conhecimento do revisor, **a confirmar na norma**. O código
     e a memória usam 50 mca e citam o “item 6.4”.
6. **DN escolhido só pela V_máx (3,0 m/s):**
   - tende ao menor DN possível, com perdas altas (o alerta de J > 8 m/km cobre em parte);
   - V_mín só gera alerta.
7. **C de Hazen-Williams:** o núcleo tem 130 como padrão; a tela abre com
   **140** e prevalece. A memória descreve o valor como “PVC/PEAD — COPASA T-104”.
8. **Camadas desatualizadas.**
   - Edição de DN e recálculo por lotes atualizam só a tabela.
   - Edição de geometria chama `calcular()`, que adiciona **novas camadas a
     cada edição** e descarta os DN forçados.
9. **Cotas:**
   - nó sem cota vira 0;
   - o raster é amostrado nas coordenadas da rede, sem conferir o SRC.
10. **Altura automática:** busca de 1 a 25 m. Se nenhuma serve, usa 25 m com
    aviso. Não considera a pressão estática.
11. **Recalque usa a população do campo manual**, não a dos lotes.
12. **Perfil DXF com exagero vertical de 2×**; o perfil interativo usa 10×.
13. **Planta DXF:** numera T1…Tn na ordem da lista (a tabela usa T{idx}).
14. **Sem ícone na barra;** `vendor/` no início do `sys.path` (ver esgoto).
