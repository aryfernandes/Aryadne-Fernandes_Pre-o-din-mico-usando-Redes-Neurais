# Guia Conceitual — Aryadne Meira Fernandes (`aryfernandes`)

**Esta rodada não tem nota.** As quatro rodadas anteriores (v1: 5,5 → v2: 5,0 → v3: 6,5 → v4: 4,0) mostraram que o feedback técnico ponto-a-ponto não está mudando o padrão de erro — o mesmo bug de escala foi corrigido na v3 e voltou na v4, só que sob outro nome de variável. Isso me diz que o problema não é falta de atenção a um pedido específico, é uma lacuna de entendimento em alguns conceitos que se repetem. Este documento existe para fechar essa lacuna antes de pedir uma próxima entrega avaliada.

A próxima entrega que você fizer **será tratada como v5, com nota normal**, seguindo a rubrica de sempre. Este guia não conta como rodada.

---

## Conceito 1 — Consistência de unidades e escala

**O erro concreto no seu código (v4, célula "Fase 3"):**

```python
fator_escala = row['demanda'] / (row['historico_vendas_7d'] / 7.0 + 1e-5)
```

**Por que isso quebra:** `demanda` é uma quantidade em um período (ex.: unidades vendidas *hoje* ou *nesta semana* — depende de como a coluna foi construída). `historico_vendas_7d` já é a soma/janela de 7 dias. Dividir esse histórico por 7 antes de comparar com `demanda` só faz sentido se `demanda` estiver numa escala "por dia" **e** `historico_vendas_7d` precisar virar "por dia" também para a razão fazer sentido. Você não verificou isso — só assumiu que a divisão "normaliza".

Regra prática: **antes de dividir ou multiplicar duas colunas, pergunte "essas duas colunas estão na mesma unidade e no mesmo período?"** Se a resposta não for um "sim" verificado (não assumido), a conta está inflando ou reduzindo artificialmente o resultado.

```
ERRADO (o que a v4 faz):
  demanda ≈ 100 unidades/semana
  historico_vendas_7d ≈ 700 unidades (soma de 7 dias)
  fator_escala = 100 / (700/7) = 100/100 = 1,0   ← parece OK aqui...

  ...mas quando historico_vendas_7d já está na MESMA escala que demanda
  (ambos ~100, porque os dados foram gerados de forma que a razão é ≈1,0):
  fator_escala = 100 / (100/7) = 100/14,3 ≈ 7,0   ← infla por ~7x

CERTO (o que a v3 fazia, corretamente):
  fator_demanda_real = row['demanda'] / (historico_7d + 1e-5)
  fator_demanda_real = 100 / 100 = 1,0   ← sem divisor arbitrário
```

**Como verificar isso na prática, sempre:** calcule a razão entre as duas colunas brutas (sem nenhuma transformação) para uma amostra de linhas, e olhe se o valor médio está perto de 1,0, de 7,0, de 0,14 etc. Isso te diz, com dado real, se falta ou sobra um fator de escala — em vez de adivinhar.

---

## Conceito 2 — Vazamento de dado dentro de uma simulação (data leakage)

Um segundo problema no mesmo trecho: `fator_escala` usa `row['demanda']` — o valor **real observado** do período de teste — para construir um fator que depois é aplicado tanto na simulação da heurística quanto na simulação com IA.

**Por que isso é grave:** a ideia de simular "quanto eu ganharia se usasse preço ótimo" só tem valor se a simulação representar uma decisão que você tomaria **sem saber o resultado real**. Se o fator usado na simulação já contém o valor real da demanda, você está usando informação do futuro (do resultado) para calcular o ganho — o que faz qualquer ganho parecer maior do que seria na vida real, porque parte da "mágica" é ter espiado a resposta.

```
CONCEITO GERAL DE VAZAMENTO:

  Treino/Simulação  ──usa──>  informação que só existe DEPOIS do evento
         │
         └──> resultado parece ótimo, mas não é reproduzível na prática real,
              porque no mundo real você não teria esse dado no momento da decisão

  Pergunta de verificação: "Se eu rodasse isso ONTEM, sem saber o resultado
  de HOJE, eu teria essa variável disponível?" Se não, ela não pode entrar
  na simulação/decisão.
```

Isso é o mesmo princípio geral por trás do "vazamento por proximidade" que já vimos em outros projetos da turma (dado de teste vazando pro treino) — aqui é uma variante: dado do "futuro" (resultado real) vazando pra dentro do cálculo que deveria representar a decisão.

---

## Conceito 3 — Reprodutibilidade: o notebook e o slide têm que vir do mesmo lugar

Na v4, o slide 1 tem números (+19,06%/+18,67%/+23,37%) que são da **v3** — não os que o notebook V4 atual produz (+639,98%/+695,36%/+637,10%, por causa do Conceito 1). Isso não é um erro de digitação isolado — é sintoma de um hábito: gerar o slide olhando um resultado antigo (de memória, de um print salvo, de uma versão anterior), em vez de olhando a tela do notebook que você acabou de rodar.

```
FLUXO QUE CAUSA O PROBLEMA:
  editar código → (sem rodar tudo de novo) → copiar número antigo do slide
                                              anterior → entregar

FLUXO CORRETO:
  editar código → Reiniciar o kernel → Executar TODAS as células, em ordem,
  do início → ler os números QUE APARECEM NA TELA agora → só então
  escrever esses números no slide
```

Checklist de 30 segundos antes de gerar qualquer slide com número:
1. Reinicie o kernel (Restart) e rode tudo do início (Run All).
2. Abra o notebook já executado e localize, na tela, cada número que vai para o slide.
3. Copie o número da tela — não de memória, não de uma versão anterior, não do slide anterior.
4. Se um número mudou desde a última vez, isso é esperado quando o código muda — não é motivo para manter o número antigo "porque parecia melhor".

---

## Conceito 4 — Por que células markdown importam (não é burocracia)

Você recebeu esse pedido 3 vezes (v2, v3, v4) e nas 3 vezes o notebook entregue tinha zero células markdown. O motivo do pedido não é estético — é que, sem elas, eu (e você, revendo seu próprio código depois de duas semanas) não tenho como saber **o que mudou e por quê** só olhando código. Uma célula markdown curta antes de uma mudança relevante ("Aqui corrigi o fator de escala que estava dividindo por 7 sem necessidade") é o que teria feito o Conceito 1 saltar aos olhos antes de eu precisar reexecutar tudo duas vezes para provar o retrocesso.

Regra prática: toda vez que você mudar uma fórmula que já tinha sido discutida/corrigida numa rodada anterior, escreva uma célula markdown dizendo o que mudou. Isso é tanto para o revisor quanto para você mesma não perder o controle do que já foi corrigido.

---

## Conceito 5 — Uma premissa de negócio, um número, em todo lugar

No ROI, você tem três valores diferentes para a mesma premissa (margem): "10%" no texto do cabeçalho, "12%" num comentário inline, e `0.08` (8%) na variável realmente usada no cálculo. Escolha um valor, documente de onde ele vem (mesmo que seja uma suposição, tipo "assumindo margem de 10% por falta de dado real do produto"), e use essa mesma variável em todos os lugares — texto, comentário e código precisam contar a mesma história.

---

## Fechamento

Nenhum desses 5 conceitos é complicado isoladamente — o que preocupa é que o mesmo padrão (escala sem verificar, número que não bate com o código atual, ausência de explicação do que mudou) se repete desde a v2. Releia este guia antes de mexer no notebook de novo, e use os checklists dos Conceitos 1 e 3 como parte do seu processo, não como algo a conferir só no fim.

A próxima entrega será avaliada normalmente, como v5.
