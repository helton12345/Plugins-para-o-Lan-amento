# Divergências do metadata.txt — GEO URBANO

Somente relatório. **Nenhuma alteração foi feita no `metadata.txt`.**

| # | Campo | Declarado | O que o código faz | Gravidade |
|---|---|---|---|---|
| 1 | `icon` / `precisa_icon` | `icon.svg` | **Não existe `icon.svg`**, só `icon.png`. O `plugin.py` também carrega `icon.svg`, então o botão e a Suíte ficam sem ícone | Média |
| 2 | `about` | “Trabalha em coordenadas UTM SIRGAS 2000” | O SRC não é verificado: aceita qualquer SRC e calcula no plano da camada. A planta de reserva mostra fuso e MC errados (Descritivo, item 1) | Média |
| 3 | `about` | “imóveis urbanos simples (lote, desmembramento)” | Também faz divisão e unificação geométricas, com memoriais, plantas “situação atual/futura” e requerimento de divisão | Baixa |
| 4 | `about` | “identifica os 4 lados interativamente” | Correto. A frente só é automática com um único logradouro | — |
| 5 | `description` / `about` | Não menciona | Banco de pessoas ODS, backup de sessão, camadas SHP estilizadas, cotas, planta QPT com declinação/convergência/fator k/croqui, aba RT | Baixa |
| 6 | Dependências | Não declaradas | `python-docx`, instalado automaticamente via pip; `pygeomag` opcional; internet para a NOAA e os tiles ESRI | Baixa |
| 7 | `version` | 1.6.0 | O cabeçalho do `dialog_principal.py` diz “GEO URBANO v2.0”. `metadata.txt` é de 23/09 00:33; os motores de corte e o diálogo, de 23/09 00:24–00:25 | Informativo |
| 8 | `qgisMinimumVersion` | 3.16 | APIs com *fallback* (`writeAsVectorFormatV3` → `writeAsVectorFormat`; `Qgis.LabelPlacement` → constantes antigas). Consistente | — |
| 9 | `tracker` / `repository` / `homepage` | Vazios | — | Baixa |
| 10 | `precisa_submenu` | 📄 Memoriais | Consistente | — |

**Resumo:** metadata **com divergências**:
- o ícone declarado (e usado pelo código) não existe;
- a promessa “UTM SIRGAS 2000” não é garantida;
- o escopo real é maior que o descrito.
