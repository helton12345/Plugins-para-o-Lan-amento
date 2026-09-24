# Descritivo Técnico-Funcional — Precisa Esgoto

**Pasta:** `precisa_esgoto`
**Versão declarada:** 1.0 (experimental)
**QGIS mínimo:** 3.16

**Arquivos lidos:**
- `__init__.py`, `plugin.py`, `esgoto_dialog.py`, `perfil_interativo_dialog.py`;
- todo o `core/`;
- `metadata.txt`.

`vendor/` contém o **python-docx 1.2.0** de terceiros, sem alterações
próprias. Foi verificado só quanto à versão e às dependências.

---

## 1. Arquitetura
- `__init__.py` insere `vendor/` no **início** do `sys.path` do QGIS.
- **Núcleo em Python puro, sem QGIS:**
  - `hidraulica.py`: Manning em seção circular parcialmente cheia; θ por bissecção.
  - `topologia.py`: nós por arredondamento a 1 cm; orientação por cota ou por
    árvore até um nó de descarga.
  - `dimensionamento.py`.
- **Segmentação e geometria:** `segmentacao.py`, com QGIS só para montar as geometrias.
- **Saídas:** `perfil_dxf.py`, `planta_dxf.py` (DXF R12 escrito à mão, cp1252),
  `memoria_docx.py` e `docx_template.py`.
- **Interface:** `esgoto_dialog.py` e `perfil_interativo_dialog.py` (QGraphicsView).

## 2. Cálculo (`dimensionar`)
Trechos processados de montante para jusante, em ordem decrescente de cota de montante.

**Vazões:**
- Q_dom = C·k1·k2·P·q/86400.
- Q_inf = min(0,00033·L ; 0,25·Q_média acumulada).
- Q_proj = Q_montante + Q_dom + Q_inf.
- Q_calc = max(Q_proj, Q_min). O piso não é acumulado a jusante.

**Diâmetro e declividade:**
- É escolhido o **menor DN** (200…600) com y/D ≤ 0,75.
- S = max(S_terreno, S que dá σ = σ_min, 0,05 %).
- σ = γ·Rh·S, com γ = 9810 N/m³.

**Greide:**
- CF_mont = min(CT_mont − (recobrimento + D), CF de chegada no nó).
- CF_jus = CF_mont − S·L.
- Se CF_jus ficar acima de CT_jus − (recobrimento + D), o CF_jus é rebaixado e S é recalculada.

**Alertas:**
- trecho > 80 m (DN ≤ 350) ou > 100 m;
- lâmina no limite;
- V > 5 m/s;
- profundidade > 3,50 m;
- DN forçado insuficiente.

**Outros resultados:**
- degrau no PV;
- um PV por nó (cota da tampa = terreno).

**Edições manuais:**
- DN forçado;
- profundidade de montante forçada (propaga-se a jusante);
- sentido invertido.

## 3. Normas e referências citadas no código
| Referência | Uso no código |
|---|---|
| **ABNT NBR 9649:1986** | Critérios de dimensionamento, V ≤ 5 m/s, n = 0,013 |
| **COPASA T-194** | DN mín. 200, lâmina 0,75, infiltração, Q_min 1,5 L/s, espaçamento de PV (80/100 m), prof. máx. 3,50 m, k1 e k2 |
| ABNT NBR 14486:2000 | Tensão trativa (texto da memória) |
| NBR 12207, NBR 12208, Lei 11.445/2007, Lei 14.026/2020, NR-18 | Só citadas na memória |
| Tsutiya & Sobrinho (2011); Von Sperling (2014) | Referências bibliográficas da memória |

## 4. Dependências
- QGIS ≥ 3.16.
- python-docx 1.2.0 embarcado.
  - Declara dependência de **`lxml` e `typing_extensions ≥ 4.9`**, que
    **não vêm embarcados**.
  - Se o Python do QGIS não tiver `typing_extensions`, a importação
    do `docx` falha. Não verificado.
- Nenhuma dependência de rede.

## 5. Saídas
- **Camadas em memória:**
  - `rede_esgoto_dim`: trecho, DN, declividade, Q, V, lâmina, σ, CF, profundidades, alerta;
  - `PVs_esgoto`;
  - simbologia por regras.
- **Arquivos:** CSV; perfil DXF (camadas PERFIL_*); planta DXF (camadas DN_*, PV,
  TEXTO, LABEL, QUADRO); memória DOCX com 10 seções + assinatura.

## 6. Limitações conhecidas e pontos de atenção
1. **Memória afirma conformidade de forma fixa.**
   - Na Síntese, “Tensão trativa mínima verificada … — **ATENDE**” e “Lâmina
     máxima … — **ATENDE**” são texto fixo.
   - As Conclusões dizem “**Todos os trechos atenderam** aos critérios de
     tensão trativa mínima e de lâmina máxima”, sem checar os resultados.
2. **Capacidade excedida é escondida.**
   - Quando nem o DN 600 comporta a vazão, o código fixa y/D = 0,75 e usa
     a declividade do terreno.
   - O único alerta é “lâmina no limite (0,75)”; não há aviso de sobrecarga.
   - A tensão trativa desse caso também não é verificada.
3. **Tensão trativa calculada com a vazão de fim de plano** (com piso Q_min).
   - Não existe cálculo de início de plano.
   - A memória diz que a autolimpeza é verificada “para início de plano”.
4. **Sentido do escoamento só por cota de terreno.**
   - O diálogo **nunca passa `no_descarga`**. A orientação por árvore e o
     `ORCAMENTO_SUBIDA`, descritos no changelog do `metadata.txt`, não são usados pela interface.
   - Cruzamentos planos e rebaixamentos podem gerar “nascentes” e
     “descargas” falsas. A correção é manual (inverter sentido).
5. **Recobrimento único.**
   - `recobrimentos` (0,20 m em faixa de servidão) nunca é passado pelo diálogo.
   - Vale sempre o valor do campo (padrão 0,90 m).
6. **Nó fora do MDT = cota 0**, com um aviso só na barra de mensagens.
   - O MDT é amostrado na banda 1, nas coordenadas da rede, **sem
     reprojeção**. Com SRC diferente, todas as cotas viram 0.
7. **Camadas do mapa ficam desatualizadas.**
   - Edições na tabela ou no perfil, inversão de sentido e recálculo
     automático por população atualizam só a tabela e as exportações.
   - Já o recálculo por **edição de geometria** chama `calcular()`, que
     **adiciona novas camadas a cada edição**, acumulando duplicatas no
     projeto, e **descarta as edições manuais**.
8. **Parâmetros da memória são relidos na hora da exportação.**
   - A tabela de parâmetros pode não corresponder ao cálculo feito.
9. **Inconsistências de texto:**
   - a memória cita γ = 10.000 N/m³ (o código usa 9810);
   - a memória fala em escoramento acima de 1,25 m (NR-18), mas o quadro de
     PVs marca a partir de 1,5 m;
   - a planta DXF numera trechos como T1…Tn na ordem da lista, enquanto a
     tabela e o CSV usam T{idx}.
10. **Segmentação:**
    - geometrias multiparte usam só a parte mais longa (com segmentação) ou
      a primeira parte (sem segmentação);
    - sem segmentação, trechos longos não ganham PV intermediário (só alerta).
11. **Planta DXF:** o DN 350 não tem cor própria (cai na cor 7).
12. **`vendor/` no início do `sys.path`.**
    - Esta cópia do `docx` passa a valer para **todos os plugins** da
      sessão do QGIS (ou é sobreposta pela primeira importada).
    - Os três plugins de águas embarcam a mesma cópia.
13. **Textos padrão do escritório:**
    - RT, título e CREA pré-preenchidos;
    - rodapé do DOCX com os dados do autor;
    - cabeçalho “GEO RURAL 3.0” se o ícone não for achado;
    - município padrão “Guiricema-MG”, se chamado sem `info_obra`.
14. **Sem ícone na barra:** a `QAction` é criada sem `QIcon`.
