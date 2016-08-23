---
title: Modularize seu front-end com Browserify
cover_index: /assets/blocos_home.jpg
cover_detail: /assets/blocos.jpg

---

Ultimamente tenho optado cada vez menos por utilizar SPA em pequenas aplicações ou mesmo em aplicações com *Server Rendering*, simplesmente pelo fato de que de se não for para prototipação rápida ou prova de conceito (ou mesmo nesses casos), sua arquitetura front-end deve ser emergente para evitar otimização prematura, como qualquer arquitetura, certo?

Se há uma parte da arquitetura que não pode ser deixada pra depois é a modularização, sem ela o código é repetido, confuso, macarrônico e super difícil de testar.

Ai é que o problema começa. Quais das um milhão de formas de modularizar eu uso? Module Pattern? Módulos AMD? CommonJS? Namespace Pattern? Classes?

Como é de se esperar código *vanilla* é mais fácil de modularizar, já que as novas features de modularização do ES6/7 estão já há algum tempo disponíveis através de transpilers como o BabelJS. Então isso resolve meu problema com poucas linhas de configuração e um *package.json*. Um exemplo pode ser visto abaixo (da própria doc do BabelJS):

```
// lib/math.js
export function sum(x, y) {
  return x + y;
}

// app.js
import * as math from "lib/math";
console.log("2π = " + math.sum(math.pi, math.pi));

```

Mas e os módulos que não são *AMD* ou *CommonJS*, como eu utilizo?

Foi ai que o Browserify me ganhou, com ele, além de ganhar módulos eu ganho bundle, transpiling e a infinidade de pacotes que existem para o Node.JS no NPM. E isso no front-end. Além disso tenho o *shim* um velho conhecido desde o *RequireJS*, que nos permite usar qualquer módulo mesmo não sendo *AMD* ou *CommonJS*.

Este é exatamente o objetivo de criação do Browserify, então vamos por a mão na massa.


**Instalação**

Bom, ele é um plugin NPM entao:

```
npm init && npm i --save-dev browserify
```

Você vai ver que no site oficial é instruída a instalação global, prefira usar módulos locais assim você garante integração a qualquer builder e/ou desenvolvedor :D.

```
npm i --save bluebird
```
Junto com o browserify, instalamos junto o pacote *bluebird* do node.js (usado para gestão de promisses super performáticas) para ver sua mágica como dependência da aplicação.

Agora vamos configurar

**Configuração**

Adicione o seguinte comando a seção de scripts do seu packages.json para rodar de maneira local:

```
scripts:{
    "build": "browserify index.js > main.js"
}
```

O browserify precisa de um arquivo raiz pra começar a ler a árvore de dependências, como uma árvore de dependências:

![Árvore de Dependências](/assets/dependency-tree.png)


O que fizemos foi configurar index.js como essa raíz e main.js como o arquivo final de bundle.

Crie o arquivo index.js:

```
//index.js

var doSomething = require('./dosomething')

console.log('doSomething', doSomething);
```

Ele está requisitando um módulo chamado *dosomething*, como no node.js.

Crie também um arquivo "doSomething.js":

```
//doSomething.js

var Promise = require('bluebird')

module.exports = function(a, b){
	var p = new Promise()
	if(a > b)
		p.reject()
	p.resolve()
}
```
Este módulo exporta uma função com dois parâmetros e faz o uso do módulo nativo do node.js para realizar uma promessa.

**Uso**

Pronto! Agora se nós rodarmos o comando:

```
npm run-script build
```

O bundle é realizado e temos um arquivo main.js com o código compilado e minificado.
Se adicionarmos à uma página HTML simples verá o console aparecendo :D 

o código:
```
<!DOCTYPE html>
<html>
<head>
	<title>Browserify me!</title>
	<script type="text/javascript" src="./main.js"></script>
</head>
<body>

</body>
</html>
```

e o console:

![Console log](/assets/console-log-browserify.png)


Mas e o meu pipeline de build? Usa grunt, gulp e não estou afim de mudar?

Então pega esses plugins lindões [gulp-browserify](https://www.npmjs.com/package/gulp-browserify) e [grunt-browserify](https://www.npmjs.com/package/grunt-browserify)


Ainda é possível transpilar *Typescript*, *ES6/7*, *Coffescript*(S2) e até handlebars no meio do bundle, mas isso é assunto pros próximos posts.

Só pra dar um gostinho, veja o mesmo código com ES6:


```
//doSomething.js > es6

import Promise from 'bluebird'

export default (a, b) => {
	let p = new Promise()
	if(a > b)
		p.reject()
	p.resolve()
}
```

ou 

```
//doSomething.js > es6 v2

import Promise from 'bluebird'


export function sum(a, b) => {
	return a + b
}

export default (a, b) => {
	let p = new Promise()
	if(a > b)
		p.reject()
	p.resolve()
}

//index.js

import doSomething from './doSomething'
import {sum} from './doSomething' // --> importando a função de forma explícita S2
```

**Conclusão**

Instalar e configurar o browserify é simples e não traz quase nada de *boilerplate*. Além disso ele te induz a modularizar melhor a sua *code base*, facilitando até o refactoring enquanto testa. É simples, testável, e pra quem gosta de *vanilla* atende sem ~~super-mega-features-de-bind-lentas-e-factories-que-nem-sempre-voce-precisa~~ funcionalidades a mais por exemplo, o controle da orientação a objetos ainda está na tua mão.

Eu tenho gostado muito de utilizá-lo e para um cenário onde já tenho build-tool configurada, se encaixa perfeitamente, sendo, na minha opinião, melhor do que utilizar o Webpack que acaba atuando como build-tool também. Mas isso é assunto para os próximos posts.

Falarei sobre testes, watch e integrações com Browserify nos próximos posts. Espero que tenham gostado. Quero saber o que acham, se têm utilizado e quais suas glórias e dores com seu uso, deixe seu comentário!

Abraços e até mais!


Link da Demo: https://github.com/fabiodamasceno/browserify-love-demo
BabelJs: https://babeljs.io/
Browserify: http://browserify.org/

