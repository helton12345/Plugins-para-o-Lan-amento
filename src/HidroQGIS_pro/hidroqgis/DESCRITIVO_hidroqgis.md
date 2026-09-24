# Descritivo Técnico-Funcional — HidroQGIS

**Pasta:** `hidroqgis` · **Versão declarada:** 1.1.0 · **QGIS mínimo:**
3.16 (máx. 4.99).

**Arquivos lidos:** todos os `.py` de:
- `processing/`, `publicacao/`, `reporting/`, `utils/`, `ui/`, `test/`;
- `hidroqgis.py`, `__init__.py`.

Também foram lidos `metadata.txt` e `publicacao/templates/site_template.html`.

---

## 1. Arquitetura
- **Entrada e interface:**
  - `hidroqgis.py`: ação, menu e `run()` para a Suíte.
  - `ui/hidroqgis_dialog.py`: diálogo único.
  - `utils/validator.py`: validação das entradas.
- **Processamento:**
  - O pipeline roda numa **`QgsTask` (`TarefaHidrologica`)**.
  - Chama o WhiteboxTools pela biblioteca `whitebox`.
  - Chama `processing.run` (algoritmos `gdal:` e `native:`).
- **Pós-processamento** (na thread principal, após o término da task):
  - `reporting/`: planta, memorial, KMZ, hillshade, croqui;
  - `publicacao/`: visualizador web;
  - `processing/comparacao_temporal.py`.

## 2. Pipeline (`TarefaHidrologica`)
| # | Etapa | Implementação |
|---|---|---|
| 0 | Recorte (opcional) | `gdal:cliprasterbymasklayer` com o limite |
| 1 | Remoção de depressões | WBT `fill_depressions` |
| 2 | Direção de fluxo | WBT `d8_pointer` (`esri_pntr=True`) |
| 3 | Acumulação | WBT `d8_flow_accumulation` (saída em células) |
| 4 | Drenagem | WBT `extract_streams` + `raster_streams_to_vector` (ver limitação 4) |
| 5 | Sub-bacias | WBT `subbasins` → `gdal:polygonize` (campo `id_bacia`) |
| 6 | Corpos d'água | Acumulação ≥ limiar → polygonize → `native:extractbyexpression` (`$area >= mínimo`) |
| 7 | Exportação | Camadas para `HidroQGIS_resultados.gpkg` |
| 8 | Carga | Grupo “HidroQGIS — Resultados” |

O `validator` exige:
- arquivos existentes;
- limiar de corpos > limiar de drenagem;
- diretório gravável.

## 3. Funcionalidades complementares
- **Download TOPODATA:** `pystac_client` no catálogo BDC/INPE.
  - Coleção `topodata-1`, asset `ZN`.
  - A busca usa a extensão do canvas convertida para EPSG:4326.
  - Se houver várias folhas, elas são unidas com `gdal:merge`.
- **Planta PDF** (`reporting/planta_qpt.py`): `QgsPrintLayout` montado em código.
  - Folhas A4 a A0; saída em 300 dpi.
  - Relevo sombreado (`hillshade.py`) e hipsometria.
  - Curvas de nível com equidistância automática.
  - Croqui de localização (`croqui_localizacao.py`, fundo ESRI).
  - Grade, legenda, escala, norte e carimbo.
  - O compositor é aberto no designer ao final.
- **Memorial DOCX** (`reporting/memorial_bacia.py`, python-docx): morfometria por sub-bacia.
  - Área e perímetro.
  - **Kc = 0,28·P/√A**.
  - **Dd = L/A** (comprimento de drenagem dentro da sub-bacia ÷ área).
  - Linhas finais TOTAL/MÉDIA.
  - Metodologia e referências bibliográficas.
- **KMZ** (`reporting/kmz_export.py`): drenagem, sub-bacias e corpos com estilo.
- **Visualizador web** (`publicacao/`):
  - HTML Leaflet com as camadas em GeoJSON embutido;
  - publicação via API Netlify (ZIP) ou gravação local;
  - a API key pode ser salva em `config_local`.
- **Comparação temporal** (`processing/comparacao_temporal.py`):
  - roda o mesmo pipeline no MDE anterior em `comparacao_temporal_mde_anterior/`;
  - pareia cada sub-bacia com a de maior interseção;
  - grava `comparacao_temporal.shp` com as diferenças de área, perímetro, Kc e Dd;
  - acrescenta a seção 6 ao memorial.

## 4. Normas e referências citadas no código
Não há norma técnica aplicada ao cálculo. O memorial cita:

| Referência | Uso |
|---|---|
| TUCCI (2009); CHRISTOFOLETTI (1980) | Kc, Dd e interpretação morfométrica |
| VALERIANO (2008) | TOPODATA |
| NOVO (2010); FLORENZANO (2011) | Sensoriamento remoto / geomorfologia |
| Resolução IBGE nº 1/2005 | SIRGAS 2000 (texto) |

## 5. Dependências
| Item | Obrigatória | Observação |
|---|---|---|
| QGIS + Processing (GDAL/native) | Sim | Usados pelo pipeline, apesar do que diz o `about` |
| `whitebox` (WhiteboxTools) | Sim | Baixa o binário na 1ª execução |
| `python-docx` | Não | Memorial |
| `pystac-client` | Não | Download do MDE |
| Internet | Não | Download TOPODATA, fundo ESRI do croqui, Netlify |

O `utils/dependency_installer.py`:
- pergunta antes de instalar;
- usa `sys.executable -m pip install --user`.

## 6. Limitações conhecidas e pontos de atenção
1. **Ícone ausente na barra.**
   - `initGui` procura `resources/icon.png`, que **não existe**.
   - O ícone está na raiz (`icon.png`).
   - Resultado: botão sem ícone. Pela Suíte, `precisa_icon=icon.png` funciona.
2. **Publicação Netlify sem `_headers`.**
   - O ZIP enviado contém só `index.html`.
   - O Geo Visualizador (mesmo padrão) documenta que, sem o arquivo
     `_headers`, o Netlify pode servir o HTML como `text/plain`.
   - Não verificado em produção.
3. **JavaScript do visualizador sem bacias.**
   - O enquadramento chama `fitBounds` usando `camadaBacias`.
   - Sem sub-bacias, isso gera um `ReferenceError` e o mapa não é enquadrado.
   - Mesmo com bacias, o enquadramento considera só essa camada.
4. **Fallback da drenagem gera polígonos.**
   - Quando `raster_streams_to_vector` falha, o fallback usa rastercalculator
     + polygonize e produz **polígonos**, não linhas.
   - A camada de drenagem muda de geometria.
   - A Dd do memorial sai **NULL** e o KMZ/web mostram manchas.
5. **“Corpos d'água” = limiar de fluxo acumulado.**
   - São células com acumulação ≥ limiar, não lâminas d'água detectadas.
   - É uma limitação metodológica, não um erro de código, mas o nome pode
     induzir a erro.
6. **Textos fixos na planta e no memorial:**
   - “SIRGAS 2000 (EPSG:4674)” e “Fonte: TOPODATA/INPE”, qualquer que seja
     o SRC ou o MDE de entrada.
7. **Unidades.**
   - Áreas e perímetros vêm da geometria no SRC da camada.
   - Em MDE geográfico (o TOPODATA vem em EPSG:4674) os valores podem não
     estar em m/km² se não houver reprojeção.
   - Não verificado para todos os caminhos.
8. **Linha TOTAL do perímetro.**
   - Soma os perímetros das sub-bacias, o que conta duas vezes os limites
     internos.
   - É uma grandeza sem significado físico.
9. **Comparação temporal.**
   - Executa `calcular_morfometria` editando a camada de sub-bacias
     **in-place** (adiciona e atualiza campos).
   - O pareamento pela maior interseção é heurístico: bacias divididas ou
     fundidas não são tratadas.
10. **Download do MDE na thread da interface:** o QGIS fica congelado até
    terminar.
11. **Código não usado pela interface:**
    - `processing/validation.py` (índice de Jaccard);
    - `utils/crs_handler.py`;
    - a camada de `flow_direction` não é consumida a jusante.
12. **Testes:** `test/test_processing.py` só faz *asserts* de constantes.
    **Nenhum processamento real é testado.**
13. **Ambiente Windows.**
    - `sys.executable` pode apontar para `qgis-bin.exe`.
    - Nesse caso a instalação automática de dependências falha.
    - Não verificado aqui.
14. **Versão na interface.**
    - O rodapé mostra “HidroQGIS v1.0”; o metadata diz 1.1.0.
    - Todos os arquivos têm a mesma data (06/09).
