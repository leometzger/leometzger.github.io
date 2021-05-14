---
title: Criando um npm registry privado na AWS
date: "2021-05-13T22:12:03.284Z"
description: "Hospedando verdaccio na AWS utilizando Fargate e S3."
---

Neste post vou mostrar como hospedar um npm registry privado
na AWS, para que serve ter um registry desse tipo e como utilizá-lo.

## NPM Registry

Quando estamos desenvolvendo no ecossistema Javascript,
é natural que em algum momento se torne necessário criar uma
biblioteca e distribuí-la através de um pacote.

Em casos onde o pacote pode ser distribuído publicamente o
[https://www.npmjs.com/](https://www.npmjs.com/) é uma opção excelente.
Já em casos onde precisamos publicar o pacote de maneira privada, o
npmjs acaba ficando menos atrativo, principalmente pelo preço ($7 por usuário).

Um dos principais motivos para buscar um NPM registry próprio e privado
é para publicar e distribuir pacotes privados.

Além disso um registry privado também pode ajudar com alguns outros
casos de uso, como:

- Testes end to end;
- Caching de pacotes;
- Customização de pacotes.

Então, como criar um NPM registry privado para distribuir os pacotes?

![Packages Illustration](./packages.png)

## Verdaccio

[Verdaccio](https://verdaccio.org/) é um proxy e registry privado Open Source.
O verdaccio fornece uma CLI que roda o registry privado. Além da CLI com as APIs
de armazenamento e gerenciamento dos pacotes, ele também fornece uma interface WEB
e uma série de plugins que podem ser utilizados para adequar melhor a solução
para cada caso.

Nesse post, vamos utilizar o Verdaccio com o
[plugin de armazenamento do S3 da Amazon.](https://github.com/verdaccio/verdaccio/tree/master/packages/plugins/aws-storage)

Que é útil quando o time está distribuído e não precisa
desse registry na internet ou em uma rede privada.

## Passos para implementar

Os seguintes passos vão ser implementados, inicialmente passo a passo e ao final
terá um script de provisionamento dessa infraestrutura que será montada no passo
a passo.

A sequência de passos será:

1. Criar um bucket no S3;
2. Criar uma imagem docker do verdaccio com o plugin;
3. Fazer deploy da imagem no ECR;
4. Criar uma task definition;
5. Rodar a task como um serviço.

### Criar um bucket no S3

A primeira coisa a se fazer é criar um bucket. Nesse
bucket serão armazenados alguns dados de aplicação, os diretórios
dos pacotes. Dentro dos diretórios serão armazenados
as tarballs dos arquivos dos pacotes e o package.json dele.

A criação do bucket pode ser feito através da CLI da aws ou
através da interface. Utilizando a CLI, utilizamos o seguinte comando:

```bash
aws s3 mb verdaccio-storage
```

#### Criar uma imagem Docker do verdaccio

Para o plugin funcionar, é necessário criar uma imagem
do verdaccio que contenha o plugin de storage instalado.
Essa imagem vai ser utilizada para fazer deploy no Fargate.

```Dockerfile
FROM verdaccio/verdaccio

USER root
ENV NODE_ENV=production
RUN npm i && npm install verdaccio-s3-storage
USER verdaccio
```

Além de criar o arquivo docker, precisamos fazer build da imagem.

```bash
docker build . -t verdaccio-with-s3
```

#### Deploy da imagem para o ECR

Agora o deploy para uma imagem do Docker no ECR.
Primeiramente é necessário criar um
repositório de imagens Docker na AWS.

Nesse caso também é possível usar a CLI ou a interface, fica
a critério de afinidade em como fazer essa criação.

```bash
aws ecr create-repository --repository-name verdaccio-docker
```

O passo seguinte é adicionar a tag com o caminho do repositório
para nossa imagem docker. (subistitua a URL do repositório que foi criado).

```bash
docker tag verdaccio-with-s3:latest [URL Repositório ECR]/verdaccio-with-s3:latest
```

Depois de criado o repositório, podemos fazer deploy
da imagem

```bash
docker push  [URL Repositório ECR]/verdaccio-with-s3:latest
```

#### Criação da task definition

#### Rodando a task como um serviço

## Utilizando o registry privado

#### Publicando um pacote

## Conclusão
