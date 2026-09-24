# Manual de Instruções — Precisa Earth Engine

**Pasta:** `precisa_earth_engine` · versão declarada 1.0.0 (experimental) · QGIS 3.28+

---

## 1. Para que serve

Baixa imagens de satélite **recortadas pelo polígono da sua área** (com buffer
opcional), em um período de datas, e carrega no QGIS já estilizadas. Três fontes:

| Fonte | O que entrega | De onde vem |
|---|---|---|
| **Dynamic World V1** | Mapa de uso do solo (9 classes) — classe mais frequente no período | Google Earth Engine |
| **Sentinel-2** | Imagem colorida (RGB) — mediana do período, só cenas com ≤ 35 % de nuvens | Google Earth Engine |
| **CBERS-4A WPM pancromático** | Uma cena pancromática de alta resolução (a de menor nuvem do período) | INPE / Brazil Data Cube |

## 2. Preparação (uma vez)

### 2.1 Instalar bibliotecas no Python do QGIS
No **OSGeo4W Shell** (Windows) ou no terminal do Python do QGIS:
```
pip install earthengine-api requests
```
Reinicie o QGIS.

### 2.2 Credenciais do Google Earth Engine (para Dynamic World e Sentinel-2)
1. No Google Cloud, crie uma *Service Account* com acesso ao Earth Engine e
   baixe a chave **JSON**. Registre a conta no Earth Engine, se ainda não tiver.
2. No plugin, clique em **Configurar credenciais**:
   - **Procurar…** → escolha o arquivo `.json` (o e-mail da conta aparece
     automaticamente);
   - informe o **Project ID** do Google Cloud;
   - clique em **Testar conexão** (fica verde se deu certo) e depois **Salvar**.

As credenciais ficam gravadas só no seu computador (configurações do QGIS).

### 2.3 Token do Brazil Data Cube (só para CBERS-4A)
Crie conta gratuita em `brazildatacube.dpi.inpe.br`, gere um *Personal Access
Token* e cole no campo **Token BDC** do plugin.

## 3. Como abrir
Ícone na barra de ferramentas ou menu **Complementos → Precisa Earth Engine**.

## 4. Passo a passo

1. **Carregue no QGIS a camada de polígono** da área de interesse.
2. **Camada de polígono:** selecione-a na lista. Todas as feições da camada
   são unidas numa única área.
3. **Buffer (m):** amplia (positivo) ou reduz (negativo) a área. 0 = sem buffer.
4. **Período (De / Até):** padrão = últimos 3 meses.
5. **Resolução de exportação:** deixe em *Padrão da fonte* (10 m) ou informe
   outro valor em metros (até 1000 m). Não se aplica ao CBERS-4A.
6. **Fontes de dados:** marque uma ou mais.
   - Se marcar **CBERS-4A**, preencha o **Token BDC**, clique em **Atualizar**
     e escolha a **Coleção INPE** na lista.
7. **Modo de saída:**
   - **Baixar GeoTIFF** → escolha a pasta; os arquivos são salvos como
     `<Fonte>_<data inicial>_<data final>.tif`.
   - **Streaming (tiles)** → carrega direto do Earth Engine sem salvar arquivo.
     O CBERS-4A **sempre é baixado** (vai para a pasta temporária do sistema
     neste modo) — o plugin avisa na barra de mensagens.
8. Clique em **Processar**. O QGIS continua livre enquanto o processamento
   roda; a barra de progresso avança a cada fonte concluída.
9. Ao terminar, as camadas aparecem no projeto já estilizadas:
   - Dynamic World com a legenda oficial (Água, Árvores, Grama, Vegetação
     alagada, Plantações, Arbustos, Construído, Solo exposto, Neve/gelo);
   - Sentinel-2 em cor natural (realce 0–3000);
   - CBERS-4A em tons de cinza (realce mínimo–máximo).

## 5. Mensagens comuns

| Mensagem | O que fazer |
|---|---|
| “Credenciais do Earth Engine não configuradas” | Botão **Configurar credenciais** |
| “A biblioteca 'earthengine-api' não está instalada” | Ver item 2.1 |
| “Selecione uma coleção CBERS-4A WPM” | Informe o token e clique em **Atualizar** |
| “O buffer aplicado resultou em uma geometria vazia” | Buffer negativo maior que a área — reduza |
| “Nenhuma cena CBERS-4A WPM encontrada…” | Amplie o período ou troque a coleção |
| “Falha ao autenticar no Google Earth Engine” | Confira JSON, Project ID e o registro da conta no Earth Engine |

## 6. Dicas
- Para áreas grandes, prefira **Streaming** ou aumente a resolução (ex.: 30 m):
  o download direto do Earth Engine tem limite de tamanho por arquivo.
- Camadas em streaming dependem de um link temporário do Earth Engine; ao
  reabrir o projeto dias depois, pode ser preciso processar de novo.
- A data final digitada não entra no filtro do Earth Engine (o intervalo é
  “até, sem incluir” o dia final) — acrescente um dia se precisar dele.
