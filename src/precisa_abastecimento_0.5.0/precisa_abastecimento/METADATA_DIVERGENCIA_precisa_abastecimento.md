# Divergências do metadata.txt — Precisa Abastecimento

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | Changelog 0.4.0/0.4.1 | Base do reservatório pelo greide/terreno do lote, “não mais o nó de maior cota do terreno, que é chute” | Só a **cota** muda. A **origem da árvore** continua no nó de maior cota (Descritivo, item 1) | Alta (funcional) |
| 2 | Changelog 0.4.2/0.4.3 | Memória com os dados reais do projeto | Os dados de identificação vão, mas os **parâmetros** saem com piezométrica de origem 0,00 m e a população do campo manual (Descritivo, item 2) | Alta (documento) |
| 3 | `about` | “verificação de pressão nos nós (10 a 50 mca)” | Só pressão dinâmica. A pressão estática não é verificada | Média |
| 4 | `about` | “Detecta anéis/malha e avisa” | Detecta e avisa, mas **descarta** os trechos de fechamento (sem DN, fora das camadas) | Média |
| 5 | `description` / `about` | Não menciona | Recalque (Bresse), população por lotes, altura automática, DN editável, perfil interativo, CSV/DXF/DOCX | Baixa |
| 6 | Dependências | Não declaradas | python-docx embarcado | Baixa |
| 7 | `icon` | Vazio | Sem ícone na barra. `precisa_icon=icon.png` existe | Baixa |
| 8 | `tracker` / `repository` | `example.invalid` | URLs inválidas | Baixa |
| 9 | Changelog 0.5.0 | “sentido já vem sempre da árvore de caminhos mínimos a partir da fonte” | É uma BFS (menor número de trechos), não caminhos mínimos por comprimento | Informativo |
| 10 | `about` | “CRS padrão SIRGAS 2000 / UTM 23S” | Aceita qualquer SRC projetado (31983 é só *fallback*) | Informativo |
| 11 | `version` / `qgisMinimumVersion` | 0.5.0 / 3.16 | Datas coerentes (19/09). APIs compatíveis | — |

**Resumo:** metadata **com divergências**:
- o changelog promete que o reservatório sai do lote escolhido, mas o
  código só usa a cota desse lote;
- a memória não reflete os parâmetros realmente calculados.
