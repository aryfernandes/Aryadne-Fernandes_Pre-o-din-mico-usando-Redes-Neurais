# Análise — Aryadne Meira Fernandes (`aryfernandes`)

**Projeto:** Precificação Dinâmica com Redes Neurais — previsão de demanda e simulação de preço ótimo para maximizar receita
**Repositório:** [aryfernandes/Aryadne-Fernandes_Pre-o-din-mico-usando-Redes-Neurais](https://github.com/aryfernandes/Aryadne-Fernandes_Pre-o-din-mico-usando-Redes-Neurais)
**Arquivos analisados:** `Aryadne Fernandes_notebook_precificação dinâmica.txt` (código-fonte, não é `.ipynb` — sem outputs salvos), `Aryadne Fernandes_Apresentação trabalho Redes Neurais.pptx`, `VF_Aryadne Fernandes_Apresentação trabalho Redes Neurais.pptx` (versão final do pptx, subida um dia depois da anterior — conteúdo idêntico à primeira versão nos 5 slides, nenhuma nota desta análise muda em função da diferença de versão), `Aryadne Fernandes_dados_precificacao_dinamica_oficial.csv` (usado só para contexto das colunas)

---

## 1. Abertura

Aryadne, parabéns por concluir o projeto! Como Especialista de Finanças na Gerdau há 7 anos, faz todo sentido você ter escolhido precificação dinâmica de produtos siderúrgicos (Vergalhão, Tela e Treliça) — é uma extensão natural do raciocínio de receita e margem que você já trabalha no dia a dia, agora testando se uma rede neural capta padrões de demanda que um modelo linear simples deixaria passar. O cuidado de traduzir o ganho técnico em receita simulada por produto, em R$, já no pptx mostra que você pensou o projeto com a lente de negócio certa desde o início.

## 2. Resumo do projeto

O projeto usa uma base histórica de 3.288 observações (2023-2025) de três produtos siderúrgicos (Vergalhão de Aço, Tela, Treliça) para prever a demanda diária a partir de preço próprio, preço do concorrente, sazonalidade e histórico de vendas/estoque. Compara uma Regressão Linear (baseline) com um `MLPRegressor` otimizado via `GridSearchCV`, e usa as previsões de demanda para simular, produto a produto, o preço que maximizaria a receita esperada — comparando a receita obtida com o preço ótimo linear contra o preço ótimo da rede neural.

## 3. Nota por critério

### Critérios de negócio (peso maior)

**1. Aderência ao negócio — 10,0/10**
1.1. Métrica de sucesso nomeada como receita/custo: **5** — "maximizar receita" é o objetivo declarado explicitamente no slide 1 do pptx ("Objetivo: usar previsão de demanda e simulação de preços para apoiar decisões comerciais e maximizar receita"), e o próprio código nomeia colunas como `Ganho Incremental IA (R$)` (Seção 4 do notebook).
1.2. Métrica quantificada: **5** — valores em R$ e % explícitos, ex.: slide 4 do pptx, tabela "Receita Linear / Receita Rede Neural / Ganho": Tela R$ 31.584 → R$ 114.446 (+262,4%).
1.3. Conexão entre métrica técnica e impacto de negócio: **5** — o pptx conecta explicitamente R²/MAE (slide 3: "MAE da Tela: 18,95 → 6,37 unidades") ao ganho de receita simulado por produto (slide 4), e o notebook (Seção 4, "IMPACTO NO NEGÓCIO: MAXIMIZAÇÃO DE RECEITA ANUAL") converte a previsão de demanda diretamente em receita (`preco * demanda`).

**2. Viabilidade econômica (ROI) — 2,5/10**
2.1. Custo de construção estimado: **1** — não há qualquer estimativa de custo de dados, treinamento ou infraestrutura, nem no notebook nem no pptx.
2.2. Custo de sustentação estimado: **1** — o slide 5 ("Próximos Passos") menciona "Alimentação diária" e "Escalar o modelo", mas sem qualquer estimativa de custo de retraining, monitoramento ou infraestrutura em produção.
2.3. Retorno esperado estimado com número: **5** — presente e detalhado por produto: Vergalhão +21,8%, Tela +262,4%, Treliça +15,9% (slide 4 do pptx; `df_negocio_final` na Seção 4 do notebook).
2.4. Comparação explícita custo vs. retorno: **1** — nenhum payback, ROI% ou breakeven é calculado; só o lado do retorno existe, nunca comparado a um custo.

### Critérios técnicos (peso menor)

**3. Necessidade real de IA — 0,0/10**
3.1. Discute alternativa de automação/regra determinística: **1** — não há, em nenhum dos três arquivos, menção a uma regra simples de precificação (ex: ajuste percentual sobre o preço do concorrente por faixa de estoque) como alternativa considerada e descartada antes de partir para a rede neural.

**4. ML tradicional vs. Redes Neurais — 6,3/10**
4.1. Compara explicitamente contra ML tradicional: **5** — `LinearRegression` como baseline explícito, treinado e comparado produto a produto (notebook, Seção 3, `pipe_lr` vs. `best_mlp`; pptx slide 3).
4.2. Baseline simples de fato executado e comparado no notebook: **2** — a estrutura do código está correta (mesmo split, mesmas features, mesmas métricas para os dois modelos), mas como está escrito hoje o notebook **não chega a executar essa comparação**: a linha `best_mlp = grid_search.best_estimator__` (Seção 3) usa um atributo inexistente (`best_estimator__` com dois underscores; o correto é `best_estimator_`), o que gera `AttributeError` logo após o `grid_search.fit(...)`, antes mesmo de a primeira métrica ser calculada para o primeiro produto. Ou seja, a tabela `df_metricas_finais` — origem presumível dos números do pptx — não pode ter sido gerada pela versão publicada do código.

**5. Aderência ao conteúdo do curso — 10,0/10**
5.1. Nomeia arquitetura vista em aula: **5** — `MLPRegressor` (sklearn.neural_network), explicitamente chamado de "Rede Neural" no notebook e no pptx (Aula 2/3 — Perceptron multicamadas).
5.2. Arquitetura adequada ao tipo de dado: **5** — dado tabular (preço, sazonalidade, estoque, histórico de vendas), MLP é escolha adequada.
5.3. *(não aplicável — problema não é de texto)*

**6. Aderência ao template de projeto — 6,3/10**
6.1. Cobre os 7 blocos do `templates_projetos_ia.md`: **5** — os 7 blocos têm algum tipo de evidência, espalhada entre o pptx e os comentários do notebook: (1) visão/objetivo — slide 1; (2) coleta e engenharia de features — Seção 2 do notebook (razão de preço, gap, cobertura de estoque, codificação cíclica de sazonalidade); (3) split/estratégia de bases — Seção 3 ("Split temporal estratégico para evitar vazamento de dados"); (4) seleção de algoritmos — LR vs. MLP, Seção 3; (5) treinamento/otimização — `GridSearchCV` com grade de hiperparâmetros, Seção 3; (6) testes/métricas — R², MAE, RMSE, MAPE, Seção 3; (7) MLOps — o mais frágil dos sete: só dois bullets em "Próximos Passos" do pptx (slide 5: "Alimentação diária", "Escalar o modelo") e uma frase genérica sobre "monitorando receita, volume e margem", sem estratégia concreta de deploy, cadência de retraining ou monitoramento de drift.
6.2. Profundidade do bloco 7 (MLOps): **2** — o slide 5 nomeia o que monitorar em termos de negócio ("Decisão recomendada: Iniciar um piloto controlado de precificação dinâmica, monitorando receita, volume e margem versus a política tradicional"), o que já é mais específico que um genérico "monitorar periodicamente". Mas nenhum dos três bullets de "Próximos Passos" (1. "Alimentação diária — Atualizar preços dos concorrentes e sinais de demanda"; 2. "Otimizar lucro — Incluir custos para maximizar margem"; 3. "Escalar o modelo — Adicionar regiões, filiais e regras comerciais") define frequência de retreino do modelo — "Alimentação diária" fala em atualizar os dados de entrada, não em retreinar a MLP — nem existe qualquer critério numérico ou qualitativo de quando o modelo deve ser revisado ou aposentado.

**7. Correção técnica — 7,5/10**
7.1. Código executa do início ao fim sem erro: **1** — não apenas "sem evidência de execução" (o arquivo é `.txt`, sem outputs salvos): há dois bugs confirmados por leitura do código. O primeiro, já citado, é `grid_search.best_estimator__` (Seção 3), que quebra o notebook na primeira iteração do loop por produto. O segundo, mesmo que o primeiro fosse corrigido, está na Seção 4: `receita_otimizada_mlp += preco_otimo_mlp * demande_real_mlp` usa a variável `demande_real_mlp` (sem o "a"), que nunca foi definida — a variável calculada duas linhas acima é `demanda_real_mlp`. Isso gera `NameError` e impede que a simulação de receita (base dos números do slide 4) rode como está.
7.2. Split treino/teste antes de pré-processamento: **5** — o `ColumnTransformer` (StandardScaler + OneHotEncoder) está dentro do `Pipeline`, ajustado apenas em `X_train` via `pipe_lr.fit(X_train, ...)` e `grid_search.fit(X_train, ...)` (Seção 3) — sem vazamento.
7.3. Métrica de avaliação adequada: **5** — R², MAE, RMSE e MAPE são adequados para um problema de regressão de demanda contínua (Seção 3, `tabela_metricas`).
7.4. Baseline avaliado no mesmo split/dados que o modelo principal: **5** — `pipe_lr` e `best_mlp` usam exatamente o mesmo `X_train`/`X_test`/`y_train`/`y_test` por produto (Seção 3).

**8. Qualidade do código — 5,0/10**
8.1. Seeds fixadas: **5** — `np.random.seed(42)` (Seção 1), `random_state=42` no `MLPRegressor` e no `KFold` (Seção 3).
8.2. Dependências declaradas: **1** — não há célula `%pip install`, `requirements.txt` ou qualquer arquivo de ambiente no repositório; só os `import`s no topo do script.
8.3. Organização em funções/seções: **3** — o código é dividido em 4 seções numeradas com comentários de bloco (boa leitura) e evita duplicação usando um loop sobre `df['produto'].unique()`, mas não há nenhuma função (`def`) — tudo roda em nível de módulo/loop, sem encapsulamento nem reuso.

**9. Honestidade dos resultados — 0,0/10**
9.1. Resultado reportado com mais de 1 execução/seed, ou justificativa: **1** — apenas uma seed fixa (`random_state=42`), sem repetição nem justificativa de que não seria necessária. Isso pesa mais aqui porque o ganho reportado é grande e desigual entre produtos (R² até 0,97 na Tela, ganho de receita de +262,4% no mesmo produto) — exatamente o tipo de resultado "bom demais" que pede confirmação de robustez antes de virar slide de conclusão.
9.2. Seção de limitações presente: **1** — ausente tanto no notebook quanto no pptx; o slide 5 fala em "Próximos Passos", mas não em limitações do experimento atual (dados sintéticos/curto período, extrapolação de preço fora da faixa observada, ausência de custo).

## 4. Pontos fortes

- **1.1-1.3 Métrica de negócio nomeada, quantificada e conectada ao impacto técnico** (5/5): "maximizar receita" é objetivo explícito (pptx slide 1), quantificado em R$ e % por produto (slide 4: Tela R$ 31.584 → R$ 114.446, +262,4%) e conectado ao ganho técnico (MAE 18,95 → 6,37 na Tela, slide 3; coluna `Ganho Incremental IA (R$)`, notebook Seção 4).
- **2.3 Retorno esperado estimado com número** (5/5): ganho de receita simulado, detalhado por produto — Vergalhão +21,8%, Tela +262,4%, Treliça +15,9% (pptx slide 4; `df_negocio_final`, notebook Seção 4).
- **4.1 Compara explicitamente contra ML tradicional** (5/5): `LinearRegression` como baseline explícito, treinado e comparado produto a produto contra a MLP (notebook Seção 3, `pipe_lr` vs. `best_mlp`; pptx slide 3).
- **7.2 Split treino/teste antes de pré-processamento** (5/5): `ColumnTransformer` (StandardScaler + OneHotEncoder) dentro do `Pipeline`, ajustado apenas em `X_train` (notebook Seção 3) — sem vazamento de dados para o teste.

## 5. Pontos de melhoria

- **2.1/2.2/2.4 Custo de construção, sustentação e comparação custo vs. retorno** (1/5): nenhuma estimativa de custo de dados, treinamento ou infraestrutura, nem de retraining/monitoramento em produção — e por isso nenhum payback, ROI% ou breakeven é calculado. É a combinação de sub-itens de maior peso na rubrica e a que mais penaliza a nota final; hoje só o lado do retorno (2.3) existe.
- **7.1 Código executa do início ao fim sem erro** (1/5): dois bugs confirmados impedem a execução — `grid_search.best_estimator__` (Seção 3) e `demande_real_mlp` (Seção 4) — o que significa que os números do pptx não podem ter vindo da versão publicada do código.
- **9.1/9.2 Múltiplas seeds e seção de limitações** (1/5): resultado desproporcional entre produtos (R²=0,97 e +262,4% de receita na Tela) testado numa única seed, sem seção de limitações que discuta dados sintéticos, extrapolação de preço ou ausência de custo.
- **3.1 Discute alternativa de automação/regra determinística** (1/5): não há, em nenhum dos três arquivos, menção a uma regra simples de precificação como alternativa considerada e descartada antes de partir para a rede neural.

## 6. Nota final

**5,5 / 10** — A moldura de negócio é a mais forte que já vi na turma nesse critério específico (métrica de receita nomeada, quantificada e simulada produto a produto), mas isso é neutralizado por problemas técnicos concretos e verificáveis: o notebook tem dois bugs que impedem sua execução como está publicado, não há seção de limitações nem validação de robustez para um resultado que salta para +262% de receita num único produto, e o lado de custo do ROI simplesmente não existe — o que impede qualquer leitura real de retorno sobre investimento.

**Nível de maturidade: PoC/protótipo.** O pptx recomenda "iniciar um piloto controlado de precificação dinâmica" (slide 5), mas os critérios de negócio ainda não sustentam esse próximo passo: não há nenhuma estimativa de custo de construção ou sustentação (2.1/2.2), nenhuma comparação custo vs. retorno (2.4), e o bloco de MLOps se limita a bullets genéricos sem cadência de retreino nem critério de quando revisar o modelo (6.2). O projeto ainda está validando se a rede neural funciona no notebook (e hoje nem isso, por conta dos bugs em 7.1) — falta a base econômica e operacional para avançar para um piloto controlado de fato.

## 7. Task list para evoluir o trabalho

**2. Viabilidade econômica (ROI)**
- [ ] **2.1 Custo de construção estimado (1/5):** o Bloco C traz isso numa tabela de fatores a avaliar antes de escolher uma rede neural — linhas "custo computacional" / "treino e inferência têm custo, meça contra o orçamento disponível" e "tempo de treinamento" / "modelos complexos podem levar horas ou dias para treinar de novo". Para o `GridSearchCV` com grade de hiperparâmetros rodado por produto (notebook Seção 3), estime o custo de coleta/preparação dos dados de vendas e o tempo/infra necessários para treinar (e depois re-treinar) o modelo por produto. — ver notebook Seção 3 (`GridSearchCV`)
- [ ] **2.2 Custo de sustentação estimado (1/5):** o Bloco B contrasta automação com IA numa tabela comparativa — linha "Manutenção" / "atualiza-se a regra manualmente (automação) vs. retreina-se o modelo periodicamente (IA)" — esse é exatamente o custo recorrente que falta estimar: cadência de retreino por produto e custo de monitorar a receita simulada contra a realizada. — ver pptx slide 5 ("Alimentação diária", "Escalar o modelo")
- [ ] **2.4 Comparação explícita custo vs. retorno (1/5):** o Bloco C fecha o raciocínio de seleção de algoritmo com "Regra prática: comece sempre por um baseline simples. Se ele já resolve, um modelo mais complexo só se justifica se o ganho superar o custo." Depois de estimar 2.1/2.2 (custo), falta comparar contra o retorno já calculado em 2.3 (ganho de receita por produto) — o curso não formaliza uma fórmula, mas `ROI = (retorno - custo) / custo` ou um payback em meses já bastam para essa régua. — ausente em todo o material analisado

**3. Necessidade real de IA**
- [ ] **3.1 Discute alternativa de automação/regra determinística (1/5):** o Bloco B traz o teste direto: "A lógica pode virar regras fixas (se-então)? O problema muda muito/tem grande variação? Há dados históricos suficientes? Três "sim" seguidos = provavelmente um projeto de IA. Se alguma resposta for "não", automação simples resolve com menos custo e mais previsibilidade." O notebook `08-precificacao-dinamica.ipynb` (células 16-19) trata do mesmo domínio do seu projeto e já calcula `receita = preço × demanda_prevista` comparando modelos — vale usar essa mesma estrutura para registrar por que um ajuste percentual simples sobre o preço do concorrente (por faixa de estoque, por exemplo) não bastaria antes de partir para a MLP. — nenhuma menção encontrada nos três arquivos analisados

**4. ML tradicional vs. Redes Neurais**
- [ ] **4.2 Baseline simples de fato executado e comparado no notebook (2/5):** a estrutura já está correta (mesmo split, mesmas features para LR e MLP), mas o bug em `grid_search.best_estimator__` (Seção 3) impede que essa comparação realmente rode. Corrija o typo (o atributo correto é `best_estimator_`, com um underscore) e reexecute para confirmar que os números do pptx (slide 3) de fato vêm da comparação LR vs. MLP. — ver notebook Seção 3 (bloco "B) Rede Neural MLP com GridSearchCV")

**6. Aderência ao template de projeto**
- [ ] **6.1 Cobre os 7 blocos do template (5/5):** os 7 blocos estão presentes — critério cumprido integralmente pela contagem de blocos (ver evidência completa na seção 3). Fica como observação qualitativa, sem impacto na nota: a evidência está espalhada entre o pptx e os comentários do notebook, sem um documento de plano de projeto único e consolidado nos moldes do `templates_projetos_ia.md`. Consolidar isso num documento à parte facilitaria a defesa do trabalho e a auditoria de cada bloco (o bloco 7/MLOps, o mais frágil, hoje fica reduzido a dois bullets de slide — ver 6.2). — ver pptx (slides 1-5) e notebook (Seções 1-4)
- [ ] **6.2 Profundidade do bloco 7/MLOps (2/5):** o bloco 7 do template pede uma estratégia de implantação e monitoramento concreta o suficiente para orientar uma decisão real de produção. Hoje o pptx nomeia o quê monitorar em termos de negócio ("monitorando receita, volume e margem", slide 5), mas falta frequência de retreino (a "Alimentação diária" citada é sobre atualizar dados de entrada, não sobre retreinar a MLP) e um critério de quando revisar ou aposentar o modelo. O Bloco B já dá a pista de que essa periodicidade precisa ser decidida explicitamente: "retreina-se o modelo periodicamente (IA)". — ver pptx slide 5 ("Próximos Passos" / "Decisão recomendada")

**7. Correção técnica**
- [ ] **7.1 Código executa do início ao fim sem erro (1/5):** dois bugs confirmados impedem a execução: `grid_search.best_estimator__` (Seção 3, o atributo correto é `best_estimator_`) e `demande_real_mlp` (Seção 4, a variável definida duas linhas acima é `demanda_real_mlp`). Isso não é um conceito do curso, é disciplina de execução: corrija os dois typos, rode o notebook do início com kernel limpo e só então reporte os números do pptx como reprodutíveis (idealmente salvando como `.ipynb` com outputs). — ver notebook Seção 3 (bloco "B) Rede Neural MLP com GridSearchCV") e Seção 4 (loop de simulação de preços)

**8. Qualidade do código**
- [ ] **8.2 Dependências declaradas (1/5):** não é conteúdo do curso (é boa prática geral de engenharia, não de IA de negócio), mas vale como dica prática: adicionar uma célula `%pip install` ou um `requirements.txt` no repositório evita que quem for reproduzir o trabalho precise adivinhar versões de scikit-learn/pandas. — ver notebook Seção 1
- [ ] **8.3 Organização em funções/seções (3/5):** também fora do escopo do curso, mas como dica prática: o código já evita duplicação com um loop sobre `df['produto'].unique()` (Seções 3-4); extrair esse loop e as etapas de treino/simulação em funções (ex: `treinar_modelos(produto)`, `simular_receita(produto)`) facilitaria testar isoladamente as correções dos bugs de 7.1. — ver notebook Seções 3 e 4

**9. Honestidade dos resultados**
- [ ] **9.1 Resultado reportado com múltiplas seeds (1/5):** o Bloco A é direto — "Cada comparação foi rodada com 3 sementes aleatórias diferentes (não uma vez só), para separar ganho real de sorte da rodada", com exemplos como "CNN varia de 40% a 83% de acurácia dependendo da semente." O ganho reportado para a Tela (R²=0,97, +262,4% de receita) é grande e desigual o bastante entre os três produtos para pedir o mesmo teste: repita o treino da MLP com pelo menos 3 seeds (ex: 42, 7, 123) e reporte média ± desvio-padrão de R²/MAE e do ganho de receita simulado, produto a produto. — ver notebook Seção 3 (`MLPRegressor(...random_state=42)`) e pptx slides 3-4
- [ ] **9.2 Seção de limitações presente (1/5):** ausente tanto no notebook quanto no pptx — o slide 5 fala em "Próximos Passos", não em limitações do experimento atual. Vale registrar explicitamente: dados sintéticos/período curto (2023-2025), extrapolação de preço fora da faixa observada no treino (a simulação testa -20% a +20% do preço do concorrente na Seção 4, sem checar se esse intervalo está dentro do que o modelo viu) e ausência de custo no ROI (2.1/2.2/2.4). — ver pptx slide 5 ("Conclusão e Próximos Passos")

## 8. Tópicos para o aluno revisar

- **Validação de execução ponta a ponta / reprodutibilidade** (Bloco C — Design de Projetos de IA) — motivado por `Aryadne Fernandes_notebook_precificação dinâmica.txt`: os bugs em `best_estimator__` (Seção 3) e `demande_real_mlp` (Seção 4) mostram a importância de rodar o notebook do zero (kernel limpo) antes da entrega para garantir que os números reportados no pptx realmente vêm do código publicado.
- **Validação de robustez / múltiplas seeds** (Bloco C) — motivado pelo pptx (slides 3-4): R²=0,97 e ganho de +262,4% de receita para Tela são resultados grandes o suficiente para pedir confirmação com mais de uma execução antes de virar conclusão de negócio.
- **ROI e viabilidade econômica de projetos de IA** (Bloco C / `templates_projetos_ia.md`, seção MLOps) — motivado pela ausência total de estimativa de custo tanto no notebook quanto no pptx: o template pede explicitamente custo de dados/treino/infra vs. retorno, e hoje só o lado do retorno está presente.
- **Validação cruzada em dados com componente temporal** (Aula 2/3 ou Bloco B) — motivado pelo uso de `KFold(n_splits=4, shuffle=True, random_state=42)` na Seção 3 do notebook: como o próprio split treino/teste já é temporal (`sort_values(by='data')`, "Anti-Leakage"), vale revisitar `TimeSeriesSplit` para manter essa mesma disciplina também dentro do `GridSearchCV`, em vez de embaralhar (`shuffle=True`) observações que têm ordem temporal.
- **Extrapolação de redes neurais fora do domínio de treino** (Aula 2 — Arquitetura e Camada Oculta) — motivado pela simulação de preços de -20% a +20% em torno do concorrente (notebook Seção 4) sem checar se esse intervalo está dentro da faixa de preços vista no treino; isso pode ajudar a explicar por que o ganho simulado para Tela (+262,4%) é tão maior que o dos outros dois produtos.
