---
title: Como criar um npm registry privado na AWS
description: "Hospedando verdaccio na AWS utilizando ECS e S3."
slug: verdaccio-on-aws
date: 2023-02-21 00:00:00+0000
draft: true
comments: false
image: cover.png
categories:
  - Aws
  - Frontend
tags:
  - Verdaccio
  - Elastic Container Service
---

Neste post vou mostrar como hospedar um **npm registry privado**
na AWS, explicar para que serve ter um registry desse tipo e como utilizá-lo.

## NPM Registry

Quando estamos desenvolvendo sistemas utilizando ferramentas do ecossistema Javascript,
é natural que em algum momento se torne necessário criar um ou mais pacotes reutilizáveis.

Em casos onde o pacote pode ser distribuído publicamente o
[https://www.npmjs.com/](https://www.npmjs.com/) é uma opção excelente.
Já em casos onde precisamos publicar o pacote de maneira privada, para que seja visto
somente pela empresa que o código pertence, o npmjs acaba ficando menos atrativo
por conta do preço ($7 por usuário).

O principal motivo para buscar um NPM registry próprio e privado
é para publicar e distribuir pacotes privados de domínio da empresa.
Mas, para além disso, um registry privado também pode servir para outros
casos de uso, como:

- Testes end to end (e2e) de pacotes;
- Caching de pacotes públicos;
- Customização de pacotes utilizados.

Então, e como faz para criar um NPM registry privado?

## Verdaccio

[Verdaccio](https://verdaccio.org/) é um proxy e registry privado Open Source.
O verdaccio fornece uma CLI que roda o registry privado. Além da CLI, ele também
fornece as APIs de armazenamento e gerenciamento dos pacotes, uma interface WEB
e uma série de plugins que podem ser utilizados para adequar melhor a solução
para cada caso de uso.

Neste post, vamos utilizar o Verdaccio em conjunto com o
[plugin de armazenamento no S3 da Amazon.](https://github.com/verdaccio/verdaccio/tree/master/packages/plugins/aws-storage).

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

- Ter configurado a [aws cli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html);
- Conhecer docker.

### Criar um bucket no S3

A primeira coisa a se fazer é criar um bucket. Nesse
bucket serão armazenados alguns dados de aplicação e os diretórios
dos pacotes contendo o conteúdo de distribuição (package.json e tarball do pacote).

A criação do bucket pode ser feito através da CLI da aws ou
através da interface. Utilizando a CLI, utilizamos o seguinte comando:

```sh
# Substitua {company} pelo nome da sua empresa
aws s3 mb s3://verdaccio-storage-{company}
```

Como o nome do bucket do s3 deve ser único global, substitua o trecho _{company}_ pelo
nome da empresa ou seu nome.

#### Criar uma imagem Docker do verdaccio

Para o plugin do **verdaccio** funcionar, é necessário criar uma imagem
do verdaccio que contenha o plugin de storage instalado.
Essa imagem vai ser utilizada para fazer deploy no ECS. A imagem pode
ser construida em 2 passos como demonstrado a seguir:

1. Criação do arquivo (config.yaml) de configuração do verdaccio para utilizar o plugin:

```yaml
storage: /verdaccio/storage/data

store:
  aws-s3-storage:
    bucket: verdaccio-storage-lm
    region: us-east-1
    ## !! São parâmetros opcionais, não usar em ambiente de produção !!
    ## !! Em ambientes de produção, usar roles na AWS
    accessKeyId: # Sua access key
    secretAccessKey: # Seu secret

web:
  title: Sua empresa Proxy Registry

uplinks:
  npmjs:
    url: https://registry.npmjs.org/

packages:
  "**":
    access: $all
    publish: $all
    # if package is not available locally, proxy requests to 'npmjs' registry
    proxy: npmjs

server:
  keepAliveTimeout: 60

logs: { type: stdout, format: pretty, level: http }
```

2. Criação da imagem do docker que vai ser utilizada no deploy:

```Dockerfile
FROM verdaccio/verdaccio:5

ADD config.yaml /verdaccio/conf/config.yaml
USER root
ENV NODE_ENV=production
RUN npm install --global verdaccio-s3-storage
USER verdaccio
```

Depois de criar o **Dockerfile**, precisamos fazer build da imagem.

```sh
docker build . -t verdaccio-with-s3
```

#### Deploy da imagem para o ECR

Depois de ter criado a imagem, é necessário fazer o deploy
dela no AWS ECR. Para isso, precisamos criar um
repositório de imagens Docker no AWS ECR.

Nesse caso também é possível usar a CLI ou a interface, fica
a critério de afinidade em como fazer essa criação.

**É importante copiar o campo "repositoryUri" do output**

```sh
aws ecr create-repository --repository-name verdaccio-with-s3
```

O passo seguinte é adicionar a tag com o caminho do repositório
para nossa imagem docker. (subistitua a URL do repositório que foi criado).
**Use a URL do repositório que foi criado no passo anterior**.

```sh
docker tag verdaccio-with-s3:latest [aws_account_id].dkr.ecr.us-east-1.amazonaws.com/verdaccio-with-s3:latest
```

Depois de criado o repositório, podemos fazer deploy
da imagem. Para isso, precisamos também fazer login na AWS e isso é
feito com os seguintes comandos:

**Não esqueça de substituir a região e sua aws_account_id** e a região se necessário

```sh
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin [aws_account_id].dkr.ecr.us-east-1.amazonaws.com
```

```sh
docker push [aws_account_id].dkr.ecr.us-east-1.amazonaws.com/verdaccio-with-s3:latest
```

#### Criação da task definition

Após a imagem estar publicada no nosso ECR e o Bucket
no S3 criado para armazenar os dados do verdaccio, é hora de criar
nosso cluster e nossa task definition para ser rodada pelo ECS Fargate.

Para criar o cluster no Fargate, usamos o seguinte comando:

```sh
aws ecs create-cluster --cluster-name verdaccio-cluster
```

Para conseguir criar a definição de task no ECS, é necessário antes criar uma role com policies
que permitam que essa task tenha acesso a imagem que adicionamos ao ECR. Os comandos a seguir
fazem com que seja criada a role para ser utilizada pela task.

Criação do arquivo (**iam-assume-policy.json**) da política de utilização da Role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ecs.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Criação da role (lembre-se de copiar o ARN para adicionar no arquivo da task):

```bash
aws iam create-role --role-name verdaccio-task --assume-role-policy-document file://iam-assume-policy.json
```

Necessário associar a policy para dar permissão de acessar os recursos do ECR. No exemplo, para fins de demonstração
estou usando uma policy mais permissiva, porém em ambientes de produção deve-se utilizar o princípio de
menos privilégio possível, atribuindo somente o necessário ou criando uma policy específica para a task.

```sh
aws iam attach-role-policy --role-name verdaccio-task --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess
```

Com o cluster e role criados, o próximo passo é criar uma definição de task para ser rodada como um serviço pelo Fargate.
A definição da tarefa é feita utilizando um arquivo JSON com as infromações necessárias
para rodar.

O arquivo deve conter a seguinte configuração. (Lembre-se de substituir a account_id e se necessário a região também)

```yaml
# ecs-task.yaml
family: verdaccio-service-task
executionRoleArn: "arn:aws:iam::[aws_account_id]:role/verdaccio-task"
networkMode: awsvpc
containerDefinitions:
  - name: verdaccio
    image: [aws_account_id].dkr.ecr.us-east-1.amazonaws.com/verdaccio-with-s3:latest
    cpu: 512
    memory: 1024
    portMappings:
      - containerPort: 4873
        hostPort: 4873
        protocol: tcp
requiresCompatibilities:
  - EC2
  - FARGATE
cpu: "512"
memory: "1024"
runtimePlatform:
  cpuArchitecture: ARM64
  operatingSystemFamily: LINUX
```

Essa definição cria uma Task que exige 0.5vCPU e 1GB de memória RAM. Caso
você sinta necessidade, ou o verdaccio ficar lento, esses parâmetros podem ser alterados
para escalar e atender melhor sua necessidade.

Nessa definição, também é indicado um port mapping, da porta 4873 para a porta
do container 4873 que é a porta padrão que o verdaccio expõe sua API. Também é possível
alterar para outra porta caso seja de interesse, como por exemplo 80/443.

Tendo isso, o comando que registra a task é:

```sh
aws ecs register-task-definition --cli-input-yaml file://ecs-task.yaml
```

#### Criando o serviço do verdaccio usando a task

## Utilizando o registry privado

Para testar a utilização do nosso registry privado, vamos
criar um pacote com o prefixo configurado, para configurar o namespace
e publicá-lo.

Para criar o pacote vamos utilizar os seguintes comandos:

```sh
mkdir my-pkg
cd my-pkg
npm init -y

// Alterar o nome do pacote
```

Vamos alterar o nome do pacote para ele casar com a
configuração verdaccio.

```sh
// Alterar o arquivo .npmrc para configurar o proxy
```

### Publicando um pacote

```bash
npm publish
```

## Conclusão

Criar uma instância do verdaccio manualmente utilizando ECS é um tanto trabalhoso. Porém algumas vezes atende
a demanda. O proxy com certeza é muito útil em diversos casos de uso.

Em um futuro próximo vou publicar um projeto em terraform para que o deploy seja simplificado
e possa ser mais facilmente replicado.

## Referências

- https://verdaccio.org/
