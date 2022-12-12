---
title: Otimização da entrega de imagens usando CloudFront com base no userAgent
description: Aprenda como entregar imagens mais otimizadas para os dispositivos do usuário utilizando CloudFront Functions
slug: cloudfront-function-image-sizes
date: 2022-12-17 00:00:00+0000
draft: false
comments: false
image: cover.svg
categories:
  - Aws
tags:
  - Cloudfront
  - Optimization
  - Content Delivery
---

Se suas aplicações não são API-only e possue uma interface web, seguramente você utiliza direta ou indiretamente algum CDN para distribuir arquivos para o navegador. Na AWS temos disponível um serviço gerenciado de CDN chamado **Cloudfront** e uma das funcionalidades que esse serviço é o **Cloudfront Function**. Essa funcionalidade é bastante útil e nesse post vou mostrar como implementar uma otimização de entrega de imagens para sua aplicação utilizando essa funcionalidade do Cloudfront.

## O que são Cloudfront Functions?

Cloudfront Function é uma funcionalidade do Cloudfront que permite executar funções javascript leves nas _edge locations_. Através dessa função é possível manilupar aspectos da requisição e resposta do Cloudfront como adicionar headers e trocar a URI. E ela faz isso de maneira bastante performática (tem alta escala e baixa latência) por ser apenas uma função js leve.

Casos de uso comuns para a utilização do Cloufront functions são:

- Manipulação e normalização das chaves do cache;
- Rewrite e redirect de URLs;
- Manipulação de cabeçalhos;
- Autorização de acesso.

### Diferença em relação à Lambda@edge

O Cloudfront também tem uma funcionalidade parecida que se chama **Lambda@edge**. Porém elas são diferentes e tem casos de uso bastante diferentes também e é importante saber quando se utiliza uma ou a outra.

## Otimizando a entrega de imagens

Considerando a funcionalidade apresentada, uma otimização que pode ser feita no seu ambiente é retornar diferentes tamanhos e resoluções de imagens com base no dispositivo do usuário. Ou seja, entregar imagens otimizadas para celular, desktop ou tablet utilizando as _Cloudfront functions_. Isso permite que você tenha uma redução de custos e melhorea a experiência do usuário.

Vamos pensar no seguinte cenário...

### Referências

- https://aws.amazon.com/pt/blogs/aws/introducing-cloudfront-functions-run-your-code-at-the-edge-with-low-latency-at-any-scale/
- https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-functions.html
