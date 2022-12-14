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

O Cloudfront também tem uma funcionalidade parecida que se chama **Lambda@edge**. A Lambda@edge foi criada em 2017, alguns anos antes que a Lambda Functions. As 2 funcionalidades são diferentes e tem casos de uso diferentes também, por isso é importante saber quando se utiliza uma ou a outra.

As pricipais diferenças entre as 2 funcionalides são as seguintes:

| -           | Cloudfront Function          | Lambda@Edge                   |
| ----------- | ---------------------------- | ----------------------------- |
| Runtime     | Apenas JS                    | Javastript e Python           |
| Computação  | Baseada em processo          | Baseada em VM                 |
| Performance | Alta escala e Baixa latência | Milhares de req/s             |
| Localização | Edge (218+) lugares          | Edge regional (13)            |
| Acessos     | Nenhum                       | Serviços aws, filesystem, etc |
| Req. Body   | Não                          | Sim                           |

Vendo essa tabela fica mais claro quando usamos uma ou outra. No caso usaremos o _cloudfront function_ quando queremos alguma manipulação simples de URI, header ou algum aspecto dessa parte da requisição e usaremos Lambda@edge quando precisarmos fazer algo mais elaborado e precisarmos acessar o body da requisição ou então outros serviços da AWS.

Vale ressaltar que as 2 podem ser utilizadas em conjunto, nada impede esse caso de uso.

## Otimizando a entrega de imagens

Considerando a funcionalidade apresentada, uma otimização que pode ser feita no seu ambiente é retornar diferentes tamanhos e resoluções de imagens com base no dispositivo do usuário. Ou seja, entregar imagens otimizadas para celular, desktop ou tablet utilizando as _Cloudfront functions_. Isso permite que você tenha uma redução de custos e melhorea a experiência do usuário.

Vamos pensar no seguinte cenário, você tem um portal que tem várias imagens de fundo e imagens. Nesse cenário você quer entregar as imagens sempre em tamanho otimizado para o dispositivo. Em desktops você não quer que a imagem fique pixelada e em mobile você não quer que consuma muita rede e transferência de dados do cloudfront. Para não precisar tratar dentro da aplicação, é possível criar uma função que faça isso na edge location reescrevendo a URI redirecionando para a imagem correta.

A função do cloudfront function associada com sua distribuição poderia ser escrita da seguinte forma:

{{< gist leometzger 43afab26cf57f6e7d239e01042f5837c >}}

Com esse exemplo de código, caso o dispositivo (user-agent) do usuário seja mobile, a função irá reescrever a URI para adicionar `_mobile` no nome da imagem. **É importante que se tenha um processo na hora de salvar na origem do cloudfront para suportar essa funcionalidade.**

![Comportamento utilizando a CF Function](example-function.png "Exemplo visual gerado")

## Conclusão

Criar otimizações desse tipo melhoraram a experiência do usuário por consumir menos rede e deixar sua aplicação mais rápida. Além disso, também pode gerar uma redução de custos, dependendo do quão diferente são os tamanhos das imagens, visto que na AWS pagamos pela transferência de dados e o custo de execução dessas funções é relativamente baixo. O cloudfront permite que as aplicações fiquem mais performáticas e menos custosas, conhecer as funcionalides desse serviço é fundamental para quem desenvolve na AWS.

### Referências

- https://aws.amazon.com/pt/blogs/aws/introducing-cloudfront-functions-run-your-code-at-the-edge-with-low-latency-at-any-scale/
- https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/cloudfront-functions.html
