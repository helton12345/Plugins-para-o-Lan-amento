# Manual de Instruções — GEO URBANO

**Pasta:** `geo_urbano` · versão declarada 1.6.0

---

## 1. Para que serve
Gera documentação de lotes urbanos a partir de um polígono no QGIS:
- memorial **tabular** (Frente/Fundos/Laterais);
- memorial **georreferenciado** (vértices, azimutes, distâncias);
- planta A3;
- camadas estilizadas e cotas;
- requerimentos cartoriais: desmembramento, divisão, retificação e unificação.

Também executa **desmembramento, divisão e unificação** geométricos (aba Operações).

Acesso: botão **GEO URBANO** na barra (sem ícone — ver Descritivo), menu
**Complementos → GEO URBANO** ou, pela Suíte, **📄 Memoriais → GEO URBANO**.
A janela não é modal: dá para mexer no mapa com ela aberta.

> **Use a camada do lote em SRC projetado UTM (SIRGAS 2000).** Todas as
> medidas são calculadas no plano das coordenadas da camada. O plugin **não
> verifica** isso: em SRC geográfico, áreas e distâncias saem em graus.

Na primeira geração de DOCX, se faltar o `python-docx`, o plugin tenta
instalá-lo sozinho (pip).

## 2. Aba 📋 Dados
1. **Camada do Lote (polígono).**
   - Se a camada tiver várias feições, **selecione o lote** antes de processar.
   - Sem seleção, o plugin mostra uma lista para escolher.
2. **Camadas de referência (opcionais):**
   - **Confrontantes (polígonos):** o nome é lido do primeiro campo
     encontrado entre `confrontante`, `proprietario`, `nome`, `name`,
     `logradouro`, `rua`, `matricula`, `descricao`, `obs`…;
   - **Logradouros (linhas/polígonos):** usados para achar a **frente**.
   - A própria camada do lote também é usada para achar os lotes vizinhos.
3. **Dados do Imóvel:** nome, endereço, matrícula(s), município, comarca e
   ART/RRT.
4. **Proprietário(s):**
   - **+ Adicionar** / **✏ Editar** (ou duplo clique) / **− Remover** /
     **🔍 Buscar no banco**.
   - A ficha tem qualificação completa: gênero, estado civil, regime de
     bens, RG, CPF, filiação, cônjuge, fração.
   - Cada pessoa salva vai para o banco `~/GEO_URBANO_DADOS/banco_pessoas.ods`.

## 3. Aba ⚙️ Processamento
1. Deixe marcado **Detectar confrontantes automaticamente**. Desmarque se
   quiser digitar todos os confrontantes à mão.
2. Clique em **⚙️ Processar**. O plugin:
   - calcula a área e o perímetro;
   - separa o contorno em segmentos (retas e curvas);
   - marca como **Frente** o que encosta em **um único** logradouro.
3. **Os segmentos que sobram aparecem um a um**, destacados em amarelo no
   mapa. Para cada um, clique em **Frente / Fundos / Lateral Direita /
   Lateral Esquerda**.
   - Lote com muitos vértices = muitas janelas.
4. Se faltar algum dos 4 lados, o plugin pergunta se deve continuar.
5. **Confrontantes de cada lado:**
   - se a busca achar exatamente **um** confrontante, ele é usado;
   - senão, abre a ficha de confrontante: tipo, nome, CPF/CNPJ, matrícula…
6. Ao terminar:
   - aparece uma **tabela editável** de confrontantes (duplo clique para
     corrigir);
   - a janela vai para a aba **Saída**;
   - se a pasta de saída já estiver escolhida, é feito um backup automático
     da sessão.

## 4. Aba 📁 Saída
1. **Memorial:** Tabular, Georreferenciado ou **os dois** (padrão).
2. **Requerimento (opcional):**
   - **Desmembramento:** informe as áreas dos lotes resultantes, separadas
     por vírgula e com ponto decimal (ex.: `250.00, 180.50`).
   - **Retificação:** informe a área da matrícula e o motivo. O memorial e a
     planta passam a mostrar “Área Registrada” e “Área Medida”.
   - **Unificação:** informe as matrículas a unificar.
3. **Extras:**
   - camadas do imóvel: vértices, limite, polígono, bandeirinhas e nomes
     dos confrontantes, salvas como SHP em `camadas/`;
   - camada de **cotas**;
   - **planta A3**: abre no compositor para revisão; exporte o PDF por lá;
   - shapefile de vértices;
   - **prefixo do vértice** (padrão `V-`) e **subtítulo da planta**.
4. **Pasta de Saída** → **✔ Gerar Documentos**.
   - Os arquivos são nomeados com data e hora (`Memorial_Tabular_AAAAMMDD_HHMMSS.docx` etc.).

## 5. Aba ✂️ Operações
Usa a camada do lote da aba Dados. Para gerar documentos, é preciso ter
escolhido a pasta de saída na aba Saída.

### Desmembramento
- **Cortar por linha:** a camada de linha deve atravessar o lote de lado a lado.
- **Cortar por polígono:** o polígono separa a parte desejada.
- **Só a 1ª feição** da camada de corte é usada.
- Escolha qual parte é o **Desmembrado**. Ver o aviso no Descritivo sobre
  “parte menor/maior”.
- **Matrícula original:** fica no remanescente.
- Opcionalmente, gera os memoriais de cada parte, uma planta
  “SITUAÇÃO FUTURA — DESMEMBRAMENTO” e o requerimento.

### Divisão
- Igual ao desmembramento, mas todas as partes recebem matrícula nova.
- Requerimento de divisão.

### Unificação
1. Selecione um lote no mapa e clique em **+ Adicionar lote selecionado**.
   Repita para cada lote (mínimo 2).
2. **▶ Executar Operação.** Para cada área, os lados e confrontantes são
   processados como na aba Processamento.
3. Resultado:
   - memoriais (tabular + geo) de cada área e da área unificada;
   - planta “Situação Atual” e planta “Situação Futura”;
   - camadas;
   - requerimento de unificação.
4. Os lotes precisam **compartilhar vértices idênticos** (tolerância de 1 mm) na divisa.

## 6. Aba 🧑‍💼 RT
- Nome, título, CREA, credenciamento INCRA, endereço, celular e e-mail.
- Esses dados entram no rodapé dos DOCX, na assinatura e na planta.
- Marque **Salvar RT como padrão** para reutilizar.
- Sem RT salvo, os campos vêm preenchidos com os dados fixos do autor do plugin.

## 7. Sessão
- **💾 Salvar sessão / 📂 Carregar sessão:** guarda até 5 versões em
  `<pasta de saída>/backups/`.
- A sessão guarda **só** os dados do imóvel, os proprietários e a pasta.
- Lados, confrontantes, RT e opções **não** são salvos: é preciso processar
  de novo.

## 8. Conferência obrigatória
Antes de protocolar, confira:
- o CPF dos proprietários no cabeçalho do memorial;
- os confrontantes;
- o quadro da planta: fuso/MC e matrícula dos confrontantes;
- o “/MG” fixo nos textos.

As limitações estão no Descritivo, seção 6.
