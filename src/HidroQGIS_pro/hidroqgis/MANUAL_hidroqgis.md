# Manual de Instruções — HidroQGIS

**Pasta:** `hidroqgis` · versão declarada 1.1.0

---

## 1. Para que serve

A partir de um MDE (pensado para o TOPODATA/INPE, 30 m), o plugin gera
automaticamente a **rede de drenagem**, as **sub-bacias** e as áreas de
**acúmulo de fluxo** (“corpos d'água”). Opcionalmente, também produz planta em
PDF, memorial em DOCX, KMZ, visualizador web e comparação entre duas datas.

Acesso: botão **HidroQGIS** na barra, menu **Complementos → HidroQGIS** ou,
pela Suíte, **📁 Projetos → HidroQGIS**.

## 2. Primeira execução — dependências

Se faltar alguma biblioteca, o plugin **pergunta** se pode instalar
(`pip install --user`):

| Biblioteca | Uso |
|---|---|
| `whitebox` | Processamento hidrológico (obrigatória) |
| `python-docx` | Memorial DOCX |
| `pystac-client` | Download do MDE TOPODATA |

Na primeira execução, o WhiteboxTools pode baixar o próprio executável.

## 3. Grupo “Dados de Entrada”

1. **MDE TOPODATA (.tif):** escolha o raster **ou** use
   **⬇ Baixar MDE (TOPODATA/INPE) para a extensão atual do mapa**. Nesse caso:
   - enquadre antes a área no canvas;
   - o plugin baixa as folhas da coleção `topodata-1` (BDC/INPE), junta tudo
     num mosaico e preenche o campo.
   - A tela fica travada durante o download.
2. **Diretório de saída** e **Nome do projeto/bacia** (usado nos títulos).
3. **MDE de comparação (opcional):** MDE de data anterior, usado só se a
   comparação temporal for marcada.
4. **Limite da área de estudo (opcional):** camada de polígono. Marque
   **Recortar o MDE ao limite…** para processar só essa área (mais rápido).

## 4. Grupo “Parâmetros de Processamento”

| Campo | Padrão | Efeito |
|---|---|---|
| Limiar de drenagem (células) | 1000 | Menor → rede mais densa |
| Limiar de corpos d'água (células) | 10000 | **Tem de ser maior** que o limiar de drenagem |
| Área mínima de corpos d'água (m²) | 10000 (1 ha) | Descarta polígonos menores |

## 5. Grupo “Produtos de Saída”

- **Planta PDF**, com escolha do tamanho da folha (A4 a A0; padrão A3):
  - conteúdo: relevo sombreado, hipsometria, curvas de nível, drenagem,
    sub-bacias e croqui de localização;
  - o compositor de impressão também é aberto para ajustes.
- **Memorial DOCX:** metodologia, drenagem, corpos d'água, relevo e
  tabela morfométrica por sub-bacia (área, perímetro, Kc, Dd).
- **KMZ** para o Google Earth.
- **Comparação temporal** (exige o MDE de comparação).
- **Publicar visualizador web (Netlify):** ao final, abre uma janela para:
  - informar a API key do Netlify (e, se quiser, salvá-la), **ou**
  - **salvar o `index.html` localmente**.

## 6. Executar

1. **▶ Executar Processamento.** A barra de progresso mostra as etapas,
   nesta ordem:
   1. recorte;
   2. preenchimento de depressões;
   3. direção de fluxo;
   4. acumulação;
   5. drenagem;
   6. sub-bacias;
   7. corpos d'água;
   8. exportação.
2. **✕ Cancelar** interrompe a tarefa.
3. Ao terminar:
   - as camadas entram no grupo **“HidroQGIS — Resultados”**;
   - os vetores ficam em `HidroQGIS_resultados.gpkg`, no diretório de saída;
   - os produtos marcados são gerados em seguida.

### Saídas principais
- Camadas de **drenagem** (linhas), **sub-bacias** (campo `id_bacia`) e
  **corpos d'água** (polígonos).
- Rasters intermediários no diretório de saída (MDE preenchido, direção
  e acumulação de fluxo).
- Planta (`.pdf`), memorial (`.docx`) e `.kmz`, conforme as opções marcadas.
- Comparação temporal:
  - pasta `comparacao_temporal_mde_anterior/`;
  - arquivo `comparacao_temporal.shp`;
  - seção 6 no memorial.

## 7. Cuidados
- **Os “corpos d'água” são áreas de alto fluxo acumulado**, não espelhos
  d'água mapeados. Confira com imagem.
- A planta e o memorial trazem **“SIRGAS 2000 (EPSG:4674)” e “Fonte:
  TOPODATA/INPE” fixos**. Se usar outro MDE/SRC, corrija no compositor e no
  DOCX.
- Para a planta e o memorial, prefira um MDE em SRC projetado (UTM):
  - as áreas/perímetros usam as medidas do QGIS;
  - veja o Descritivo.
- Publicação Netlify: veja a limitação sobre o arquivo `_headers` no Descritivo.
