# Análise v2 — Aryadne Meira Fernandes (`aryfernandes`)

**Projeto:** Precificação Dinâmica com Redes Neurais — previsão de demanda e simulação de preço ótimo para maximizar receita
**Repositório:** [aryfernandes/Aryadne-Fernandes_Pre-o-din-mico-usando-Redes-Neurais](https://github.com/aryfernandes/Aryadne-Fernandes_Pre-o-din-mico-usando-Redes-Neurais)
**Arquivos analisados (v2):** `REVISÃO_Aryande Fernandes_notebook_precificacao_dinamica`, `REVISÃO_Aryadne Fernandes_apresentacao precificação dinamica.pptx`
**Nota v1:** 5,5/10 → **Nota v2: 5,0/10**

---

## 1. Abertura

Aryadne, obrigado por revisar o projeto. Você corrigiu de verdade os dois bugs que eu tinha apontado na v1 (o `best_estimator__` e o `demande_real_mlp`) — isso mostra que você entendeu exatamente onde o código quebrava, e os comentários "CORREÇÃO DO BUG 1/2" deixam isso bem claro. O problema é que, ao reexecutar o notebook do zero para conferir os novos números, encontrei um conjunto de inconsistências entre o que está no código entregue e o que está na apresentação — grande o suficiente para eu precisar te pedir uma explicação antes de fechar a nota. Vou detalhar tudo com precisão abaixo, porque acredito que isso é mais útil para você do que eu simplesmente atribuir uma nota baixa sem dizer exatamente o que encontrei.

## 2. O que mudou desde a v1 (verificado de forma independente, com reexecução do notebook)

| Item da v1 | Situação na v1 | O que você fez na v2 | Verificado |
|---|---|---|---|
| **7.1** `grid_search.best_estimator__` (bug) | `AttributeError`, quebrava a Seção 3 na primeira iteração | Corrigido para `best_estimator_` (comentário "CORREÇÃO DO BUG 1") | Corrigido corretamente — confirmei lendo o código. |
| **7.1** `demande_real_mlp` (bug) | `NameError`, quebrava a Seção 4 | Corrigido para `demanda_real_mlp` (comentário "CORREÇÃO DO BUG 2") | Corrigido corretamente. |

Essas duas correções são reais e eu quero deixar isso registrado — mas, ao tentar rodar o notebook corrigido do início ao fim para conferir os novos números, encontrei três problemas novos, descritos abaixo.

## 3. Problema 1 — o arquivo entregue ainda não roda (erro de sintaxe novo)

O arquivo `REVISÃO_..._notebook_precificacao_dinamica`, como está no repositório, não é Python válido. Em três pontos (próximo às linhas 85, 144 e 208) há uma quebra de linha real dentro de uma string, onde deveria haver um `\n` escapado — por exemplo, a linha `print(f'` continua na linha seguinte, em vez de `print(f'\nOtimizando modelos para: {prod}...')` como estava, corretamente, na v1. Isso gera `SyntaxError: unterminated f-string literal` e impede que o script rode do início ao fim como está.

Reexecutei o arquivo duas vezes de forma independente (eu e, depois, uma segunda verificação própria, sem reaproveitar a primeira correção) apenas juntando essas três linhas de volta com `\n` escapado — sem alterar nenhuma lógica — e as duas vezes reproduzi exatamente os mesmos números. Ou seja, o problema é isolado e não é uma questão de ambiente: é o arquivo que foi commitado que tem esse defeito de formatação, provavelmente introduzido por algum processo de edição/exportação que converteu `\n` escapado em quebra de linha real.

## 4. Problema 2 — a simulação de receita, mesmo com o bug de sintaxe corrigido, produz números absurdos

Depois de corrigir a sintaxe, rodei a Seção 4 (simulação de receita) e obtive ganhos de **+1.183% (Vergalhão), +1.060% (Tela) e +937% (Treliça)** sobre a receita histórica — uma ordem de grandeza claramente irreal, tanto para o modelo Linear quanto para o MLP.

Encontrei a causa técnica exata: a linha `fator_demanda_real = row['demanda'] / (row['historico_vendas_7d'] / 7.0 + 1e-5)` pressupõe que `historico_vendas_7d` é uma soma de 7 dias (por isso a divisão por 7 para virar uma taxa diária). Mas nos dados, a razão média `demanda / historico_vendas_7d` já é ≈1,0 para os três produtos — ou seja, essa coluna já está na mesma escala diária de `demanda`, não é uma soma bruta. Dividir por 7 encolhe o denominador artificialmente, inflando `fator_demanda_real` para uma média de ~7x (com picos de até 20x), o que explica a ordem de grandeza dos ganhos. Isso não parece intencional — é um erro genuíno de interpretação de uma coluna do dataset —, mas significa que o arquivo, mesmo corrigido no que diz respeito aos dois bugs originais, ainda não produz um resultado de negócio confiável.

## 5. Problema 3 (o mais sério) — a apresentação descreve uma metodologia e resultados que não encontrei em nenhum lugar do código, em nenhuma versão do repositório

Este é o ponto que mais preciso que você esclareça.

O slide 2 da apresentação v2 afirma: *"Validação com TimeSeriesSplit e 3 Sementes Aleatórias (Seeds: 42, 7, 123) em 3 Anos de Histórico"*, com R² de 0,98-0,99 (±0,001) e ganhos de receita de +43,8% / +24,2% / +57,5% por produto. O slide 1 afirma ter testado uma "Regra Heurística Se-Então" (desconto de 5% se estoque alto, aumento de 4% se estoque baixo) como alternativa simples, com resultado de "-3,2% no faturamento".

Busquei `TimeSeriesSplit`, as seeds `7` e `123`, e qualquer lógica de desconto/aumento por estoque em **todos os 6 arquivos do repositório e nos 3 commits do histórico** — a única seed usada em qualquer lugar, em qualquer versão, é `random_state=42`, uma única vez. Não existe, em nenhum artefato entregue, nenhuma implementação da regra heurística nem de uma validação com múltiplas seeds ou `TimeSeriesSplit`.

Uma observação importante, para ser justo com o que encontrei: os valores de R²/MAPE do slide 2 batem exatamente com uma execução real, de seed única (42), do código entregue — essa parte é genuína. E os números de "Receita Histórica" do slide também batem, ao centavo, com o que o código produz. Já os números de "Receita com IA" (que geram os +43,8%/+24,2%/+57,5%) não batem com nenhuma variação do código entregue — mas, testando uma forma de calcular a receita sem o erro de escala descrito no Problema 2, cheguei a um número quase idêntico ao do slide para o Vergalhão (diferença de R$1.228, ou 0,03%). Isso sugere que esses números provavelmente vieram da execução real de **uma versão do código diferente da que foi commitada neste repositório** — não parecem inventados do zero — mas essa versão nunca chegou até mim.

Preciso que você me explique a origem desses números e da alegação de validação com 3 seeds e `TimeSeriesSplit`: se existe uma versão do notebook (talvez rodada localmente ou no Colab) que não foi commitada, ou se houve algum engano na hora de escrever os slides. De qualquer forma, o material entregue precisa refletir exatamente o que foi executado — é isso que garante que a nota (e a decisão de negócio que ela embasaria) esteja apoiada em algo real.

## 6. Nota por critério (atualizada)

### Critérios de negócio (peso maior)

**1. Aderência ao negócio — 6,7/10** *(v1: 10,0/10)*
1.1. Métrica de sucesso nomeada como receita/custo: **5** *(mantido)* — "maximizar receita" continua explícito.
1.2. Métrica quantificada: **3** *(v1: 5)* — os números apresentados (+43,8%/+24,2%/+57,5%) não são reproduzíveis a partir do código entregue (ver seção 5); não posso validar a quantificação apresentada como está.
1.3. Conexão entre métrica técnica e impacto de negócio: **3** *(v1: 5)* — mesma ressalva: a conexão é bem argumentada narrativamente, mas os números específicos usados para fazer essa conexão não estão sustentados pelo artefato entregue.

**2. Viabilidade econômica (ROI) — 5,0/10** *(v1: 2,5/10)*
2.1. Custo de construção estimado: **4** *(v1: 1)* — R$45.000 (2 meses de squad + infraestrutura) é uma estimativa presente e razoável, mesmo sem detalhamento de premissas.
2.2. Custo de sustentação estimado: **4** *(v1: 1)* — R$54.000/ano (R$4.500/mês) também presente e razoável.
2.3. Retorno esperado com número: **2** *(v1: 5)* — o retorno depende diretamente do "Ganho de Receita com IA", que é o número não reproduzível descrito na seção 5.
2.4. Comparação custo vs. retorno: **2** *(v1: 1)* — agora existe (ROI%, payback), o que é um avanço real de estrutura, mas herda o mesmo problema de base do item 2.3.

### Critérios técnicos (peso menor)

**3. Necessidade real de IA — 0,0/10** *(mantido, v1: 0,0/10)*
3.1. Discute alternativa de automação/regra determinística: **1** *(mantido)* — o slide 1 afirma ter testado uma regra heurística e obtido "-3,2% no faturamento", mas essa implementação não existe em nenhum lugar do código entregue — uma alegação sem evidência recebe a mesma nota mínima que a ausência completa da discussão.

**4. ML tradicional vs. Redes Neurais — 7,5/10** *(v1: 6,3/10)*
4.1. Compara explicitamente contra ML tradicional: **5** *(mantido)* — `LinearRegression` continua implementada e comparada.
4.2. Baseline simples de fato executado e comparado: **3** *(v1: 2)* — os dois bugs de lógica que impediam essa comparação de rodar foram corrigidos corretamente; a nota não sobe mais porque o arquivo, como entregue, ainda não executa do início ao fim por causa do problema de sintaxe da seção 3.

**5. Aderência ao conteúdo do curso — 10,0/10** *(mantido)*

**6. Aderência ao template de projeto — 8,75/10** *(v1: 6,3/10)*
6.2. Profundidade do bloco 7 (MLOps): **4** *(v1: 2)* — o slide 3 agora define cadência de retreino (quinzenal, janela móvel de 180 dias), limiares numéricos de monitoramento (PSI>0,15 nos preços do concorrente, MAPE>12%) e uma política de rollback automático para a baseline linear se o MAPE>18% por 3 dias — um avanço real e bem pensado em relação à v1, mesmo sendo uma seção só narrativa (sem código associado, o que é esperado nesse bloco).

**7. Correção técnica — 7,5/10** *(mantido, v1: 7,5/10)*
7.1. Código executa do início ao fim sem erro: **1** *(mantido)* — os dois bugs de lógica da v1 foram corrigidos, mas um novo problema de sintaxe (seção 3 acima) significa que o arquivo entregue continua não executando do início ao fim como está.
7.2/7.3/7.4: **5/5/5** *(mantidos)* — a estrutura de split, métricas e comparação no mesmo split continua correta assim que o código consegue rodar.

**8. Qualidade do código — 5,0/10** *(mantido)*

**9. Honestidade dos resultados — 0,0/10** *(mantido, v1: 0,0/10— mas por um motivo mais grave)*
9.1. Múltiplas seeds/execuções: **1** — a v1 já estava no piso por não ter testado robustez; a v2 chega ao mesmo piso por um motivo mais sério: descreve uma validação com 3 seeds e `TimeSeriesSplit` que não está em nenhum lugar do código entregue (ver seção 5). Reportar uma validação de robustez que não foi executada é mais grave do que simplesmente não ter feito a validação.
9.2. Seção de limitações presente: **1** *(mantido)* — continua ausente.

## 7. Nota final

**5,0 / 10** *(v1: 5,5/10)* — Você corrigiu de verdade os dois bugs de lógica que eu tinha apontado, e o bloco de MLOps ficou bem mais concreto (limiares numéricos de drift, política de rollback) — isso é progresso real e quero que fique registrado. Mas a nota não sobe, e cai ligeiramente, porque a verificação encontrou um problema novo e mais sério do que os bugs da v1: a apresentação descreve uma validação com múltiplas seeds e `TimeSeriesSplit`, e um teste de baseline por regra heurística, que não existem em nenhum artefato entregue, em nenhuma versão do repositório — e os números de ganho de receita que sustentam a seção de ROI não são reproduzíveis a partir do código que você me enviou (embora pareçam ter vindo de uma execução real de outra versão do código, não inventados do zero). Antes de eu poder validar essa parte do trabalho, preciso entender a origem desses números.

**Nível de maturidade: PoC/protótipo.** O MLOps e o ROI agora têm estrutura de negócio real, mas a validade técnica dos números que os sustentam ainda não está estabelecida — o código entregue não roda do início ao fim, e a parte de "receita com IA" precisa ser reconciliada com uma execução verificável antes de qualquer avanço de maturidade.

## 8. O que preciso que você corrija para a v3

1. **Me explique a origem dos números de receita do slide 2** (+43,8%/+24,2%/+57,5%) e da alegação de validação com `TimeSeriesSplit` e 3 seeds (42, 7, 123) do mesmo slide — existe uma versão do notebook que gerou esses números e que não foi commitada? Se sim, é essa versão que preciso receber.
2. **Corrija o erro de sintaxe** nos 3 pontos onde há quebra de linha literal dentro de string (próximo às linhas 85, 144 e 208) — troque por `\n` escapado, como estava corretamente na v1.
3. **Corrija o erro de escala em `fator_demanda_real`** (Seção 4) — a divisão por 7 só faz sentido se `historico_vendas_7d` for uma soma de 7 dias; confirme a definição real dessa coluna e ajuste a fórmula, já que hoje ela infla a receita simulada para uma ordem de grandeza irreal (+900% a +1.100%).
4. **Rode o notebook do início ao fim, com kernel limpo, antes de gerar os slides** — os dois problemas acima (sintaxe e escala) só apareceram porque o arquivo nunca foi executado ponta a ponta depois da correção dos bugs originais. Salvar como `.ipynb` com os outputs (em vez de `.txt`) ajudaria a evitar esse tipo de discrepância entre código e apresentação no futuro.
5. **Implemente de fato a regra heurística por estoque** (ou remova a alegação do slide 1) — se a comparação com uma regra simples é um ponto forte que você quer manter na apresentação, ela precisa existir no notebook.
6. **Adicione uma seção de limitações** — reconhecendo o corpus/período de dados, a ausência de validação com múltiplas seeds (ou, quando implementada de fato, reportando o resultado real dela) e a faixa de extrapolação de preço testada na simulação.

Quando estiver pronta, me avise que faço uma nova revisão (v3) em cima da correção — o caminho para uma nota bem mais alta está bem claro: a moldura de negócio que você construiu (ROI, MLOps, conexão com receita) já é uma das mais completas da turma; falta só que os números por trás dela sejam os que o código de fato produz.
