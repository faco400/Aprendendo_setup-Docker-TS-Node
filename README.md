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

# Referências
- link: https://dev.to/dariansampare/setting-up-docker-typescript-node-hot-reloading-code-changes-in-a-running-container-2b2f

- video tutorial: https://youtu.be/3o_DEJ9tB5A