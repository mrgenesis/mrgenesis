---
layout: post
title:  Diferenças entre var, let e const
categories: [JavaScript]
excerpt: A declaração de variáveis em JavaScript pode ser feita usando três palavras chaves: var, let e const. Com eles, é possível escrever códigos muito mais consistentes e legíveis. Continue lendo! Acredito que avançaremos em conhecimentos valiosos sobre este assunto.
---

A declaração de variáveis em JavaScript pode ser feita usando três palavras chaves: `var`, `let` e `const`.
Mas, o que é uma variável e o que significam essas três palavras? De forma resumida uma variável é um **nome que representa um valor**. Exemplo:
```js
var nome = "valor da variável";
console.log(nome); // Output: valor da variável
```
No exemplo acima vimos como declarar uma variável e atribuir um valor a ela. Neste caso, estamos dizendo que a palavra `nome` representa o valor  "valor da variável". Fácil, né?
A princípio, a maneira de declarar variáveis é essa, e você poderia fazer assim em todas as variáveis do seu código. Mas isso seria um grande equívoco, porque `let` e `const` proporcionariam recursos excelentes. Com eles, é possível escrever códigos muito mais consistentes e legíveis. Continue lendo! Acredito que avançaremos em conhecimentos valiosos sobre este assunto.

#### Fazer resumo do que será falado (O que vamos ver)



## Escopo - Contexto léxico
Não conseguimos falar de variáveis sem falar em escopo ou contexto léxico. Então você precisa entender bem esta questão para assim conseguir diferenciar `var`, `let` e `const`. Para ilustrar, **escopo é como se fosse um território**. *Você não vai conseguir votar nas eleições canadenses usando um Título de Eleitor brasileiro*. Da mesma forma se tentar acessar o valor de uma variável que está fora do escopo (em outro território) não vai dar certo.
Você vai notar que **os escopos são definidos por chaves** `{}`; e quando não há chaves é porque a variável está no escopo global.
Com esse principio em mente, lembre de que sempre que uma variável é criada é **definido um escopo (*território*) para ela**.
```js
(function brasil(){ // escopo da função {brasil}
	var paisBrasil = "Brasil";
}());
(function canada(){ // esscopo da função {canadá}
	var paisCanada = "Canadá";
	console.log(paisBrasil); //paisBrasil is not defined
}());
```
Se você notar, o exemplo acima mostra que os valores não podem ser acessados de fora: isso gera um erro. Mas o inverso é aceitável. Veja:
```js
var mundo = "Variável Global";
(function brasil(){ // escopo da função {brasil}
	// aqui dentro é possível acessar o escopo Global
	console.log(mundo); // Output: Variável Global
}());
```
Então, grave isto: _quem está dentro consegue ver quem está fora, mas quem está fora não consegue ver quem está dentro_.

---------

Importante!  Se você quiser **tornar visível** algo que está dentro de um escopo, isso é possível; mas um escopo é privado por padrão. De qualquer forma isso é outra história! Não falaremos disso aqui, okay?

-------

## var
Seu comportamento pode ser imprevisível. Por exemplo, é bem ruim quando temos uma variável no contexto global, e se você precisar fazer um loop e usar o `var` você terá este problema.
```js
for (var i = 0; i < 5; i++) {
	// codes
}
console.log(i); // Output: 4

if(true) {
	var varialvel = "Dentro do if";
}
console.log(varialvel); // Output: Dentro do if
```
Note no exemplo acima que após executar o `for` e `if`, teremos `i` e `variavel` disponíveis no escopo global. O `var` somente se mantem dentro do escopo de funções.

---

Não é legal criar variáveis no escopo global, porque você pode sobrescrever alguma outra variável e fazer seu código quebrar. Mas isso é outra história! Não falaremos disso aqui.
- - - 

## let

A expressão `let` surgiu de um conceito matemático, na linguagem [Logic of Computer Functions]([https://en.wikipedia.org/wiki/Let_expression#History](https://en.wikipedia.org/wiki/Let_expression#History) "Logic of Computer Functions"): uma etapa da evolução de cálculos lambda, [Dana Scott - nascido em 1932]([https://en.wikipedia.org/wiki/Dana_Scott](https://en.wikipedia.org/wiki/Dana_Scott) "Dana Scott - nascido em 1932"). A ideia era definir um escopo limitado àquela função para que a variável se tornasse apenas local.

Declarando uma variável com `let` dentro de `if`, o valor fica somente em `if`. Veja:
```js
if(true) {
	let privado = "Dentro do if";
	console.log(privado); // Output: Dentro do if
}
console.log(privado); // Output: privado is not defined
```
Mas quando é declarado com `var` conseguimos acessar o valor normalmente.
Lembre-se: **escopos são definidos por chaves `{}`**.


Você deve ter se apercebido até aqui de que a diferença entre `var` e `let` é que respectivamente um **ignora o escopo**   e o outro **só funciona dentro do escopo `{}`** . E se você chegou a conclusão que é muito melhor usar `let`, digo que concluiu corretamente: com `let` você terá muito mais controle do seu código, e o deixa muito mais claro.

#### let vs var
De forma bem resumida, funciona assim: 
1 - tudo dentro de chaves `{}` declarado com `let` só vai funcionar lá dentro
2 - Com `var` é diferente:
	- em `function(){}` funciona de forma privada igual `let`
	- em `if, else, for, while, switch, try`, vai vazar para o escopo de fora
```js
(function(){
	// escopo de fora do if
	if (true) {
		var varEmIf = "Var em if";
		let letEmIf = "Let em if";
	}
	console.log(varEmIf); // Output: Var em if
	console.log(letEmIf); // Output: varEmIf is not defined
}());
// escopo Global
console.log(varEmIf); // Output: varEmIf is not defined
```
# const
Funciona quase igual `let` com a diferença que o valor em `const` não pode ser mudado. Veja:
```js
const nome = "João";
nome = "Maria";
// Uncaught TypeError: Assignment to constant variable.

if(true) {
	const teste = "Dentro de if";
}
console.log(teste); // Uncaught ReferenceError: teste is not defined
```
Então se você precisa mudar o valor de uma determinada variável use `let`, porque `const` não permitirá alterações. Tem uma exceção: se você **trocar apenas a propriedade** de  um objeto.
```js
const obj = {
	nome: "Maria",
	sobrenome: "Sousa"
};
obj.sobrenome = "Sousa Silva";
console.log(obj.sobrenome); // Output: Sousa Silva
```
## Chaves
Quando estiver escrevendo código, há situações em que você poderá escolher entre usar ou não chaves `{}`. Figuras respeitadas no mundo da programação desencorajam o uso abreviado do `if` (single-statement context). O livro *Padrões JavaScript*, página 42, é um exemplo disso. lá é feita uma forte recomendação que se use as chaves, mesmo se o código em `if` tiver apenas uma linha. Não use single-statement context!

> ... **mas** você deve usá-las de qualquer forma - "*JavaScript
> Patterns*, por Stoyan Stefanov (O'Reylly). Todos os direitos
> reservados 2010 Yahoo!, 9780596806750".

De qualquer forma, `let` e `const` não te darão liberdade de escolha, [você será obrigado a usar chaves](https://github.com/Microsoft/TypeScript/issues/20435#issuecomment-348988471).
```js
//////////////// ERRADO!!!
if (true) 
	var nome = "Maria"; // Vai funcionar, mas não é aconselhável

if (true)
	let nome = "Maria";
// Uncaught SyntaxError: Lexical declaration cannot appear in a single-statement context

/////////////// CERTO!!!
if (tre) {
	var nome = "Maria";
}
```
Notou? Com `var` funciona, mas é desincentivado o uso. Com `let` e `const` não vai funcionar. Então se você ver o erro "`Uncaught SyntaxError: Lexical declaration cannot appear in a single-statement context`" é porque está faltando as chaves `{}`.

## Declaração das variáveis
Um detalhe muito importante é a questão sobre como declarar as variáveis. Não espalhe declarações ao longo do código. Faça tudo d uma vez: concentre as declarações todas no início, mesmo que você não tenha um valor a ser atribuído a ela. Este princípio dá legibilidade para o seu código. 
Página 30 do livro "*JavaScript Patterns*, por Stoyan Stefanov (O'Reylly). Todos os direitos reservados 2010 Yahoo!, 9780596806750".
```js
let myArray = [],
	count = 0,
	max = document.querySelectorAll("#id");

// Codes	
```


Note que quando não tenho uma valor a ser atribuído, eu indico o que será minha variável, por meio de atribuição de valores vazios: `[]` e `0`.
Esta prática resolve ainda outro problema de legibilidade que [as declarações de variáveis são processadas antes de qualquer outro código ser executado]([https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Statements/var#Description](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Statements/var#Description)), e isso causa uns comportamentos meio estranhos. Veja:
```js
console.log(a); // Output: undefined
let a = 4;
```
Antes de executar o código, o compilador avalia o código e identifica a declaração da variável; então atribui o valor `undefaned`. Na hora da execução, foi realizado o acesso a ela antes da atribuição do valor.
O **comportamento natural** seria o mostrado abaixo:
```js
console.log(a); // ReferenceError: a is not defined
```
Isso nos leva a outra questão importante: declarar é diferente de atribuir. `var` e `let` servem para declarar uma variável e sinal igual `=` para atribuir valores.

---

`const` força você declarar e atribuir ao mesmo tempo

---
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTkxMjQ5NDk3XX0=
-->