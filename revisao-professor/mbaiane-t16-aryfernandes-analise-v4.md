# Análise v4 — Aryadne Meira Fernandes (`aryfernandes`)

**Projeto:** Precificação Dinâmica com Redes Neurais — previsão de demanda e simulação de preço ótimo para maximizar receita
**Repositório:** [aryfernandes/Aryadne-Fernandes_Pre-o-din-mico-usando-Redes-Neurais](https://github.com/aryfernandes/Aryadne-Fernandes_Pre-o-din-mico-usando-Redes-Neurais)
**Arquivos analisados (v4, commit `a420aefb`):** `V4_Aryadne Fernandes_notebook.ipynb`, `V4_Aryadne Fernandes_apresentacao precificação dinamica.pptx`
**Nota v1:** 5,5/10 → v2: 5,0/10 → v3: 6,5/10 → **Nota v4: 4,0/10**

---

## 1. Abertura

Aryadne, preciso ser direto: esta v4 é um retrocesso, e o motivo específico me preocupa mais do que a queda de nota em si. O bug de escala que você tinha corrigido corretamente na v3 (aquele que inflava a receita simulada para +900%/+1.100%) **voltou** — sob um nome de variável diferente, mas com o mesmo erro matemático. E o slide 2 desta v4 afirma explicitamente que esse mesmo cálculo "foi corrigido, retirando o divisor de escala temporal que estava sendo aplicado de forma errada" — o que é o oposto exato do que o código que você entregou faz.

Reexecutei tudo duas vezes de forma independente para ter certeza absoluta antes de escrever isso. Vou detalhar cada ponto, porque o caminho de volta é claro e você já provou, na v3, que sabe fazer essa correção certo.

## 2. As 5 correções pedidas na v3 — verificação item a item

| # | Pedido da v3 | Situação na v4 |
|---|---|---|
| 1 | Corrigir o "-3,2%" do slide 1 (regra heurística) | **Trocado por outro número desatualizado.** O slide agora traz +19,06%/+18,67%/+23,37% — que são os números **da v3**, não os que o notebook V4 atual produz (639,98%/695,36%/637,10%, por conta do achado da seção 3). |
| 2 | Corrigir 2 dígitos trocados na tabela do slide 2 | **Corrigidos no texto**, mas a tabela inteira continua sendo a da v3 — não reflete o que o notebook V4 produz agora. |
| 3 | Adicionar célula markdown explicando a evolução | **Não feito.** O notebook tem 5 células, todas de código, zero markdown — 3ª vez que esse pedido é feito e ignorado. |
| 4 | Calcular o ROI/payback a partir do código | **Parcialmente.** Existe uma célula nova que calcula ROI/payback — mas o resultado (ROI 5.477%, payback de 3 dias) é absurdo, consequência direta do achado da seção 3, e **o slide 3 não mostra nenhum número de ROI** — só a frase "não conseguimos calcular com precisão o ROI por se tratar de dados fictícios". O cálculo existe no código e foi escondido do slide. |
| 5 | Corrigir o bug do gráfico de elasticidade (1 modelo reusado para 3 produtos) | **Corrigido, e bem verificado.** Agora existe `modelos_treinados = {}`, um por produto — confirmei com `id()` que os 3 modelos usados no gráfico são objetos distintos, e as 3 curvas do slide têm formatos e escalas visivelmente diferentes entre si. |
| — | Opcional: incluir `LinearRegression` na validação robusta | **Regrediu.** Na v3 ela ao menos era usada na simulação; na v4 é importada mas nunca instanciada — código morto. |

## 3. Achado mais grave — o bug de escala corrigido na v3 voltou, com um claim falso no slide negando isso

Na v3, você tinha corrigido (com comentário próprio no código): `fator_demanda_real = row['demanda'] / (historico_7d + 1e-5)`. Nesta v4, a célula "Fase 3" tem:

```python
fator_escala = row['demanda'] / (row['historico_vendas_7d'] / 7.0 + 1e-5)
```

A divisão por `/ 7.0` foi **reintroduzida** — exatamente o que a v3 tinha removido. Confirmei que é um erro real: `demanda` e `historico_vendas_7d` estão na mesma escala (razão entre elas ≈1,0 nos 3 produtos), então dividir por 7 infla o fator sistematicamente em ~7x. Reexecutei 2 vezes, de forma independente, e obtive:

| Produto | Ganho Heurística (%) real, V4 | Ganho IA (%) real, V4 | (para comparação, v3) |
|---|---|---|---|
| Tela de Aço | 639,98% | 1.022,57% | 19,06% / 60,37% |
| Treliça | 695,36% | 927,51% | 18,67% / 46,79% |
| Vergalhão de Aço | 637,10% | 1.161,54% | 23,37% / 80,22% |

Esses números (600%-1.100%) são da mesma ordem de grandeza do erro original de rodadas anteriores à v3. Há ainda um segundo problema no mesmo trecho: `fator_escala` usa `row['demanda']` — o valor real do período de teste — multiplicando a própria previsão do modelo, o que mistura o rótulo real dentro da simulação de receita tanto para a heurística quanto para a IA.

O ponto que mais preciso que você entenda: o slide 2 desta v4 diz *"Nota de ajuste no modelo: o cálculo da demanda foi corrigido, retirando o divisor de escala temporal que estava sendo aplicado de forma errada."* Isso descreve exatamente a correção que você fez **na v3** — mas não é o que o código **desta v4** faz. Reexecutei o notebook que você entregou, não o da v3, e o divisor está de volta.

## 4. Achados menores

- Inconsistência na premissa de margem: o cabeçalho do ROI diz "margem esperada de 10%", um comentário inline diz "12%", mas a variável usada no código é `0.08` (8%) — três números diferentes para a mesma premissa.
- A apresentação perdeu conteúdo: caiu de 4 slides (v3) para 3 (v4) — todo o slide de governança/MLOps (batch scoring, monitoramento de drift, plano de rollback) foi removido, sem explicação (o que uma célula markdown, pedida 3 vezes, teria evitado).

## 5. O que não regrediu

O núcleo técnico mais difícil continua de pé: a validação `TimeSeriesSplit(n_splits=3)` + 3 seeds reproduz exatamente os mesmos R²/MAPE de antes (0,96-0,97 / 12,4%-14,3%) em 2 execuções independentes. E a correção do gráfico de elasticidade (item 5 da tabela) foi genuinamente bem feita — vale reconhecer isso.

## 6. Nota final

**4,0 / 10** *(v3: 6,5/10)* — Reconheço o que funcionou: o bug do gráfico de elasticidade foi corrigido com evidência sólida, e a validação temporal que era o núcleo mais difícil do projeto continua intacta. Mas a nota cai abaixo da v3 porque o achado mais grave de todo o histórico deste projeto — o erro de escala que inflava a receita — voltou nesta rodada, disfarçado sob outro nome de variável, e o slide afirma explicitamente que ele foi corrigido quando o código mostra o oposto. Isso não é a mesma categoria dos erros de digitação das rodadas anteriores — é uma alegação que contradiz diretamente a execução real do código entregue, no mesmo ponto que já tinha exigido duas rodadas de correção.

**Nível de maturidade: retrocedeu para PoC/protótipo.** A v3 tinha alcançado uma validação técnica confiável; esta v4 reabre exatamente a dúvida que a v3 tinha fechado.

## 7. O que preciso que você corrija para a v5

1. **Prioridade máxima: remova o `/ 7.0`** da linha `fator_escala` (célula "Fase 3") — volte para a fórmula que você mesma escreveu corretamente na v3 (`row['demanda'] / (historico_7d + 1e-5)`, sem o divisor).
2. **Regenere a tabela do slide 2 e os números do slide 1 a partir da execução real do notebook V4** depois da correção acima — não copie de uma versão anterior.
3. **Mostre o ROI/payback no slide**, calculado a partir do código, depois que os números de receita estiverem corretos — não esconda o resultado.
4. **Adicione pelo menos uma célula markdown** explicando o que mudou nesta versão e por quê — este é o quarto pedido consecutivo do mesmo ponto.
5. **Reconcilie a premissa de margem** (10%, 12% ou 8% — escolha uma e use a mesma em todo lugar).
6. Antes de gerar os slides, **rode o notebook do início ao fim e confira se cada número do slide bate com o que aparece na tela** — é a mesma verificação de 30 segundos que já pedi para outro colega, e que teria pego o problema da seção 3 sozinha.

Você já demonstrou, na v3, que sabe fazer essa correção específica corretamente. O caminho de volta é reaplicar exatamente o que já funcionou.
