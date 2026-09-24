# FAQ — Gerador de Memoriais

## Erro "python-docx não está instalado"

Necessário para os modos Camada QGIS e SIGEF:
```
pip install python-docx
```
No Windows (OSGeo4W Shell como administrador):
```
python -m pip install python-docx
```

## Erro "pdfplumber não encontrado" (modo SIGEF)

```
pip install pdfplumber
```

## Modo Texto: o parser não encontra vértices

Suporta apenas o padrão narrativo com "vértice X, de coordenadas N ... e E ..." (UTM) ou "vértice X ..., Longitude ..., Latitude ..." (geográfico). Memoriais em formato tabular (planilha) não são reconhecidos por este modo — use o modo Camada QGIS ou SIGEF conforme a origem do dado.

## Modo Camada QGIS: por que não classifica frente/fundos/laterais?

Essa classificação automática por lado é uma funcionalidade dos produtos comerciais DANI e GEO URBANO. Neste plugin gratuito, o campo de confrontante fica como `[PREENCHER CONFRONTANTE]` para você completar manualmente após a geração.

## Modo Camada QGIS: o memorial não detectou uma curva que existe na geometria

A detecção exige um mínimo de segmentos uniformes (padrão: 100) para reconhecer um arco com confiança. Curvas desenhadas com poucos vértices (ex: um arco simplificado em 5-10 pontos) podem não ser detectadas e aparecerão como uma sequência de retas — isso não afeta a precisão da área/perímetro, só a forma de descrição no texto.

## Modo Camada QGIS: erro/aviso de CRS geográfico

O modo Camada QGIS exige CRS projetado UTM (ex: SIRGAS 2000 / UTM). Se a camada estiver em CRS geográfico (graus, lat/lon), o plugin avisa antes de gerar — reprojete a camada primeiro (Camada → Exportar → Salvar Feições Como → escolha um CRS UTM).

## Modo SIGEF: não aparece confrontante no memorial gerado

Esperado — este modo não extrai nem classifica confrontante, o campo fica como `[PREENCHER CONFRONTANTE]` para preenchimento manual. Classificação automática de confrontantes está disponível no GEO RURAL / DANI.

## Modo SIGEF: erro ao gerar pontos/polígono das parcelas

Ocorre quando o PDF não tem coordenadas em formato DMS reconhecível para algum vértice (ex: campo cortado no PDF). Desmarque "Gerar também pontos e polígono" para gerar só o memorial em DOCX.

## Qual a diferença entre os três modos?

| Modo | Quando usar |
|---|---|
| Texto | Você já tem um memorial pronto (de outro sistema, ou escrito à mão) e quer só gerar a geometria |
| Camada QGIS | Você desenhou o polígono no QGIS e quer gerar o memorial descritivo |
| SIGEF | Você tem o PDF oficial gerado pelo sistema SIGEF/INCRA |
