# Descritivo Técnico-Funcional — GEO URBANO

**Pasta:** `geo_urbano`
**Versão declarada:** 1.6.0
**QGIS mínimo:** 3.16

**Arquivos lidos:**
- todos os `.py` de `core/`, `documentos/` e `ui/`;
- `plugin.py`, `__init__.py` e `metadata.txt`.

**Não analisados linha a linha:** os estilos `.qml` e o template
`templates/MODELO_GEO.qpt`, que só foram consultados pontualmente.

---

## 1. Arquitetura
- **Interface:**
  - `plugin.py`: recria o diálogo a cada abertura.
  - `ui/dialog_principal.py`: janela não-modal com 5 abas.
  - `ui/dialog_lado.py`: classificação de cada segmento e ficha do confrontante.
  - `ui/dialogo_pessoa.py`: fichas de proprietário e de confrontante.
  - `ui/util_dialog.py`: janelas não-modais com `QEventLoop`.
- **Núcleo (`core/`):**
  - `geometria` e `curvas`: segmentos, curvas, lados e confrontantes.
  - `camadas` e `cotas`: camadas de saída.
  - `divisao_engine`, `desmembramento_engine` e `unificacao_engine`: operações geométricas.
  - `pessoas_db`: banco ODS escrito com a biblioteca padrão (zip + XML).
  - `backup_sessao`, `docx_template` e `genero` (flexão de texto).
  - `compat`: compatibilidade entre QGIS 3 e 4.
- **Documentos (`documentos/`):**
  - memoriais tabular e georreferenciado;
  - requerimentos;
  - `planta.py`: planta A3 montada em código;
  - `planta_qpt.py`: motor do GEO RURAL sobre `MODELO_GEO.qpt`;
  - `planta_qpt_urbano.py`: adaptador urbano desse motor.

## 2. Processamento de um lote (`_processar_geometria`)
1. **Preparação do anel:**
   - extrai o **anel exterior da 1ª parte**;
   - remove segmentos menores que 0,1 mm;
   - começa no vértice mais ao norte;
   - arredonda as coordenadas para 3 casas.
2. **Área e perímetro no plano** (`area_perimetro_plano`):
   - área: fórmula de Gauss sobre E/N;
   - perímetro: soma das distâncias, cada uma arredondada a 2 casas.
3. **Trava de consistência** (`validar_consistencia`): fechamento ΣΔE/ΣΔN
   e comparação de área e perímetro (ver limitação 13).
4. **Segmentos** (`analisar_segmentos`):
   - uma **curva** exige ≥ 100 segmentos consecutivos de mesmo comprimento
     (tolerância de 1 mm) e raio estável;
   - todo o resto é tratado como reta.
5. **Frente automática:**
   - vale o segmento com ≥ 60 % dos pontos centrais a até 1 cm de um logradouro;
   - só funciona se houver **um único** logradouro;
   - com dois ou mais, nada é marcado.
6. **Classificação manual** dos demais segmentos (uma janela por segmento).
7. **Confrontante por lado:**
   - buffer crescente (0,01 → 0,20 m) no ponto central da corda
     início–fim do lado;
   - busca na camada de confrontantes, na camada do próprio lote (excluindo
     o próprio lote) e nos logradouros;
   - filtro angular de 45°;
   - só é aceito automaticamente quando há **1** candidato; senão, abre a ficha.

## 3. Saídas
| Saída | Implementação |
|---|---|
| Memorial tabular | Um parágrafo por lado. Comprimento total do lado; se houver curva, raio e ângulo central |
| Memorial geo | Narrativa vértice a vértice: coordenadas N/E em formato brasileiro, azimute em GMS, distância, confrontante e curvas (raio, desenvolvimento, AC). Fecha com o texto do sistema de projeção extraído do SRC |
| Camadas | Imóvel, Limite (LIMITE_GEO_URBANO.qml), Vértices (PONTOS_GEO.qml, V-01…), Mudança de Confrontantes, Nomes Confrontantes (rótulo fora do polígono, CPF “___.___.___ - __” se vazio) e estilo do logradouro. Salvas como SHP em `camadas/` |
| Cotas | Camada de pontos com comprimento (formato brasileiro), azimute e área/perímetro no centroide, usando as mesmas distâncias do memorial |
| Planta A3 (principal) | `MODELO_GEO.qpt` com textos injetados no XML. Inclui: todas as camadas visíveis do projeto, grade UTM + geográfica, escala automática (múltiplos de 50), declinação magnética (pygeomag ou API NOAA online), convergência meridiana, fator k e croqui de localização (tiles ESRI, 12,5 km). Abre no compositor sem exportar |
| Planta A3 (reserva / operações) | `documentos/planta.py`, montada em código. Usada se a do QPT falhar e nas plantas de desmembramento, divisão e unificação |
| Requerimentos | Desmembramento, divisão, retificação e unificação, com qualificação completa, cônjuge, bloco PEP e assinaturas |
| Banco de pessoas | `~/GEO_URBANO_DADOS/banco_pessoas.ods`, com migração de `~/.geo_urbano`. Atualiza por CPF |
| Sessão | JSON versionado (até 5) em `<saída>/backups/` |

## 4. Operações
- **Desmembramento e divisão:**
  - `splitGeometry` pela linha, com *fallback* por buffer de 1e-6;
  - ou interseção/diferença com o polígono de corte;
  - checagem de conservação de área (Δ < 0,5 m²).
- **Unificação:**
  - exige segmentos de divisa com vértices coincidentes (A→B num lote e
    B→A no outro, arredondados ao mm);
  - depois faz `unaryUnion`;
  - se o resultado for multiparte, usa a maior parte com aviso;
  - Δ de área < 1 m².

## 5. Normas e referências citadas no código
| Referência | Onde |
|---|---|
| Lei 6.015/1973: art. 167, I, nº 18; art. 213, II e §14; art. 227; art. 234; art. 235 | Requerimentos e comentários dos motores |
| Lei 6.766/1979 | Requerimentos de desmembramento e divisão |
| Provimento Conjunto CGJ/MG nº 93/2020 (redação do nº 142/2025) | Cabeçalho do `requerimento.py` (qualificação) |
| Provimento CNJ nº 149/2023 (Código Nacional de Normas) | Bloco de declaração PEP |
| SIRGAS 2000 / GRS80 / WMM | Textos da planta, fator k, declinação |

## 6. Limitações conhecidas e pontos de atenção
1. **Fuso e meridiano central errados nas plantas de `planta.py`** (bug
   confirmado na leitura do código).
   - Onde aparece: planta de reserva e **todas** as plantas de
     desmembramento, divisão e unificação.
   - Causa: `zona_utm = epsg − 31971` e a tabela `_mc_por_epsg` estão deslocadas.
   - Exemplo: para EPSG:31983 (SIRGAS 2000 / UTM 23S), a planta mostra
     “UTM Fuso **12S**” e “Meridiano Central: **-9°W**”.
   - O correto seria 23S e 45°W. O `core/geometria.extrair_info_crs` e o
     `core/camadas` usam a fórmula certa (`epsg − 31960`).
2. **CPF dos proprietários não sai nos memoriais.**
   - `memorial_tabular` e `memorial_geo` leem `p['cpf']`.
   - A ficha e o banco gravam `cpf_cnpj`.
   - O CPF nunca aparece no cabeçalho.
3. **“Parte menor / maior → Desmembrado” não é garantido.**
   - `desmembrar_poligono` escolhe por índice (0 ou 1) na ordem devolvida
     pelo `splitGeometry` ou por `[dentro, fora]`, **sem ordenar por área**.
   - Com mais de 2 partes, a lista é ordenada da maior para a menor, e o
     “menor” (índice 0) passa a ser a **maior**.
   - Confira as áreas na mensagem final.
4. **Divisão limitada:**
   - só a **1ª feição** da camada de corte é usada;
   - na prática, a divisão costuma gerar 2 partes.
   - As partes da divisão saem **sem proprietários** nas camadas; os
     memoriais usam os proprietários da aba Dados.
5. **SRC não validado.**
   - Tudo é calculado no plano da camada.
   - Em SRC geográfico, área, distâncias e cotas saem em graus, sem aviso.
6. **Curvas quase nunca detectadas.**
   - `MIN_SEG_CURVA = 100` segmentos uniformes (±1 mm).
   - Arcos com menos vértices viram várias retas, e cada uma exige
     classificação manual.
7. **Classificação manual segmento a segmento:**
   - só a frente é automática, e apenas com um único logradouro;
   - fundos e laterais são sempre manuais.
   - Segmentos deixados sem lado (“Continuar?” = Sim) **não entram no
     memorial tabular**.
8. **Planta QPT, slots de confrontantes (provável).**
   - O adaptador urbano envia `matricula=''`, e `_preencher_layout` troca
     vazio por **“Posseiro”**.
   - Resultado: “Mat. Posseiro” em todos os confrontantes, inclusive ruas.
   - Esses slots só são preenchidos via `sip.cast`. Se o cast falhar
     (`import sip` indisponível), ficam os textos do template (modelo
     “SÍTIO BOA VISTA”).
   - Não testado em QGIS.
9. **Extensão e camadas da planta QPT:**
   - o enquadramento usa a extensão da **camada inteira** do lote (não do
     lote processado);
   - entram todas as camadas visíveis do projeto;
   - o texto “[Zona NN**S**]” é fixo no hemisfério sul.
10. **Textos fixos:**
    - “/MG” no cabeçalho da planta de reserva e em todos os requerimentos;
    - datum “SIRGAS 2000” fixo na planta de reserva e no campo `datum` da camada Limite;
    - município padrão “Guiricema” na data, quando o campo está vazio;
    - RT padrão = dados do autor, em `base_docx.RT` e `docx_template`;
    - retificação: afirma “foram obtidas as anuências de todos os confrontantes”;
    - unificação: afirma “mesmo proprietário e sem ônus”.
11. **Shapefile de vértices (opção Extras):** percorre **todas as feições**
    da camada do lote, não só a processada.
12. **Unificação estrita:**
    - exige vértices idênticos (1 mm) nas divisas;
    - lotes contíguos com vértices em “T” falham com “Nenhuma divisa interna”.
13. **`validar_consistencia` é quase tautológica.**
    - Compara área e perímetro com a mesma função que os calculou.
    - O teste de fechamento é sobre um anel que sempre fecha.
    - Na prática, a trava não dispara.
14. **Instalação automática do `python-docx`:**
    - `garantir_docx` roda `pip install --break-system-packages` **sem perguntar**;
    - com `sys.executable` no Windows, pode não funcionar.
15. **Internet:**
    - declinação via API NOAA quando o `pygeomag` não está instalado;
    - croqui com tiles ESRI.
    - Sem conexão, os campos ficam “N/D” e o croqui não sai.
16. **Sessão parcial:** não guarda lados, confrontantes, RT nem opções.
17. **Código herdado do GEO RURAL sem uso ou incompleto:**
    - `planta_qpt._calc_area_perimetro` importa `core.geodetic`, que não
      existe (só é usado com `pre_calculado=False`, fora do fluxo urbano);
    - `fase0_coleta.satelite` também não existe (cai no ESRI direto);
    - `_aplicar_offsets_confrontantes` está desativado;
    - `maior_parte` tem erro de precedência no `return`, mas não é usado;
    - o QPT temporário se chama `MODELO_GEO_georural_tmp.qpt`.
18. **Compatibilidade com QGIS 4 (Qt6) não verificada:**
    `QApplication.desktop()` em `dialog_lado.py` não existe no Qt6.
