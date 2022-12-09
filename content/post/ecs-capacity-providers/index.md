---
title: ECS Capacity Providers
description: Conheça os capacity providers e aprenda o que é possível fazer com eles
slug: ecs-capacity-providers
date: 2022-12-11 00:00:00+0000
draft: false
comments: false
categories:
  - Aws
tags:
  - Docker
  - Elastic Container Service
---

Você já utilizou o serviço ECS da AWS? Se já utilizou, possívelmente já conhece os capacity providers em algum nível e sabe como eles funcionam. Nesse post, vou explorar um pouco do que aprendi sobre eles e mostrar algumas coisas interessantes que são possíveis de implementar utilizando esse orquestrador de containers.

## O que é o Elastic Container Service (ECS)?

O ECS é um serviço gerenciado da AWS de orquestração de containers, ele é totalmente gerenciado e como todo serviço gerenciado tira muito da sobrecarga operacional.

Esse serviço é considerado mais simples que o EKS (Elastic Kubernetes Service). Embora seja mais simples de operacionalizar, ele não deixa de ser um serviço poderoso e que entrega diversas funcionalidades semelhantes as do EKS.

Dentro do contexto do ECS é importante conhecer bem a definição dos seus principais componentes:

- **Containers & Images**: As imagens dos containers são os componentes ou serviços que a aplicação precisa rodar em algum ambiente. Essas imagens precisam estar hospedadas em algum docker registry (Docker hub, ECR, Etc.);
- **Cluster**: É um agrupamento lógico de serviços e tarefas. Esse agrupamento serve para isolar os containers da aplicação. A infraestrutura que o cluster gerencia (seja serverless ou não) é responsável por botar os containers para rodar;
- **Tasks Definition**: É a especificação de um components composto por um ou mais containers que fazem parte da sua aplicação;
- **Task**: É uma instância de uma task definition que devem ser rodadas pelo _cluster_. Fazendo uma analogia com POO, a _task definition_ é uma classe enquanto a task é um objeto;
- **Services**: Serviços são as tasks da aplicação que devem rodar continuamente, como uma API ou uma web application. Quando parados por algum motivo como erro ou algo do tipo eles são automaticamente iniciadas novamente pelo ECS.

Tá, e onde entram os capacity providers nesse esquema? Primeiro, temos que entender o que são os capacity providers e como funcionam.

## Como funcionam os capacity providers?

_Capacity provider_ a grosso modo é uma especificação de provisionamento da infraestrutura do cluster. Como o próprio nome sugere é a definição de como vai ser criada a capacidade computacional que um cluster precisa para rodar as _Tasks_ e _Services_. Um cluster pode ter um ou mais _capacity providers_ e utilizá-los da meneira que atender melhor os objetivos da aplicação. (Vai ficar mais claro ao ver as estratégias de utilização dos capacity providers)

Quando você está criando um _capacity provider_, tem a opção de escolher um capacity provider Fargate ou EC2. A seguir entenda a diferença entre os 2.

### EC2

No modelo utilizando instâncias EC2 você escolhe o(s) tipo(s) de instância(s) que você gostaria de utilizar e associa um autoscaling group com esse capacity provider. Normalmente é mais interessante utilizar o autoscaling que a própria amazon recomenda utilizar e não configurar nada muito mirabolante, apenas as configurações básicas das instâncias como AMI, família e etc.

### Fargate

Já com fargate, o gerenciamento de instâncias não é necessário. O Fargate é uma maneira serverless de rodar as tasks e containers. Muito interessante se busca simplicidade, embora seja um pouco mais caro do que no modelo com instâncias EC2.

Com ele basta especificar a quantidade de memória e vCPU que a task ou serviço irá utilizar e mandar rodar.

![With Fargate vs without fargate](fargate.jpeg "Fargate vs EC2 provisioning")

### (Fargate | EC2) Spots

Em ambos os modelos, caso seja de interesse buscar uma economia de custos é possível utilizar a versão Spot (EC2 Spot ou Fargate Spot).

A ideia do modelo _Spot_ tem a mesma ideia para as instâncias EC2 e para o Fargate. A aplicação/instância não tem garanteia de que continuidade da alocação. Em determinado momento a instância ou espaço no caso do fargate pode ser requerido por outra pessoa e sua aplicação perder acesso aquele recurso.

Porém, na prática isso não acontece tão frequentemente a ponto de ser uma preocupação para as aplicações que não tem criticidade em relação a isso.

Uma coisa que é muito recomendado quando utiliza spot para as instâncias EC2 é que sejam selecionadas várias famílias de instâncias. Quanto mais famílias forem utilizadas, menor a chance de se ter uma interrupção do serviço que está rodando nesse modelo.

## Estratégias de utilização dos capacity providers

Dito tudo isso, agora vamos para o que interessa que são as estratégias de criação e como tirar proveito de um bom uso dos _capacity providers_.

**E é aqui que as coisas ficam realmente interessantes**

Quando vai rodar uma task ou um service é possível utilizar a estratégia de _capacity providers_. Essa estratégia pode ser a default configurada para o cluster ou então uma customizada.

### Definição de uma estratégia

Ao definir uma estratégia de _capacity providers_ você tem que escolher qual ou quais providers você quer utilizar e definir alguns parâmetros caso seja selecionado mais de 1 que são:

- **Base**: Que é o número de tasks que certamente vão ser utilizados no capacity provider configurado com Base > 0;
- **Weight**: O peso considerado para cada _capacity provider_ ao escolher onde rodar a task.

#### Exemplo

Uma estratégia que tenha 2 _capacity providers_ (CP1 e CP2) onde:

- CP1 tenha base 2 e weight 1
- CP2 tenha base 0 e weigth 1

Nesse cenário ao mandar provisionar 6 tasks o resultado é que as tasks sejam provisionadas da seguinte forma:

- CP1: 4 tasks (task 1,2,3,5)
- CP2: 2 tasks (task 4 e 6)

Ou seja, o peso é considerado depois que a base já foi preenchida e em caso de pesos iguais a ordem dos providers importa.

![Exemplo provisionamento de tarefas](ex1.png "Política 1 exemplo")

### Spots Capacity provider

### Misturando provider Spot com provider On Demand

### Somente On Demand

## Informações adicionais

- Possível ter entre 0-10 capacity providers associados com um cluster;
- Na _task definition_ precisa ser selecionado a compatibilidade com o fargate/ec2 launch type e ela tem que ser compatível com o tipo do capacity provider;
- A interface nova no console da AWS não tem uma maneira de gerenciar os capacity providers direito ainda (nesse momento de escrita).

## Conclusão

As aplicações estão em constante evolução e a containerização delas já é uma realidade há algum tempo. Saber aproveitar os recursos da AWS da melhor maneira possível tem um grande valor para as empresas e os capacity providers são uma peça importante para quem utiliza o serviço ECS da AWS. Além disso aproveitar as opções de provisionamento da melhor forma possível permite uma redução de custos significativa e faz com que seus gastos com AWS sejam otimizados.

### Referências

- https://aws.amazon.com/pt/ecs/
- https://docs.aws.amazon.com/AmazonECS/latest/developerguide/cluster-capacity-providers.html
- https://aws.amazon.com/pt/blogs/containers/deep-dive-on-amazon-ecs-cluster-auto-scaling/
- https://docs.aws.amazon.com/AmazonECS/latest/developerguide/welcome-features.html

Apresentações por Adam Keller e Amazon:

- https://www.youtube.com/watch?v=Fb1EwgfLbZA
- https://www.youtube.com/watch?v=Vb_4wAEcfpQ
