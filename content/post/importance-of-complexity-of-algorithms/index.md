---
title: Importância de se conhecer complexidade de algoritmos
description: Porque é importante entender sobre complexidade dos algoritmos e como isso impacta o seu software
slug: importance-of-complexity-of-algorithms
date: 2023-08-01 00:00:00+0000
draft: false
comments: false
image: cover.png
categories:
  - cs
tags:
  - Algorithms
  - Data Structures
---

Alguns dias atrás estava resolvendo um problema no [Hackerrank](https://www.hackerrank.com/) utilizando
a estrutura de dados Disjoint Sets. A primeira solução que implementei não passou em um dos testes,
quando estava investigando, percebi que não tinha aplicado uma otimização de ranking na estrutura, o que fazia  
um dos testes falhar por estar cobrindo justamente esse caso base. Após aplicada a otimização, o
mesmo caso de teste que testava **demorando 8 segundos para executar passou a executar poucos millissegundos**.

Esse cenário aconteceu porque **a primeira implementação estava executando em O(N) e a segunda implementação passou a executar em O(log N).**
Como esse caso ficou bem evidente a diferença de tempo de execução, resolvi escrever um pouco sobre a importância
de se conhecer um pouco e saber avaliar minimamente a complexidade de um algoritmo.

**Obs: O objetivo desse Post não é ensinar como fazer análise de algoritmos, já tem bons conteúdos sobre o assunto.
É mais sobre como isso impacta o software e como é importante conhecer minimamente.**

## Análise assintótica

A análise assintótica é utilizada para definir matematicamente o quanto um algoritmo utilizará de recursos computacionais
para ser executado. Através dela é possível comparar a escalabilidade de diferentes algoritmos de forma objetiva.

Na análise assintótica, podem ser considerarados: o melhor caso (Big Omega), pior caso (Big O) ou caso médio (Big Theta).
O mais comum fora do meio acadêmico é utilizar a notação Big O (pior caso de execução).

Os tempos de execução mais comuns utilizados são:

### Notação Big O

Como dito, a notação Big O, serve para representar o pior caso de execução de um algoritmo. Então, por exemplo,
caso você esteja inserindo um item em uma Linked List

## Exemplo - Fibonacci

@TODO

## Exemplo - Disjoint Set

O [Problema do hackerrank](https://www.hackerrank.com/challenges/components-in-graph) envolvia identificar os
componentes independentes de um grafo. Ou seja, **a solução envolvia contar quantos conjuntos disjuntos existiam** ao passar
por todos os nós do grafo. Para essa solução se utiliza a **estrutura de dados Disjoint Set**, ou também conhecida como
**_Union-find data structure_**.

A implementação mais simples de se fazer dessa estrutura, utilizando o problema como exemplo é a seguinte:

{{< gist leometzger bcb858b518bcca0fa90911a4957b333d>}}

Um teste automatizado para simular o pior caso seria

{{< gist leometzger 62db6a1079d8f22ff764a79cf2ca0125>}}

Ao aplicar a otimização de ranking para a estrutura **_Disjoint Set_**

{{< gist leometzger 244651f23a527d7457c06c70b9140644>}}

## Conclusão

reflexão...

conclusão...

### Referências

- [Livro: Cracking the coding interview](https://www.amazon.com/Cracking-Coding-Interview-Programming-Questions/dp/0984782850);
- https://www.geeksforgeeks.org/what-is-algorithm-and-why-analysis-of-it-is-important/;
- https://www.geeksforgeeks.org/introduction-to-disjoint-set-data-structure-or-union-find-algorithm/
