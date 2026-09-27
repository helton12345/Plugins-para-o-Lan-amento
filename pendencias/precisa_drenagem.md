# Pendências — Precisa Drenagem

Versão atual entregue: **0.9.9**.
Das 14 pendências anotadas na 0.9.4, **13 foram implementadas** (0.9.7 a 0.9.9). A que falta (teste real do Whitebox) depende do QGIS do usuário. Abaixo estão o histórico, o que continua aberto e o que conferir no QGIS.

## Histórico das versões desta revisão

| Versão | O que entrou |
|---|---|
| 0.9.2 | Trecho sem área (S de 50 %) corrigido; memória: Q própria em L/s com C do trecho, SOBRECARGA/VERIFICAR, município e empreendimento pedidos, parâmetros do último cálculo, textos do método; Whitebox com sub-bacias incrementais e vínculo sem dupla contagem; campo prof. máx. da vala; aviso de cruzamentos fora das pontas |
| 0.9.3 | Declividade mínima construtiva editável, padrão 0,5 % (FCTH/CDren); alertas em dois níveis (erro/aviso) |
| 0.9.4 | Método Racional com tc acumulado por trecho (tc de entrada 10 min); opção de tc único para reproduzir projetos antigos |
| 0.9.5 | Tabela editável: DN fixo por trecho; Área e C gravados na camada de rede; recálculo a cada edição |
| 0.9.6 | Recalcular atualiza no lugar as camadas drenagem_galerias e drenagem_PVs (mantém estilo, rótulos e arquivo) |
| 0.9.7 | Área do lote lida em m² e gravada em AREA_HA; lotes que drenam para o fundo somados à frente (opção); lotes/rede em SRC diferente param; nó sem cota interrompe o cálculo; terreno amostrado no SRC certo; seta do sentido do fluxo; salvar/carregar projeto (.json com resultado); ícone; cores DXF DN 800–1500; typing_extensions de reserva |
| 0.9.8 | Declividade de V mín limitada pela vala (aviso "V < V mín"); C de Horner; vazão pontual no nó; cota de fundo fixa por trecho; inverter sentido (ordem topológica); ponto de deságue no PV vai para o trecho que sai do nó |
| 0.9.9 | Catálogo IDF com 158 equações (7 formas do CDren/FCTH); verificação de sarjetas (Izzard + fator DAEE/CETESB) e contagem de bocas de lobo; quantitativos (CSV); dividir rede nos cruzamentos; ajuste dos pontos ao talvegue (Whitebox); tela não congela na delineação |

## Pendências abertas

### Precisa do QGIS do usuário (não dá para testar no ambiente de desenvolvimento)
**1. Teste real da delineação Whitebox**
- Nada disso rodou com o Whitebox real: o download do binário foi bloqueado no ambiente de teste. Só houve simulação.
- Conferir no QGIS:
  - cada sub-bacia recebe o número do seu ponto (1, 2, 3…);
  - o ajuste ao talvegue (`snap_pour_points`, raio de 5 m) move os pontos para o lugar certo;
  - a tela continua respondendo durante o processamento.
- Usar os DXFs de exemplo do pacote do CDren (ruas, curvas, pluvial pontos).

**2. Roteiro de conferência da 0.9.9 no QGIS**
- O diálogo foi testado num QGIS simulado (32 verificações).
- Falta conferir no real:
  - desenho da tela e colunas novas;
  - seta do fluxo;
  - menu do botão direito;
  - botões "Salvar/Carregar projeto", "Escolher IDF da cidade", "Exportar Quantitativos" e "Dividir rede nos cruzamentos";
  - camadas GeoPackage (campo `fid`).

### Melhorias possíveis (não pedidas)
**3. Preços (SINAPI) nos quantitativos.** Hoje o CSV traz só as quantidades; o orçamento depende da tabela de referência do usuário.

**4. Parâmetros de sarjeta na tela.** Altura da lâmina (0,15 m), tg θ (12), n (0,016) e capacidade das bocas de lobo (40/60 L/s) estão fixos no código, com os padrões do CDren. Só a verificação liga/desliga pela tela.

**5. Largura de vala para PVC (RibLoc).** Hoje só existe a tabela de concreto. O manual traz também a fórmula máx(D + 0,40; 1,25·D + 0,30).

**6. Tempo de concentração inicial por Kerby/George Ribeiro.** O CDren oferece essas fórmulas; hoje o tc de entrada é digitado.

**7. `metadata.txt`.**
- `tracker` e `repository` ainda apontam para `example.invalid`: faltam os endereços reais.
- O texto de `about` não descreve as funções novas.

**8. Equação IDF tipo 10 (Santo André, SEMASA).** Não entrou no catálogo: o manual não traz a fórmula.

## Critérios e fontes usados (para revisão técnica)
- **Horner:** C = 0,364·log(t) + 0,0042·P − 0,145, mínimo 0,05 (tela do CDren).
- **Izzard:** Q0 = 0,375·(tg θ/n)·y^(8/3)·S^(1/2). Fator F (DAEE/CETESB): 0,50 (≤ 0,4 %), 0,80 (1–3 %), 0,50 (5 %), 0,40 (6 %), 0,27 (8 %), 0,20 (≥ 10 %).
- **Bocas de lobo:** 40 L/s em declive ≥ 1 % e 60 L/s em rua plana (limites inferiores das faixas do manual do CDren).
- **Largura de vala:** tabela do manual do CDren para tubos de concreto. Escoramento acima de 1,25 m (NR-18).
- **IDF:** fórmulas das páginas "Equações IDF" do manual do CDren. α de Pfafstetter pela tabela universal de "Chuvas Intensas no Brasil".
