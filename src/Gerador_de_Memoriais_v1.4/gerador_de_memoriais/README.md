# Gerador de Memoriais

Gera memorial descritivo georreferenciado a partir de três fontes diferentes — texto narrativo, camada QGIS ou PDF do SIGEF/INCRA — com pontos e polígono.

## Instalação

**Via repositório oficial QGIS:**
1. QGIS → Plugins → Gerenciar e Instalar Plugins → buscar "Gerador de Memoriais"
2. Instalar

**Dependência externa — python-docx (modos Camada QGIS e SIGEF):**
```
pip install python-docx
```

**Dependência externa — pdfplumber (modo SIGEF):**
```
pip install pdfplumber
```

No Windows, use o OSGeo4W Shell:
```
python -m pip install python-docx pdfplumber
```

**Manual:**
1. Baixe o ZIP no [GitHub](https://github.com/precisagrimensuratop-byte/Plugins)
2. QGIS → Plugins → Gerenciar e Instalar Plugins → Instalar a partir do ZIP

## Requisitos

| Item | Versão |
|---|---|
| QGIS | 3.16 ou superior |
| python-docx | Necessário para os modos Camada QGIS e SIGEF |
| pdfplumber | Necessário para o modo SIGEF |
| Modo Texto | Sem dependências externas |

## Funcionalidades

### Aba 1 — Texto
Cole ou carregue (.txt/.docx) um memorial descritivo narrativo já pronto. O plugin reconhece automaticamente coordenadas em UTM ou Geográfico (lat/lon), extrai vértices, azimutes, distâncias e confrontantes já descritos no texto, e gera camadas de pontos e polígono no QGIS (com opção de salvar como shapefile).

### Aba 2 — Camada QGIS
Selecione uma camada de polígono já desenhada, **em CRS projetado UTM** (ex: SIRGAS 2000 / UTM — obrigatório; camadas em CRS geográfico produzem medidas incorretas). O plugin gera o memorial descritivo georreferenciado, descrevendo o perímetro vértice a vértice — com **detecção automática de curvas** (raio, desenvolvimento, ângulo central) para precisão técnica. O sistema de cálculo é identificado a partir do CRS da camada e declarado no texto. Confrontantes ficam como campo `[PREENCHER CONFRONTANTE]` para preenchimento manual — este modo não faz classificação automática de lado (frente/fundos/laterais).

Inclui também geração opcional de shapefile de vértices numerados (prefixo configurável).

### Aba 3 — SIGEF
Lê o PDF oficial do SIGEF/INCRA (tabela de vértices) e gera:
- Memorial descritivo narrativo em DOCX, com detecção de curva (raio, ângulo central, corda, concavidade)
- Pontos e polígono das parcelas (opcional)

Confrontante fica como `[PREENCHER CONFRONTANTE]` — este modo não extrai nem classifica confrontantes.

## Sobre a fusão

Este plugin substitui três plugins anteriores do mesmo autor: **Restituição**, **Memorial Rural Gerador** e **Memorial SIGEF**. O modo Camada QGIS não inclui mais a geração de memorial tabular com classificação automática de confrontantes por lado (frente/fundos/laterais) — essa funcionalidade faz parte dos produtos comerciais DANI e GEO URBANO.

## Precisa de mais?

Para memorial mais completo — com planta, classificação automática de confrontantes por lado e mais formatos de saída — experimente o [visualizador de avaliação online](https://geo-urbano-avaliacao.onrender.com/) (sem cadastro).

## Licença

GPL v3 — veja [LICENSE](LICENSE).

## Suporte

https://github.com/precisagrimensuratop-byte/Plugins/issues
Não há suporte via e-mail ou mensagem direta.
