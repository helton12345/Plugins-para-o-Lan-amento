# Descritivo Técnico-Funcional — Precisa Drenagem

**Pasta:** `precisa_drenagem`
**Versão declarada:** 0.9.1 (experimental)
**QGIS mínimo:** 3.16

**Arquivos lidos:**
- `__init__.py`, `plugin.py`, `drenagem_dialog.py`;
- todo o `core/`;
- `metadata.txt`.

**Compartilhado com o Precisa Esgoto (arquivos idênticos, byte a byte):**
- `hidraulica.py`, `perfil_dxf.py` e `docx_template.py`;
- `vendor/` (python-docx 1.2.0).

`topologia.py` é a versão do esgoto **sem** o `sentido_forcado`.
`planta_dxf.py` é uma variante do esgoto.

---

## 1. Arquitetura
- **Núcleo em Python puro:**
  - `chuva.py`: IDF e Racional;
  - `dimensionamento.py`;
  - `lote_drenagem.py`: cópia adaptada do `engine/lote.py` do terraplanagem_suite;
  - `hidraulica.py` e `topologia.py`.
- **Com QGIS:**
  - `terreno.py`: amostra raster (identify) ou interpola TIN a partir das curvas;
  - `bacias.py`: WhiteboxTools + `gdal:polygonize` + OGR.
- **Saídas:** `perfil_dxf`, `planta_dxf`, `memoria_docx`.
- **Interface:** `drenagem_dialog.py`.

## 2. Cálculo (`dimensionar`)
**Chuva e vazão:**
- i = K·TR^a/(tc+b)^c, **única para toda a rede** (tc fixo).
- Q_própria = C·i·A/360 (m³/s).
- Q = Q_montante + Q_própria.

**Escolha do DN** (menor que atende, com y/D ≤ 0,80):
- S = max(S_terreno, S para V ≥ V_mín, 0,05 %).
- Se o DN não comporta a vazão, tenta “mergulhar”: S necessária para
  a vazão, desde que V ≤ V_máx e vala ≤ 3,00 m.

**Duas passadas:** a 2ª impõe que o DN **não diminua** a jusante (envelope).

**Greide:** igual ao do esgoto (recobrimento + D, continuidade no PV).

**Alertas:** V > V_máx, L > 80 m, lâmina no limite, vala > 3 m.

**Recursos do núcleo não expostos na interface:**
- `sem_tubo` (meia-calha);
- `recobrimentos` por trecho (servidão 0,20 m);
- `no_descarga` (orientação por árvore).

## 3. Sub-bacias
- **Whitebox** (`bacias.py`):
  - `fill_depressions` → `d8_pointer`;
  - para **cada ponto, isoladamente**, `watershed` → polígono (DN = 1);
  - área planar em ha;
  - grava `area_bacia_ha` nos pontos.
  - `vincular_areas_rede` copia essa área para os trechos com ponta a ≤ 1 m do ponto.
- **Lotes** (`individualizar_bacias_lote_ui`):
  - frente = lado mais próximo da rede;
  - plano ajustado às cotas dos cantos;
  - frente × fundo pela média das cotas da metade mais próxima e da mais distante;
  - aceita desnível até o limite de aterro;
  - soma a área no trecho mais próximo do meio da testada (≤ tolerância);
  - camada de auditoria.

## 4. Normas e referências citadas no código
| Referência | Uso |
|---|---|
| **DNIT IPR-724 (Manual de Drenagem de Rodovias, 2006)** | Texto da memória: Tr, tc, n, V mín/máx, lâmina, DN mín |
| DNIT Álbum de Projetos-Tipo (2018), ABNT NBR 10844:1989, DER-MG, DAEE-SP | Só citadas na memória |
| IDF Pluvio/UFV | Origem declarada dos parâmetros K, a, b, c |
| NR-18 | Escoramento > 1,25 m (texto) |
| Tucci; Azevedo Netto; CETESB; Chow et al. | Referências bibliográficas |

## 5. Dependências
- QGIS ≥ 3.16 com Processing (`gdal:polygonize`) e `qgis.analysis` (TIN).
- **`whitebox`**: obrigatório só para a delineação; não declarado.
- python-docx embarcado. Ver o descritivo do esgoto sobre `typing_extensions`.

## 6. Limitações conhecidas e pontos de atenção
1. **Trecho sem área gera declividade de 50 % (bug confirmado por teste).**
   - Com Q = 0, o código usa Q = 1e-6 m³/s. `declividade_para_velocidade`
     não acha V ≥ 0,75 m/s e devolve o teto s_max = 0,5.
   - Teste com os módulos do plugin: trecho de cabeceira de 50 m sem área
     → **DN 400, S = 50 %, vala de 25,5 m**.
   - Pela continuidade, o trecho seguinte (com 1 ha) **herda os 25,5 m**.
   - Acontece com qualquer trecho de montante com área vazia ou 0.
2. **Memória de cálculo — coluna “Q própria (L/s)” errada.**
   - O valor é calculado como C_global·i·A/360, que dá **m³/s**, e é impresso
     como L/s: **1000× menor**.
   - Ignora o C por trecho.
   - A fórmula impressa na seção 6.1 também diz “[L/s]”.
3. **Delineação Whitebox pode contar área em dobro (provável).**
   - Cada ponto é processado sozinho, então cada bacia inclui **toda** a área
     a montante, inclusive as bacias dos outros pontos.
   - Essa área vira “área própria” do trecho, e o dimensionamento ainda
     acumula a vazão de montante.
   - Os pontos também não são ajustados à linha de fluxo (sem
     `snap_pour_points`): um ponto fora do talvegue pode gerar uma bacia minúscula.
4. **Memória sem dados do projeto.**
   - O diálogo chama `gerar_memoria_docx` **sem `info_obra`**.
   - Resultado: empreendimento “____”, município “**Guiricema-MG**” e
     RT/CREA fixos, em qualquer projeto.
   - No esgoto isso foi corrigido na 0.3.1; na drenagem, não.
5. **Síntese “ATENDE” enganosa.**
   - Quando nenhum DN comporta a vazão, y/D é fixado em 0,80 e a síntese
     diz “ATENDE”. Só aparece o alerta “lâmina no limite”.
   - Sem alerta de sobrecarga.
6. **Texto da memória não corresponde ao método:**
   - diz que a declividade adotada é a do terreno; o código também usa a
     declividade de V mínima e o “mergulho”;
   - diz tc como “mínimo DNIT”; é um valor único digitado.
7. **Recobrimento padrão divergente.**
   - O núcleo diz 0,60 m (changelog 0.8); o campo da tela começa em **1,00 m**
     e prevalece.
   - `prof_max` = 3,00 m não é editável.
8. **Sem segmentação:**
   - cada feição é um trecho (multiparte usa só a 1ª parte);
   - nós só nas pontas;
   - cruzamentos no meio da linha não viram PV.
9. **Sentido só por cota** (sem `no_descarga`), e **sem edição manual**
   (tabela somente leitura, sem inverter sentido).
10. **Cotas:**
    - nó fora do MDT recebe cota 0;
    - o raster é amostrado com as coordenadas da rede, **sem conferir o
      SRC** (só a delineação confere).
11. **Lotes reprovados** (drenam para o fundo acima do limite) ficam
    **fora de qualquer trecho**: a vazão é subestimada. O próprio changelog
    diz que a servidão de fundo não é modelada.
12. **Edita e salva camadas do usuário:**
    - `area_bacia_ha` nos pontos;
    - campo de área na rede.
    Não há desfazer depois do commit.
13. **Delineação na thread principal** (o QGIS congela) e exige MDT em
    arquivo (`mdt_layer.source()`).
14. **Planta DXF:** DN 800–1500 sem cor própria (camada cor 7).
15. **Sem ícone na barra;** `vendor/` no início do `sys.path` (ver esgoto).
