
---
layout: post
title:  return de Funções
categories: [JavaScript]
excerpt: Ele faz uma function se tornar uma variável: isso faz um valor interno se tronar público. Ele serve para PARAR o processamento de uma function. Ele faz as duas coisas ao mesmo tempo.
---

Antes de entrarmos em detalhes, vou ilustrar o assunto para ficar fácil de entender. Saia um pouco do campo da tecnologia e volte para o mundo real. Aliás, as coisas virtuais foram copiadas do mundo real. 
Pois bem, imagine que você peça a uma pessoa para ir ao açougue comprar carne. *A pessoa vai e volta com a carne e entrega nas suas mãos[1]*. Talvez você saiba como esta tarefa foi realizada, mas não sabe de todos os detalhes. A única coisa de que sabe é que a pessoa fez o que você pediu: ela te trouxe a carne. Poderíamos dar mais um outro exemplo que abrangeria todos os pontos que queremos destacar neste post. Dentro deste mesmo contexto, imagine que você peça a uma outra pessoa para levar a carne para a casa de um amigo e ela faz isto: *ela leva a carne para um amigo[2]*.
Note que foi ilustrado duas situação:
-1 eu recebo um retorno, a carne: ele vai e volta com o resultado
-2 eu não recebo nada de volta, vácuo. Ele vai e **não** volta: no JavaScript isto aparece como `undefined`

Vamos entender como isso funciona no código:
```js
function pessoVaiComprarCarne() {
    return "Voitei com a carne";
}
console.log(pessoVaiComprarCarne()); //Voitei com a carne

function leveCaneParaAmigo () {
	// Alguns códigos/tarefas
    return;
}
console.log(leveCaneParaAmigo()); //undefined
```
Legal, né?
Com isso em mente, poderíamos notar o seguinte, sobre o `return`:
1- Ele faz uma `function` se tornar uma variável: isso faz um valor interno se tronar público
2- Ele serve para PARAR o processamento de uma `function`
3- Ele faz as duas coisas ao mesmo tempo

## Tornando algo público
Você já deve saber que uma coisa que está dentro de uma `function` não pode ser acessada de fora dela. Sendo assim, sempre que você precisar tornar algo público, uma das maneiras é usar o `return`, transformando tal função numa variável.
```js
function pessoa() {
	return {
		nome: "Maria",
		sobrenome: "Sousa"
	};
}
console.log(pessoa()); // {nome: "Maria", sobrenome: "Sousa"}
console.log(pessoa()/* Eu tenho o valor do return bem aqui e posso pegá-lo com o "ponto". Veja: */.nome); // Maria
```
## Parando um processo em andamento
Vamos inventar um problema para entender como isso acontece. Digamos que eu queira pegar o `array` `[15, 5, 75, 10, 9, 55, 90, 8]` e descobrir em qual índice obtenho a soma de 100.
```js
let idx;
(function(x){
	let soma = 0;
	for (let i = 0, max = x.length; i < max; i = i + 1) {
		soma = soma + x[i];
		if (soma >= 100) {
			idx = i;
			return; // undefined
		}
	}
	idx = -1;
	return "A soma não atingiu o valor de 100.";
}([15, 5, 75, 10, 9, 55, 90, 8])); // undefined
console.log(idx); // 3
```
Note que eu percorri o `array` somente até encontrar o que estava pesquisando. Ao encontrar o valor **dou ordem para PARAR tudo** e acabar `if (soma >= 100)`. Note que o `return` da função é `undefined`. Eu vou repetir:  a função acaba mesmo, de verdade. Ela literalmente abandona tudo! No exemplo acima termina `idx = 3`.


## Parando e informando um valor
Agora vamos usar a mesma função com outros valores no `array`: `[95, 9, 1, 0, 9, 50, 0, 5]`.
```js
let idx = (function(x){
	let soma = 0;
	for (let i = 0, max = x.length; i < max; i = i + 1) {
		soma = soma + x[i];
		if (soma >= 100) {
			return i;
		}
	}
	return "A soma não atingiu o valor de 100.";
}([95, 9, 1, 0, 9, 50, 0, 5]));
console.log(idx); // 1
```
Observe que no exemplo acima paramos o `loop` e ao mesmo tempo atribuímos o valor a `idx`.




<!--stackedit_data:
eyJoaXN0b3J5IjpbLTY4NzAwNzcyMl19
-->