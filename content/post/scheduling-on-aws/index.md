---
title: Opções para criar agendamentos de tarefas na AWS
description: Descubra diferentes maneiras de criar agendamentos de tarefas na AWS e qual é o mais apropriado para seu caso de uso
slug: scheduling-on-aws
date: 2023-01-07 00:00:00+0000
draft: true
comments: false
image: cover.svg
categories:
  - Aws
  - Architecture
tags:
  - Scheduling
  - Cloudwatch
  - Elastic Container Service
---

Em diversos tipos de sistemas é comum existirem funcionalidades que devem ser executadas de maneira agendada. Ou então
rotinas ou tarefas que precisam ser executadas de tempos em tempos para o
bom funcionamento do sistema como um todo.

Nesse post vou mostrar algumas opções quando se trata de criar esses agendamentos utilizando o ambiente da
AWS. Também vou comparar essas opções em termos de casos de uso, integrações e preço.

## Agendamento de tarefas

A ideia de ter um agendamento de tarefas é definir quando, onde e como vai ser executada
determinada tarefa. Normalmente na AWS os agendamentos são definidos baseados em **_rate_** (ex: a cada 10 minutos)
ou **_cron_** (ex: segundas as 8 da manhã, primeiro dia do mês as 5), ficando a critério do usuário escolher qual o atende melhor.

### 1. ECS Task Scheduler

Com o serviço Elastic container service (ECS) é possível rodar _Tasks_ agendadas utilizando um scheduler do próprio serviço.
A _task_ no contexto do ECS é um container docker que vai ser executado no horário configurado.

Esse tipo de agendamento é interessante para quando o próprio sistema executa no ECS, pois facilita a integração com
os outros containers da aplicação e possívelmente pode rodar no mesmo cluster.

### 2. Cloudwatch Event Bridge Event Bus

O Cloudwatch Event Bus, é uma maneira classica de rodar agendamentos na AWS. Os agendamentos são feitos através
das regras de um Bus.

Com a regra do Event Bus a execução da tarefa é um pouco diferente. A configuração da regra possui um _target_, o qual é invocado quando chegar a hora do agendamento.
Esse _target_ pode ser uma fila, um tópico SNS, uma lambda ou até mesmo uma ECS task. A lista de serviços AWS integrados
com o event bus é grande, o que traz muita flexibilidade para o desenvolvimento.

Essa maneira muito fácil e versátil de criar a funcionalidade. Porém temos que ficar atentos aos limites. É possível criar 300 regras
por event bus. Então se for tarefas fixas do sistema, provável que atenderá a demanda. Porém se a necessidade for algo
criado por usuários, ou algo mais dinâmico, o sistema pode esbarrar nesse limite.

### 3. Cloudwatch Event Bridge - Managed Schedules

### 4. AWS Batch

## Análise

### Intergrações

### Pricing

## Conclusão
