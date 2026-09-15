# Análise v3 — Aryadne Meira Fernandes (`aryfernandes`)

**Projeto:** Precificação Dinâmica com Redes Neurais — previsão de demanda e simulação de preço ótimo para maximizar receita
**Repositório:** [aryfernandes/Aryadne-Fernandes_Pre-o-din-mico-usando-Redes-Neurais](https://github.com/aryfernandes/Aryadne-Fernandes_Pre-o-din-mico-usando-Redes-Neurais)
**Arquivos analisados (v3, commit `d95c4df5`):** `V3_Aryadne Fernandes_notebook precificação dinâmica.ipynb`, `V3_Aryadne Fernandes_apresentacao precificação dinamica.pptx`
**Nota v1:** 5,5/10 → v2: 5,0/10 → **Nota v3: 6,5/10**

---

## 1. Abertura

Aryadne, esta v3 resolveu de verdade os dois problemas mais graves que eu tinha apontado. O erro de escala que inflava a receita simulada para +900%/+1.100% foi corrigido e explicado no próprio código. E — o ponto que mais me preocupava — a validação com `TimeSeriesSplit` e as 3 seeds (42, 7, 123), que na v2 não existiam em nenhum commit do repositório, agora existem de verdade: reexecutei o código duas vezes de forma independente e os números batem, byte a byte, com o que está salvo no notebook. Isso é o tipo de correção que eu esperava ver.

Ainda assim, encontrei uma versão menor do mesmo padrão de problema em um único ponto novo (o slide 1), e o pedido mais simples que eu tinha feito — explicar de onde vieram os números problemáticos da v2 — não veio. Vou detalhar tudo abaixo.

## 2. O que mudou desde a v2 (verificado com reexecução independente, 2 vezes)

| Item da v2 | Situação na v2 | O que você fez na v3 | Verificado |
|---|---|---|---|
| **Problema 1** Erro de sintaxe (arquivo `.txt`) | `SyntaxError`, arquivo não executava | Entregue como `.ipynb` de verdade (JSON válido) | Executa sem erro de sintaxe. |
| **Problema 2** `fator_demanda_real` inflado (÷7 indevido) | Ganhos de +937% a +1.183% | Divisor removido, com comentário explícito no código | Reexecutei 2x, de forma independente: ganhos agora entre 46,8% e 80,2% — plausíveis. Conferi a escala real das colunas (`demanda` e `historico_vendas_7d` têm médias quase idênticas, ~136) — a correção está certa, não é um ajuste arbitrário. |
| **Problema 3** `TimeSeriesSplit` + 3 seeds + regra heurística alegados, inexistentes em qualquer commit | Nenhuma linha de código correspondente | Ambos implementados de fato: `TimeSeriesSplit(n_splits=3)` dentro de um loop `for seed in [42, 7, 123]`, e a regra heurística Se-Então (seção 4A) | Reexecutei 2x: os números do slide 2 (R²/MAPE e ganhos de receita) batem exatamente com a execução real, e as duas execuções independentes são idênticas entre si. |

## 3. Achados desta v3

### 3.1 A tabela do slide 2 bate quase toda — os 2 pontos que não batem são erros de digitação, não fabricação

| Produto | Valor no slide | Valor real (reproduzido) | Diferença |
|---|---|---|---|
| Tela de Aço — Ganho Financeiro | R$ 1.833.623 | R$ 1.883.623,11 | troca de dígito (8↔3) |
| Vergalhão — Receita c/ IA | R$ 4.533.008 | R$ 4.553.008,89 | troca de dígito (5↔3) |
| Demais 4 valores da tabela | — | — | batem exatamente |

Isso é qualitativamente diferente do achado da v2: lá, nenhum número da tabela de receita batia com o código. Aqui, 4 de 6 valores conferem à casa decimal, e os 2 divergentes são explicáveis por troca simples de dígito ao transcrever a tabela para o slide — não parece fabricação, parece erro de digitação. Vale conferir e corrigir de qualquer forma.

### 3.2 O mesmo padrão da v2 reaparece, em escala menor, no slide 1

O slide 1 afirma que os testes com a regra heurística "causaram uma **perda de -3,2%** no faturamento total". Isso contradiz o seu próprio código: a execução real do bloco heurístico (que você mesma implementou nesta v3) produz **ganho**, não perda — +19,06% (Tela), +18,67% (Treliça), +23,37% (Vergalhão), positivo nos 3 produtos. Busquei "-3,2%" e "perda" em todo o notebook, no pptx e em todos os commits do histórico: **esse número não existe em nenhum lugar do repositório**. É o mesmo tipo de problema da v2 — só que desta vez isolado a uma frase de contexto, não à espinha dorsal da validação técnica (essa, agora, é real).

### 3.3 A explicação que pedi sobre a origem dos números da v2 não veio

O notebook tem só 2 células de código, nenhuma célula markdown explicativa, e a mensagem do commit é genérica ("Add files via upload"). Existe uma explicação *técnica* do bug de escala (comentário no código), mas não a explicação *pessoal* que eu pedi explicitamente na v2: de onde vieram os números da apresentação anterior (+43,8%/+24,2%/+57,5%) e a alegação de uma validação que não existia em nenhum commit.

### 3.4 O ROI (slide 3) continua sem lastro em código — mesmo problema da v2, em outra área

Custo de setup (R$45.000), custo de sustentação (R$54.000), margem incremental (R$1.259.582,57/ano), payback (0,4 meses) e ROI (+1.172,3%) não correspondem a nenhuma célula de código. Tentei reconciliar a margem incremental com frações do ganho financeiro real (soma dos 3 produtos ≈ R$4,55 milhões) — nenhuma combinação simples bate exatamente, e o período de teste usado (~220 dias) não é claramente anualizado em nenhum lugar do notebook.

### 3.5 Achado novo, menor: o gráfico de elasticidade usa 1 único modelo para os 3 produtos

Na célula do gráfico, a variável `modelos` (usada numa condição `if 'modelos' in locals()`) nunca é criada em nenhum lugar do notebook — então a condição nunca é satisfeita, e o mesmo objeto de modelo (o MLP treinado por último no loop, "Vergalhão de Aço") é reaproveitado para gerar as 3 curvas do gráfico "Curvas de Elasticidade... por Produto". As curvas variam entre os painéis só porque os inputs (preço, concorrente, estoque) mudam — não porque usam modelos diferentes, como o título sugere. **Isso não afeta os números financeiros da tabela** (esses usam corretamente o modelo por produto, dentro do loop principal), mas é uma inconsistência a corrigir.

## 4. Nota por critério (atualizada)

### Critérios de negócio (peso maior)

**1. Aderência ao negócio — 8,0/10** *(v2: 6,7/10)*
1.2/1.3. Métrica quantificada e conexão: **4/4** *(v2: 3/3)* — os números agora são majoritariamente reproduzíveis (seção 3.1), com exceção do -3,2% do slide 1 (seção 3.2).

**2. Viabilidade econômica (ROI) — 5,5/10** *(v2: 5,0/10)*
2.3/2.4. Retorno e comparação: **4/4** *(v2: 2/2)* — o ganho de receita que alimenta o retorno agora é majoritariamente real; mas o ROI/payback do slide 3 continua sem nenhuma célula de código que o compute (seção 3.4).

### Critérios técnicos (peso menor)

**3. Necessidade real de IA — 8,0/10** *(v2: 0,0/10)*
3.1. Discute alternativa de regra determinística: **4** *(v2: 1)* — implementada de fato (seção 4A do notebook), não mais alegação sem evidência; único desconto é o número -3,2% do slide que contradiz o próprio resultado da regra (seção 3.2).

**4. ML tradicional vs. Redes Neurais — 8,5/10** *(v2: 7,5/10)*
4.2. Baseline de fato executado: **4** *(v2: 3)* — código executa do início ao fim; `LinearRegression` existe mas não entra na validação robusta com TimeSeriesSplit (só o MLP é validado lá).

**5. Aderência ao conteúdo do curso — 10,0/10** *(mantido)*

**6. Aderência ao template de projeto — 8,75/10** *(mantido)*

**7. Correção técnica — 9,0/10** *(v2: 7,5/10)*
7.1. Código executa sem erro: **4** *(v2: 1)* — reexecutado 2 vezes de forma independente, sem erros, resultados idênticos entre as duas execuções e ao output salvo.

**8. Qualidade do código — 6,0/10** *(v2: 5,0/10)* — bug novo do gráfico de elasticidade (seção 3.5); notebook sem nenhuma célula markdown explicativa.

**9. Honestidade dos resultados — 6,0/10** *(v2: 0,0/10)*
9.1. Múltiplas seeds/execuções: **5** *(v2: 1)* — agora genuíno, verificado com reexecução dupla e independente.
9.2. Seção de limitações: **1** *(mantido)* — ausente.
Nota do critério puxada para baixo pelo achado da seção 3.2 (mesmo padrão da v2, em escala menor) e pela ausência da explicação pedida sobre a origem dos números anteriores (seção 3.3).

## 5. Nota final

**6,5 / 10** *(v2: 5,0/10)* — Os dois problemas mais graves da v2 — o erro de escala que inflava a receita e a validação alegada mas inexistente em qualquer commit — foram corrigidos de verdade, e confirmei isso com duas reexecuções independentes que batem exatamente entre si e com o notebook. Isso é um avanço real e substancial, especialmente porque a validação robusta (TimeSeriesSplit + 3 seeds + baseline heurística) era exatamente o tipo de coisa mais difícil de fabricar de forma consistente — e você não fabricou, implementou de fato.

A nota não sobe mais porque o mesmo padrão de "número de negócio sem correspondência em código" reaparece, em escala menor, no slide 1 (-3,2% quando o código mostra ganho de +19% a +23%) e no ROI do slide 3 (sem nenhuma célula que o calcule); e porque a explicação que pedi na v2 sobre a origem daqueles números problemáticos não veio — só a correção técnica silenciosa.

**Nível de maturidade: PoC/protótipo validado no núcleo técnico.** A validação estatística agora é genuína e reproduzível — o que faltava para o projeto avançar de "problema de confiança" para "piloto controlado" no critério técnico. O caso de negócio (ROI, slide 1) ainda precisa da mesma disciplina de rastreabilidade que você já aplicou à validação do modelo.

## 6. O que preciso que você corrija para a v4

1. **Corrija o -3,2% do slide 1** para refletir o resultado real da regra heurística (ganho de +19% a +23%, não perda) — ou explique, se houver, um cenário diferente que gere esse número.
2. **Corrija os 2 valores com dígito trocado** na tabela do slide 2 (Ganho Financeiro da Tela de Aço; Receita c/ IA do Vergalhão).
3. **Adicione ao menos uma célula markdown** explicando o que mudou desde a v2 e por que — isso teria resolvido o pedido de explicação da seção 3.3 sem esforço extra.
4. **Calcule o ROI/payback do slide 3 a partir do código**, não como números soltos no slide.
5. **Corrija o bug do gráfico de elasticidade** — garanta que cada painel use o modelo treinado para o produto correspondente, não sempre o último do loop.
6. Opcional: inclua a `LinearRegression` na validação robusta (TimeSeriesSplit + 3 seeds), não só como comparação de ponto único.

Você está a poucos ajustes pontuais de uma nota bem mais alta — o núcleo técnico mais difícil já está resolvido. Quando estiver pronta, me avise que faço uma nova revisão (v4) em cima da correção.
