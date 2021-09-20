# Aprendendo_setup-Docker-TS-Node

## Para excutar localmente
- Instale os modulos do projeto com:
```console
    npm i
```
- Inicie o servidor Dev:
```console
    npm run dev
```
- Faça build do projeto:
```console
    npm run build
```
- Execute o projeto:
```console
    npm start
```
## Para executar o Container Docker
> *É necessario ter o o Docker já instalado.*
- Faca build da imagem: (necessario apenas na primeira utilização ou caso imagem não esteja presenta na maquina)

```console
    docker-compose build
```
- Inicie o servidor Dev:
```console
    make up
```
- Pare o servidor Dev:
```console
    make down
```

Este passo a passo foi elaborado para fins de aprendizado e pode ser usado como template para o setup de projetos que façam uso de docker, typescript, e node.
Ele foi feito conforme estudos e tradução <a href = "https://dev.to/dariansampare/setting-up-docker-typescript-node-hot-reloading-code-changes-in-a-running-container-2b2f">deste artigo.</a> elaborado por Darian Sampare. Seu perfil pode ser encontrado <a href = "https://github.com/justDare">aqui.</a>
Também possível acessar o repositorio do autor <a href = "https://github.com/justDare/TypeScript-Node-Docker">aqui</a>

## Passo 1: Criar um servidor com Typescript e Express
- Faça um diretorio para o projeto e entre nele:
```console
    mkdir ts-node-docker
    cd ts-node-docker
```

- Inicialize um projeto node de acordo com suas preferências:
```console
    npm init
```

- Instale Typescript como uma dependência dev:
```console
    npm i Typescript --save-dev
```

- Ao término do download crie um arquivo tsconfig.json:
```console
    npx tsc --init
```
- Agora com o tsconfig.json criado na raiz do projeto (root) acrescente as seguintes entradas:
```console
    "baseUrl": "./src"
    "target": "esnext"
    "moduleResolution": "node"
    "outdir": "./build"
```
Algumas coisas a se notar:
O **baseUrl** nos conta onde o nosso codigo fonte (source code) *.ts* está localizado. Ou seja na pasta *./src*.

O **target** indica qual target de Javascript deve ser emitido a partir do Typescript fornecido. Para esse caso está como *esnext*.

O **moduleResolution** *deve* ser *setado* em node para projetos em node. Ele especifica a estratégia de modulo de resolução entre Node e Classic. Para mais detalhes confira <a href = "https://www.typescriptlang.org/docs/handbook/module-resolution.html">este link</a>

O **outdir** conta ao TS aonde colocar o código de Javascript compilado quando os arquivos de TS são compilados.

- Agora instale o <a href = "https://expressjs.com/pt-br/">express</a> e suas tipificações como dependências dev:
```console
    npm i --save express
    npm i -D @types/express
```
Agora que o código está pronto para ser executado no servidor. Cria uma pasta *./src* na raiz do projeto e adicione um arquivo *index.ts*
Dentro do *index.ts* acrescente o seguinte código:
```Typescript
    import express from 'express';

    const app = express();
    app.listen(4000, () => {
        console.log(`server running on port 4000`);
    });
```
- Para manter o código em execução e observando por mudança utilizamos o <a href = "https://www.npmjs.com/package/nodemon">nodemon</a>.
Para isso utilizamos o <a href = "https://www.npmjs.com/package/ts-node">ts-node</a> e <a href = "https://www.npmjs.com/package/nodemon">nodemon</a>.
```console
    npm i -D ts-node nodemon
```
Para quem gosta de configurar o nodemon em um arquivo config, é possível criar um arquivo nodemon.json na raiz do projeto. Para este template foram utilizadas as seguintes opções:
```JSON
    {
        "verbose": true,
        "ignore": [],
        "watch": ["src/**/*.ts"],
        "execMap": {
            "ts": "node --inspect=0.0.0.0:9229 --nolazy -r ts-node/register"
        }
    }
```
Onde os elemntos chaves são:
**watch** que indica quais arquivos o nodemon deve observar.

E a opção **ts** no **execMap** que conta como o nodemon deve tratar arquivos TS. Para este caso, eles são executados com o node, jogados (throw) em algumas flags de debugg e no register ts-node.

- Agora adicionamos scripts ao package.json que utilizam nodemon para iniciar o projeto. Adicione o seguinte trecho de código:
```JSON
    "scripts": {
        "start": "NODE_PATH=./build node build/index.js",
        "build": "tsc -p .",
        "dev": "nodemon src/index.ts",
    }
```
Aqui o comando **dev** irá iniciar o projeto com nodemon. 

O comando **build** compila o código em JavaScript.

E o comando **start** executa o projeto construido.

O **NODE_PATH** indica para a aplicação construida onde a raiz do projeto se localiza

- Agora é possível executar a aplicação com:
```console
    npm run dev
```
Com tudo isso pronto, a aplicação já está pronta para ser *dockerizada*.

## Passo 2: Preparando Docker para Desenvolvimento e Produção.
Para a realização deste passo, é necessário que o Docker já esteja instalado na máquina. Para mais informações acesse <a = href = "https://docs.docker.com/get-docker/">este site</a>

- Agora vamos adicionar um *Dockerfile* ao diretório raiz do nosso projeto e acrescentar o seguinte trecho de código:

```DOCKERFILE
    FROM node:14 as base

    WORKDIR /home/node/app

    COPY package*.json ./

    RUN npm i

    COPY . .
```
O que esse código faz é o seguinte:
- Conta qual a imagem de container será utilizada. (Ver lista completa de imagens <a href = "https://hub.docker.com/search?type=image">aqui</a>)
- *Seta* (Define) o nome para o diretorio de trabalho (Pasta para onde o app será copiado).
- Copia o package.json para a raiz do diretório de trabalho.
- Instala todas as dependências.
- Copia todos os arquivos para a raiz do diretório de trabalho.

Opcionalmente para executar o passo de Produção no mesmo arquivo acrescente o seguinte trecho:

```DOCKERFILE
    FROM base as production

    ENV NODE_PATH=./build

    RUN npm run build
```

Note que não foi adicionado nenhum comando para executar a build de desenvolvimento ou produção, é para isso que serve o arquivo *docker-compose*.

- Crie a na raiz do diretório o arquivo *docker-compose.yml* e adicione o seguinte:
```yml
    version: '3.7'

    services:
    ts-node-docker:
        build:
        context: .
        dockerfile: Dockerfile
        target: base
        volumes:
        - ./src:/home/node/app/src
        - ./nodemon.json:/home/node/app/nodemon.json
        container_name: ts-node-docker
        expose:
        - '4000'
        ports:
        - '4000:4000'
        command: npm run dev
```
Isso cria um container chamado **ts-node-docker**, executa o *Dockerfile* criado e executa a build (veja o **target**).

Isso também cria *volumes* para o código fonte (src), e nodemon config, o que possibilita o hot-reloading (carregamento rápido). 
obs: *Hot-reloading* é usado para atualizar apenas o arquivo no qual o código é alterado. 

E finalmente, ele mapeia a porta (port) da máquina para o docker container (deve ser a mesma porta na configuração do *express*).

- Com isso é possível finalmente fazer build da imagem docker com:
```console
    docker-compose build
```

- E finalmente executar o container com o comando:
```console
    docker-compose up -d
```
**Obs:** *-d* Modo desacoplado: Execute containers no fundo, imprima novos nomes de containers.
Agora já se tem um container que observa quaisquer mudanças feitas no código fonte TypeScript. Recomenda-se bastante o uso do Docker Desktop app para ver containers em execução.

- É possível parar o container tambem com:
```console
    docker-compose down
```

- Por fim para quem não gosta de ficar digitando todos esses comandos docker. Crie um **Makefile** na raiz do projeto e insira o seguintes comandos para serem executados da linha de comando:

```Makefile
    up:
        docker-compose up -d
    down: 
        docker-compose down
```


# Referências
- link: https://dev.to/dariansampare/setting-up-docker-typescript-node-hot-reloading-code-changes-in-a-running-container-2b2f

- video tutorial: https://youtu.be/3o_DEJ9tB5A

- link: https://docs.npmjs.com/cli/v7/configuring-npm/package-json

- link: https://polyester-cormorant-8db.notion.site/Docker-00539f0e67174a848dbd17ca15a83f6e