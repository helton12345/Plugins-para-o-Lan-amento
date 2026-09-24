# Divergências metadata.txt × código — GEO RURAL 3.0 (v3.199)

Este relatório compara o `metadata.txt` com o código. O `metadata.txt` **não foi editado**.

## Divergências encontradas

| # | Chave / trecho do metadata | O que diz | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `about`, "Fase 1 (SIGEF — XLSX, shapefile, **XML**)" e tag `xml` | O plugin gera XML | A opção de XML (`chk_f1_xml`) não aparece na tela nem vem marcada, então a interface **nunca gera o XML**. O XML do módulo usa namespace e XSD próprios, não o formato oficial do SIGEF **[PROVÁVEL]**. | Alta |
| 2 | `about`, "Fase 3 (**submissão eletrônica**)" | Submete ao SIGEF | Só monta o ZIP, o checklist e o log. O upload por Selenium é um esqueleto que sempre responde "Submeta manualmente". O ZIP procura nomes antigos na raiz da pasta e tende a sair incompleto. | Alta |
| 3 | `about`, "Inclui suite de **18** testes" | 18 testes | `tests/run_all.py` tem **19** (19/19 OK na execução). `fase5_divisao/test_divisao.py` tem 1 teste que **falha** (desatualizado). O README diz 12. | Baixa |
| 4 | `about`, "Compatível com QGIS 3.16+ e **QGIS 4.x**" | Roda no QGIS 4 | `core/compat.py` trata alguns enums, mas a interface usa `exec_()` (14 ocorrências), enums curtos do Qt5 (`Qt.AlignCenter`, `QFont.Bold`…) e `QVariant`. Nada disso existe no PyQt6 **[PROVÁVEL]**. Não testado. | Média |
| 5 | `about`, "Fase 5 … **seleção de linha de camada**" | Corta por linha escolhida no mapa | O módulo existe (`linha_layer_selector.py`), mas a aba atual só **importa SHP com um polígono por parcela**. As rotinas de linha de corte usam um campo (`edt_linha`) que não é criado. | Média |
| 6 | `about`, "Fase 0 (coleta externa — … **CAR, SIGEF** …)" | Coleta online | Na interface, o CAR tenta o WFS do SICAR e depois a pasta local; o SIGEF é **só pasta local**. O README afirma que as consultas online foram removidas, mas o código do CAR (e o módulo `fase0/sigef.py`) ainda consulta. | Baixa |
| 7 | `about`, "Fase 0 … **RBMC**" | Baixa o kit PPK | O kit baixa o RINEX, mas **falha** nas efemérides e no IONEX (`TypeError`, confirmado por teste). | Alta |
| 8 | `about`, "**12 tipos** de anuência e checklist para os **27 estados**" | 27 UFs cobertas | Há 12 tipos e as 27 UFs estão cadastradas, mas só **MG** tem artigos específicos. As outras 26 usam citação genérica (a própria interface avisa "revise as citações"). O **Requerimento ao CRI** e o **Cancelamento SIGEF** só existem para MG. | Média |
| 9 | `about`, "requerimentos completos para MG/Prov. 93 TJMG" | Requerimentos corretos | Há catálogos de atos com incisos que se contradizem entre dois módulos, citação truncada do art. 213 §14 e textos que afirmam fatos não verificados (ausência de sobreposição, anuências obtidas, PEP pré-marcado). Ver o Descritivo §6.1. | Alta |
| 10 | `about`, "conforme **MTGIR 2ª** e **NTGIR 3ª**" | Conformidade técnica | Pontos a conferir: tabelas de método **inconsistentes** entre o leitor de TSV e o exportador ODS; FLUTUANTE vira PG3; σh comparado ao limite de posição; SRTM ortométrico gravado como elipsoidal; precisão inventada na rota "polígono sem GNSS". **[A CONFIRMAR]** na norma. | Alta |
| 11 | `about`, "Fase 5 … delta=0.00e+00" | Conservação exata de área | Confirmado nos testes para o corte por linha (motor). Na interface atual (importar SHP pronto), a soma das áreas é **exibida** com o Δ real, que pode não ser 0. | Baixa |
| 12 | `description`, "fases 0 a 5" e `precisa_label` | Fases 0 a 5 | O título da janela diz "Fases 0 a 4" e o cabeçalho diz "Fases 1 a 4". A Fase 5 é injetada depois. | Baixa |
| 13 | Dependências (não há chave no metadata) | — | O código exige **python-docx** e **openpyxl** e usa, conforme a função, **pdfplumber**, **odfpy**, **pyshp/fiona** (CLI/testes), **pystac-client**, **pygeomag**, **selenium** e o executável **convbin** do RTKLIB. Sem openpyxl, o exportador XLSX quebra (`NameError`). | Média |
| 14 | `email=precisagrimensuratop@gmail.com` | Contato do autor | O RT padrão da interface usa outro e-mail (`heltonjclourenco@…`). O rodapé fixo dos DOCX usa o do metadata. | Baixa |
| 15 | `experimental=False` | Versão estável | Há bugs confirmados de alta gravidade (itens 1, 2, 7, 9 e 10; ver também o Descritivo §6.1: numeração M duplicada, conflito de tabela na Gestão, sessão que não restaura dados, dados pessoais do autor nos documentos). | Opinião: revisar |
| 16 | `tracker=`, `repository=`, `homepage=` | Vazios | — | Baixa (o repositório oficial do QGIS exige tracker e repository) |
| 17 | `hasProcessingProvider=no` | Sem provedor Processing | Confirmado: não há provedor. OK. | — |
| 18 | Versão | `version=3.199` | O README diz 3.59; a pasta se chama `plugin_geo_rural_3_59`; os testes e o CLI importam o pacote `plugin_geo_rural_3`; a Gestão importa `plugin_geo_rural_3_54`. Nomes de pacote diferentes quebram imports absolutos conforme o nome da pasta instalada. | Média |

## Consistente

- `name`, `author`, `category=Vector`, `qgisMinimumVersion=3.16` (sem uso detectado de API mais nova, mas **não verificado**).
- `icon=icon.png` e `precisa_icon=icon.png` existem.
- As chaves `precisa_suite`, `precisa_submenu` ("🗺️ Georreferenciamento"), `precisa_label`, `precisa_tooltip` e `precisa_ordem` são lidas por `plugin.py` e montam o menu "Precisa Agrimensura".
- As tags `sigef`, `incra`, `shapefile`, `averbacao`, `cartorio`, `provimento93`, `car`, `rtklib`, `topodata`, `divisao`, `desmembramento` e `unificacao` correspondem a funções existentes.

**Resultado:** há divergências (17 itens com divergência, 5 de gravidade alta; o item 17 é só confirmação).
