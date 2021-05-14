---
title: Criando um npm registry privado na AWS
date: "2021-05-13T22:12:03.284Z"
description: "Hospedando verdaccio na AWS utilizando Fargate e S3."
---

Neste post vou mostrar como hospedar um npm registry privado
na AWS, para que serve ter um registry desse tipo e como utilizá-lo.

## Para quê um npm registry privado?

Quando estamos desenvolvendo no ecossistema Javascript,
algumas vezes precisamos publicar nossos pacotes para os outros
conseguirem utilizar.

Quando o pacote é público, conseguimos utilizar NPM sem problema nenhum.
Nosso pacote estando lá, pode ser utilizado por qualquer um. Porém, se
no nosso caso for necessário criar pacotes privados, para serem utilizados,
os preços do NPM registry não são muito convidativos.

![Packages Illustration](./packages.png)

## Verdaccio

[Verdaccio](https://verdaccio.org/) é um proxy e registry privado Open Source.

## Passos para implementar

### Criar um bucket no S3

#### Criar uma imagem Docker

#### Deploy da imagem para o ECR

#### Criação da task definition

#### Rodando a task como um serviço

## Utilizando o registry privado
