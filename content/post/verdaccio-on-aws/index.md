---
title: Criando um npm registry privado na AWS
description: "Hospedando verdaccio na AWS utilizando ECS Fargate e S3."
slug: verdaccio-on-aws
date: 2023-01-05 00:00:00+0000
draft: true
comments: false
categories:
  - Aws
  - Frontend
tags:
  - Verdaccio
  - Elastic Container Service
---

Neste post vou mostrar como hospedar um **npm registry privado**
na AWS, para que serve ter um registry desse tipo e como utilizá-lo.

## NPM Registry

Quando estamos desenvolvendo sistemas utilizando ferramentas do ecossistema Javascript,
é natural que em algum momento se torne necessário criar um ou mais pacotes reutilizáveis.

Em casos onde o pacote pode ser distribuído publicamente o
[https://www.npmjs.com/](https://www.npmjs.com/) é uma opção excelente.
Já em casos onde precisamos publicar o pacote de maneira privada, o
npmjs acaba ficando menos atrativo, principalmente pelo preço ($7 por usuário).

Um dos principais motivos para buscar um NPM registry próprio e privado
é para publicar e distribuir pacotes privados de domínio da empresa.

Além disso, um registry privado também pode ajudar em m alguns outros
casos de uso, como:

- Testes end to end (e2e);
- Caching de pacotes;
- Customização de pacotes.

Então, como faz para criar um NPM registry privado?

![Packages Illustration](./packages.png)

## Verdaccio

[Verdaccio](https://verdaccio.org/) é um proxy e registry privado Open Source.
O verdaccio fornece uma CLI que roda o registry privado. Além da CLI com as APIs
de armazenamento e gerenciamento dos pacotes, ele também fornece uma interface WEB
e uma série de plugins que podem ser utilizados para adequar melhor a solução
para cada caso.

Neste post, vamos utilizar o Verdaccio com o
[plugin de armazenamento do S3 da Amazon.](https://github.com/verdaccio/verdaccio/tree/master/packages/plugins/aws-storage)

Adicionar o registry online é útil quando o time está distribuído e
precisa que este registry esteja acessível pela internet ou em uma rede privada.

## Passos para implementar

Os seguintes passos vão ser implementados:

1. Criar um bucket no S3;
2. Criar uma imagem docker do verdaccio com o plugin;
3. Fazer deploy da imagem no ECR;
4. Criar uma IAM Role para a task definition;
5. Criar uma task definition;
6. Rodar a task como um serviço dentro do cluster.

### Pré-requisitos

- Configurar [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html).

### Criar um bucket no S3

A primeira coisa a se fazer é criar um bucket. Nesse
bucket serão armazenados alguns dados de aplicação e os diretórios
dos pacotes contendo o conteúdo de distribuição (package.json e tarball do pacote).

A criação do bucket pode ser feito através da CLI da aws ou
através da interface. Utilizando a CLI, utilizamos o seguinte comando:

```bash
# Substitua {company} pelo nome da sua empresa
aws s3 mb s3://verdaccio-storage-{company}
```

Como o nome do bucket do s3 deve ser único global, substitua o trecho _{company}_ pelo
nome da empresa ou seu nome.

#### Criar uma imagem Docker do verdaccio

Para o plugin do **verdaccio** funcionar, é necessário criar uma imagem
do verdaccio que contenha o plugin de storage instalado.
Essa imagem vai ser utilizada para fazer deploy no ECS Fargate.

```Dockerfile
FROM verdaccio/verdaccio

USER root
ENV NODE_ENV=production
RUN npm i && npm install verdaccio-s3-storage
USER verdaccio
```

Depois de criar o **Dockerfile**, precisamos fazer build da imagem.

```bash
docker build . -t verdaccio-with-s3
```

#### Deploy da imagem para o ECR

Depois de ter criado a imagem, é necessário fazer o deploy
dela no AWS ECR. Para isso, precisamos criar um
repositório de imagens Docker no AWS ECR.

Nesse caso também é possível usar a CLI ou a interface, fica
a critério de afinidade em como fazer essa criação.

**É importante copiar o campo "repositoryUri" do output**

```bash
aws ecr create-repository --repository-name verdaccio-with-s3
```

O passo seguinte é adicionar a tag com o caminho do repositório
para nossa imagem docker. (subistitua a URL do repositório que foi criado).

```bash
docker tag verdaccio-with-s3:latest [aws_account_id].dkr.ecr.us-east-1.amazonaws.com/verdaccio-with-s3:latest
```

Depois de criado o repositório, podemos fazer deploy
da imagem. Para isso, precisamos também fazer login na AWS e isso é
feito com os seguintes comandos:

**Não esqueça de substituir a região e sua aws_account_id** e a região se necessário

```bash
aws ecr get-login-password --region us-east-1 |
  docker login --username AWS --password-stdin [aws_account_id].dkr.ecr.us-east-1.amazonaws.com
```

```bash
docker push [aws_account_id].dkr.ecr.us-east-1.amazonaws.com/verdaccio-with-s3:latest
```

#### Criação da task definition

Após a imagem estar publicada no nosso ECR e o Bucket
no S3 criado para armazenar os dados do verdaccio, é hora de criar
nosso cluster e nossa task definition para ser rodada pelo ECS Fargate.

Para criar o cluster no Fargate, usamos o seguinte comando:

```bash
aws ecs create-cluster --cluster-name verdaccio-cluster
```

Com o cluster criado, o próximo passo é criar uma definição de task para ser rodada como um serviço pelo Fargate.
A definição da tarefa é feita utilizando um arquivo JSON com as infromações necessárias
para rodar.

O arquivo deve conter a seguinte configuração.

```json
{
  "family": "verdaccio-fargate",
  "networkMode": "awsvpc",
  "containerDefinitions": [
    {
      "name": "verdaccio",
      "image": "verdaccio-with-s3:latest",
      "portMappings": [
        {
          "containerPort": 4873,
          "hostPort": 4873,
          "protocol": "tcp"
        }
      ],
      "essential": true
    }
  ],
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512"
}
```

Essa definição cria uma Task que exige 0.25vCPU e 512MB de memória RAM. Caso
você sinta necessidade, ou o verdaccio ficar lento, esses parâmetros podem ser alterados
para atender sua necessidade.

Nessa definição, também é indicada um port mapping, da porta 4873 para a porta
do container 4873 que é a porta que o verdaccio expõe sua API.

Tendo isso, o comando que registra a task é:

```bash
aws ecs register-task-definition --cli-input-json ./fargate-task.json
```

#### Rodando a task como um serviço

## Utilizando o registry privado

Para testar a utilização do nosso registry privado, vamos
criar um pacote com o prefixo configurado, para configurar o namespace
e publicá-lo.

Para criar o pacote vamos utilizar os seguintes comandos:

```bash
mkdir my-pkg
cd my-pkg
npm init -y

// Alterar o nome do pacote
```

Vamos alterar o nome do pacote para ele casar com a
configuração verdaccio.

```bash
// Alterar o arquivo .npmrc para configurar o proxy
```

#### Publicando um pacote

```bash
npm publish
```

## Conclusão
