# Divergências do metadata.txt — Precisa Drenagem

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `about` | “velocidade entre 0,75 e 5,0 m/s” | Com vazão nula (trecho sem área) a declividade vai a 50 % e a vala a dezenas de metros (Descritivo, item 1). V > 5 só gera alerta | Alta (funcional) |
| 2 | Changelog 0.8 | `p.recobrimento` passou de 1,00 para 0,60 m; servidão 0,20 m; `sem_tubo` | O diálogo abre com **1,00 m** e o usa. `recobrimentos` e `sem_tubo` não são expostos na interface | Média |
| 3 | Changelog 0.2.4/0.7 | Orientação por árvore até o nó de entrega | O diálogo nunca passa `no_descarga`: a orientação é sempre por cota | Média |
| 4 | `about` / `description` | Não menciona | Delineação de sub-bacias (Whitebox), vínculo de áreas, individualização por lotes, CSV, DXF, memória DOCX. Também não avisa que **edita e salva** as camadas de pontos e de rede | Baixa |
| 5 | Dependências | Não declaradas | `whitebox` (delineação), Processing/GDAL, python-docx embarcado | Baixa |
| 6 | `icon` | Vazio | Sem ícone na barra. `precisa_icon=icon.png` existe | Baixa |
| 7 | `about` | “CRS padrão SIRGAS 2000 / UTM 23S” | Aceita qualquer SRC projetado. EPSG:31983 é só *fallback* | Informativo |
| 8 | `tracker` / `repository` | `example.invalid` | URLs inválidas | Baixa |
| 9 | `version` | 0.9.1 | O changelog do **esgoto** 1.0 cita “drenagem 1.7/2.0” com tabela editável, perfil interativo e simbologia, que **não existem** neste pacote. Todos os arquivos são de 21/09 | Informativo |
| 10 | `qgisMinimumVersion` | 3.16 | APIs compatíveis | — |

**Resumo:** metadata **com divergências**:
- o comportamento hidráulico prometido falha em trechos sem área;
- os parâmetros descritos no changelog não chegam à interface;
- os efeitos colaterais de edição de camadas não são mencionados.
