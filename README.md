# Relatório Sprint 4: Otimização de Estoque com Programação Dinâmica

## Identificação do Grupo — 2ESR

* 558385 - Alexia Ramalho Izidio Dos Santos
* 554746 - Beatriz Vieira de Novais
* 559008 - Hellen Aparecida Moura Silva
* 557397 - Lorenzo Adinolfi Acquesta
* 558859 - Wendell Dos Santos Silva

## Código no Github

[CÓDIGO NO COLAB](https://colab.research.google.com/drive/1u7DwrIhKxBQDeZrykE3fN9cXobE37d3Z?usp=sharing)

---

### 1. Resumo do Problema

Nas unidades de diagnóstico, a falta de precisão no registro do consumo (abordada na Sprint 3) leva a um controle de estoque ineficiente, gerando custos de reposição (pedidos urgentes) ou de desperdício (excesso de estoque).

O objetivo desta Sprint foi modelar este problema usando **Programação Dinâmica (PD)**. Assumindo que a análise de dados da Sprint 3 nos permite gerar uma *previsão de demanda* para os próximos `$T$` dias, a PD pode encontrar a **política de reposição ótima**, ou seja, um plano de pedidos que minimiza o custo total (soma do custo fixo de pedidos e do custo de manutenção de estoque) ao longo do período.

### 2. Formulação do Problema (PD)

Para modelar o problema, definimos os seguintes componentes:

* **Estados (`$t$`):** Um estado `$t$` (onde `$0 \le t < T$`) representa o dia atual para o qual devemos tomar uma decisão. Nossa função `$C(t)$` é definida como o *custo mínimo para atender toda a demanda futura, começando do dia `$t$` até o dia final `$T-1$`*.
* **Decisões (`$j$`):** Estando no dia `$t$`, a decisão é escolher um dia `$j$` (`$t \le j < T$`) e fazer um pedido *hoje* (dia `$t$`) que seja suficiente para cobrir toda a demanda até o dia `$j$`.
* **Função Objetivo:** O objetivo final é encontrar `$C(0)$`, que representa o custo mínimo para todo o horizonte de planejamento de `$T$` dias.
* **Função de Transição (Recorrência):** O custo mínimo `$C(t)$` é encontrado testando *todas* as decisões `$j$` possíveis e escolhendo a que minimiza o custo total.

    $Custo(t, j) = (\text{Custo Fixo do Pedido}) + (\text{Custo de Estoque}) + (\text{Custo Futuro})$

    $$C(t) = \min_{t \le j < T} \{ K + \text{HoldingCost}(t, j) + C(j+1) \}$$

    Onde `$K$` é o custo fixo do pedido (ex: frete). `$\text{HoldingCost}(t, j)$` é o custo de manter a demanda dos dias `$t+1$` até `$j$` em estoque. `$C(j+1)$` é o custo mínimo do restante do período, que já foi (ou será) calculado.

* **Caso Base:** `$C(T) = 0$`. O custo para atender a demanda após o último dia é zero.

### 3. Explicação do Uso dos Algoritmos

Implementamos três versões da solução de PD para demonstrar a evolução do conceito e garantir que os resultados sejam idênticos.

**a. Versão Recursiva Pura (`solve_recursive`)**

* **Como foi usada:** Foi a implementação literal da Função de Transição descrita acima. A função se chama recursivamente para calcular o `$C(j+1)$` (Custo Futuro) para cada decisão `$j$` possível.
* **Contexto:** Esta versão serve como uma prova de conceito de que a lógica da formulação está correta. No entanto, ela é computacionalmente ineficiente (complexidade exponencial), pois sofre do problema de **subproblemas sobrepostos**: ela recalcula o custo para o mesmo dia (ex: `$C(5)$`) milhares de vezes.

**b. Versão com Memorização (`solve_memo`)**

* **Como foi usada:** Esta é uma otimização "Top-Down" da versão recursiva. Utilizamos um dicionário (`memo`) para armazenar o resultado de `$C(t)$` na primeira vez que ele é calculado.
* **Contexto:** Antes de calcular `$C(t)$`, a função verifica se `$t$` já está no `memo`. Se estiver, ela retorna o valor salvo imediatamente, evitando o recálculo. Isso "poda" os ramos repetidos da árvore de recursão e resolve o problema dos subproblemas sobrepostos, tornando a solução eficiente (complexidade polinomial).

**c. Versão Iterativa / Bottom-Up (`solve_bottom_up`)**

* **Como foi usada:** Esta é a abordagem de PD "clássica" (Bottom-Up). Em vez de começar do dia 0, ela começa do caso base (`$C(T)=0$`) e constrói a solução de trás para frente, preenchendo uma tabela (array `dp`).
* **Contexto:** Ela calcula `dp[T-1]`, depois `dp[T-2]` (usando o valor salvo de `dp[T-1]`), e assim por diante, até chegar em `dp[0]`, que é a resposta final.
* **Vantagem Principal:** Esta é a versão mais eficiente, pois elimina a sobrecarga da recursão. Além disso, ela nos permitiu salvar a *decisão ótima* (o `best_j`) para cada dia `$t$` em um array `policy`. É esse array `policy` que nos permite reconstruir o plano de ação.

### 4. Conclusão: Melhorando a Visibilidade e Reduzindo Desperdícios

A solução de Programação Dinâmica atende diretamente aos requisitos do problema:

1.  **Melhora a Visibilidade:** O resultado final não é apenas um número (o custo mínimo), mas sim um **plano de ação** (o array `policy`). Ao executar a função `print_policy`, a gestão da unidade de diagnóstico tem uma lista clara de **quando** pedir e **quanto** pedir (ex: "Dia 0: Pedir 60 unidades para cobrir até o dia 4").
2.  **Reduz Desperdícios:** O algoritmo garante que este plano de ação é o que tem o **custo total mínimo**. Ele equilibra de forma ótima o custo de fazer muitos pedidos (muito custo `$K$`) contra o custo de fazer poucos pedidos grandes (muito custo de estoque `$h$`), garantindo que a demanda seja atendida sem excessos caros.
