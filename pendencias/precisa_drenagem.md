# Pendências — Precisa Drenagem

Versão atual entregue: **0.9.4** (tc acumulado por trecho).
Nada desta lista foi implementado. Cada item traz a causa já diagnosticada e a correção proposta.

## Histórico das versões desta revisão

| Versão | O que entrou |
|---|---|
| 0.9.2 | Trecho sem área (S de 50 %) corrigido; memória: Q própria em L/s com C do trecho, SOBRECARGA/VERIFICAR, município e empreendimento pedidos, parâmetros do último cálculo, textos do método; Whitebox com sub-bacias incrementais e vínculo sem dupla contagem; campo prof. máx. da vala; aviso de cruzamentos fora das pontas |
| 0.9.3 | Declividade mínima construtiva editável, padrão 0,5 % (FCTH/CDren); alertas em dois níveis (erro/aviso) |
| 0.9.4 | Método Racional com tc acumulado por trecho (tc de entrada 10 min); opção de tc único para reproduzir projetos antigos |

## Pendências abertas

### Alta prioridade

**1. Área do lote em m² tratada como ha (Individualizar por Lotes)**
- **Sintoma:** tubos enormes e em SOBRECARGA depois de "Individualizar por Lotes".
- **Causa:** `drenagem_dialog.py`, `individualizar_bacias_lote_ui` (linhas 834-841 na 0.9.4). O valor do "Campo de área do lote (opcional)" é usado direto como ha, sem conversão. Com o campo em m², cada lote entra 10.000 vezes maior. Sem o campo, a área vem da geometria dividida por 10.000 e sai certa.
- **Contorno até corrigir:** deixar o campo vazio, rodar "Individualizar por Lotes" de novo (sobrescreve a área da rede) e depois "Calcular".
- **Correção decidida:** tirar o campo de área do lote OU converter o valor de m² para ha antes de somar. A escolher na implementação.
- **Pode quebrar:** quem usa um campo que já está em ha.

**2. Delineação Whitebox não foi testada com o Whitebox real (0.9.2)**
- Só foi testada com simulação: o download do binário foi bloqueado no ambiente de teste.
- Conferir no QGIS: cada sub-bacia deve receber o número do seu ponto na ordem da camada (1, 2, 3…).
- Usar os DXFs de exemplo do pacote do CDren (ruas, curvas, pluvial pontos).

### Média prioridade

**3. Salvar e carregar projeto**
- Hoje nada fica gravado: camadas, campos, IDF e parâmetros voltam ao padrão sempre que o diálogo abre.
- **Proposta:** botões "Salvar projeto…" e "Carregar projeto…" com um `.json` de parâmetros e nomes de camadas e campos. Só em `drenagem_dialog.py`.
- **A confirmar:** se deve salvar também o resultado do cálculo.

**4. Seta com o sentido do fluxo na camada `drenagem_galerias`**
- **Causa:** a camada copia a geometria na ordem em que foi desenhada (`_gerar_camadas`); o sentido do fluxo vem depois, da topologia, pela cota.
- **Proposta:** inverter a geometria de saída quando `mapa[idx][0] != no_mont` e aplicar simbologia com seta no meio da linha. A rede original não é alterada.
- **Pode quebrar:** nada no cálculo. Perfil e planta DXF usam `no_mont`/`no_jus`, não a geometria.

**5. Área pequena ainda gera declividade alta**
- É o que ficou do item 1 da revisão (opção "b", não aprovada). Exemplo: 0,001 ha dá S = 13,8 % no DN400 com V mín de 0,75 m/s.
- **Proposta:** limitar a declividade de V mín ao ponto em que a vala chega à profundidade máxima, com o alerta "V < V mín". Alternativa: usar V mín de 0,50 m/s (faixa do CDren).

**6. Ponto de deságue em PV entre dois trechos (vínculo Whitebox)**
- Hoje vai para o primeiro trecho da camada, que pode ser o de chegada. O total a jusante fica certo.
- **Ideal:** mandar para o trecho que sai do nó, o que exige conhecer o sentido da rede no momento do vínculo.

**7. Pontos de deságue sem ajuste ao talvegue**
- Falta `snap_pour_points` no Whitebox. Um ponto fora do talvegue pode gerar uma bacia minúscula.

### Baixa prioridade / melhorias (do manual do CDren/FCTH)

**8. Biblioteca IDF por estado**
- São 159 equações no pacote do CDren. 11 têm a mesma forma do plugin (tipo 1); 97 são de Pfafstetter e cerca de 40 são LnLn ou DAEE, que exigem fórmulas novas em `chuva.py`.
- Citar sempre a fonte de cada equação.

**9. Coeficiente C pela fórmula de Horner**, a partir do percentual impermeável, com C mínimo de 0,05.

**10. Vazão pontual no nó e opções "fixa diâmetro" / "fixa cota" por trecho**
- Hoje não há edição manual: a tabela é só de leitura e não dá para inverter o sentido.

**11. Verificação de sarjeta e rua e número de bocas de lobo**
- Capacidade da via por classe; bocas de 40–45 L/s em declive e de 60–65 L/s em rua plana. É uma função nova.

**12. Quantitativos e orçamento**
- Largura de vala pelo maior entre D + 0,40 e 1,25·D + 0,30, escavação, empolamento. Preços pelo SINAPI.

**13. Segmentação real nos cruzamentos**
- Hoje o plugin só avisa (0.9.2). Segmentar exige repartir o campo de área entre os pedaços.

**14. Outros pontos do DESCRITIVO ainda abertos**
- Nó fora do MDT recebe cota 0.
- O raster é amostrado sem conferir o SRC.
- Lotes reprovados ficam fora de qualquer trecho.
- A delineação roda na thread principal e congela o QGIS.
- Faltam cores próprias para DN 800–1500 na planta DXF.
- Falta ícone na barra de ferramentas.
- O python-docx embarcado precisa de `typing_extensions`, que não vem junto.
- `metadata.txt`: tracker e repository apontam para `example.invalid`.
