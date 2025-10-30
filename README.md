# CHALLENGE Sprint 4: Otimização de Estoque com Programação Dinâmica

## Identificação do Grupo — 2ESR

* 558385 - Alexia Ramalho Izidio Dos Santos
* 554746 - Beatriz Vieira de Novais
* 559008 - Hellen Aparecida Moura Silva
* 557397 - Lorenzo Adinolfi Acquesta
* 558859 - Wendell Dos Santos Silva

## Código no Github e no Colab

[COLAB](https://colab.research.google.com/drive/1u7DwrIhKxBQDeZrykE3fN9cXobE37d3Z?usp=sharing)

---

### 1. Resumo do Problema

Nas unidades de diagnóstico, a falta de precisão no registro do consumo (abordada na Sprint 3) leva a um controle de estoque ineficiente, gerando custos de reposição (pedidos urgentes/custo fixo alto) ou de desperdício (excesso de estoque/custo de retenção alto).

O objetivo desta Sprint foi modelar este problema usando **Programação Dinâmica (PD)**. Assumindo que a análise de dados da Sprint 3 nos permite gerar uma *previsão de demanda* para os próximos $T$ dias, a PD pode encontrar a **política de reposição ótima** que minimiza o custo total (soma do custo fixo de pedidos ($K$) e do custo de manutenção de estoque ($h$)) ao longo do período. Este é o problema clássico de **Dimensionamento de Lote Dinâmico (Wagner-Whitin)**.

### 2. Formulação do Problema (PD)

Para modelar o problema de forma a permitir a Programação Dinâmica, definimos os seguintes componentes, baseados na lógica de decomposição ótima:

| Componente | Definição no Contexto do Estoque | Notação (Equação de Recorrência) |
| :--- | :--- | :--- |
| **Estado** | O dia atual ($t$) em que a decisão de pedido é tomada. | $t \in \{0, 1, \dots, T-1\}$ |
| **Decisão** | Escolher o último dia ($j$) cuja demanda será atendida pelo pedido feito em $t$. | $j \in \{t, t+1, \dots, T-1\}$ |
| **Função Objetivo** | Minimizar o **Custo Total** para atender a demanda de $t$ até $T-1$, $C(t)$. | Encontrar $C(0)$ |
| **Caso Base** | O custo após o último dia ($T$) é zero. | $C(T) = 0$ |
| **Função de Recorrência** | O Custo Mínimo é a decisão $j$ que minimiza a soma dos custos. | $$C(t) = \min_{t \le j < T} \{ K + \text{HoldingCost}(t, j) + C(j+1) \}$$ |

Onde:
* $K$: Custo fixo por pedido (frete/administrativo).
* $\text{HoldingCost}(t, j)$: Custo de estoque para manter a demanda dos dias $t+1$ a $j$ em estoque.
* $C(j+1)$: Custo mínimo para o período remanescente (recursão).

---

### 3. Explicação do Uso dos Algoritmos

Implementamos três versões da solução de PD para validar a lógica e demonstrar a melhoria de desempenho.

#### a. Versão Recursiva Pura (`solve_recursive`)

* **Contexto do Problema:** Representa a formulação **literal** da Equação de Recorrência. A função é chamada para calcular o $C(j+1)$ para cada decisão $j$.
* **Como foi usado:** Serve como a prova inicial de que a lógica de "custo atual + custo futuro" está correta.
* **Limitação:** É ineficiente ($O(2^T)$) devido a **subproblemas sobrepostos** (recalcular $C(t)$ várias vezes). É inviável para horizontes de planejamento $T$ muito longos.

#### b. Versão com Memorização (`solve_memo`)

* **Contexto do Problema:** Aplica a técnica de "Top-Down" PD, onde resultados já calculados são armazenados (memoizados) para reuso.
* **Como foi usado:** Utilizamos um dicionário (`memo`) para armazenar o valor de $C(t)$ após seu primeiro cálculo. Isso garante que cada subproblema ($C(t)$) seja resolvido apenas uma vez.
* **Vantagem:** Reduz a complexidade de exponencial para **polinomial** ($O(T^2)$), tornando o algoritmo prático.

#### c. Versão Iterativa / Bottom-Up (`solve_bottom_up`)

* **Contexto do Problema:** Implementação "clássica" e mais eficiente da PD. A solução é construída do caso base ($T$) para o estado inicial ($0$).
* **Como foi usado:** Preenchemos um array (`dp`) de trás para frente, garantindo que o valor de $C(j+1)$ (o *custo futuro*) já esteja disponível quando calculamos $C(t)$.
* **Vantagem Principal:** É a versão mais limpa e performática. Além disso, ela nos permitiu armazenar o **$j$ ótimo** (o array `policy`) para cada $t$. Este array **é o plano de ação** final, traduzindo o resultado da otimização em uma diretriz prática para a compra de insumos.

### 4. Conclusão: Melhorando a Visibilidade e Reduzindo Desperdícios

A solução de Programação Dinâmica oferece uma abordagem robusta para o controle de estoque:

* **Melhora a Visibilidade:** O resultado final é um **plano de ação claro** (`print_policy`), que indica exatamente *quando* fazer o pedido e *quanto* pedir, eliminando a incerteza no consumo diário.
* **Reduz Desperdícios:** O algoritmo encontra o **equilíbrio ótimo** (Custo Mínimo Total) entre os custos de pedido ($K$) e os custos de estoque ($h$), garantindo que a demanda seja atendida (sem falta de estoque) com o menor custo financeiro possível.
