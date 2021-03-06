# Functional Programming

Programação funcional se baseia em alguns conceitos principais:

- Função pura
- Sem estado compartilhado
- Dados imutaveis
- Sem efeitos colaterais
- Declarativa

## High Order Functions

Toda função de ordem superior recebe ou retorna uma função. Inverso de funções de ordem normal.

```js
function reduce(reducer, initial, arr) {
  let acc = initial
  for (let i = 0; i < arr.length; i++) {
    acc = reducer(acc, arr[i])
  }
  return acc
}
```

As funções do JavaScript são de primeira classe, ou seja, ela pode ser usada como qualquer outro valor. Pode ser atribuida a variaveis, podem receber funções, retonar funções, enfim... literalmente pode ser usada com flexibilidade extremamente alta.

Funções de ordem superior nos permite abstrair calculos especificos ou regras de negocio para que sejam mais genericas e possam ser usadas em varios lugares.

Exemplo de função que retorna de um array os valores que tem tamanho maior que 4:

```js
function censor(words) {
  const filtered = []
  for (let i = 0 i < words.length; i++) {
    const word = words[i]
    if (word.length !== 4) filtered.push(word)
  }
  return filtered
}
```

Ok, é uma função normal, mas e se quisermos que ela trouxesse apenas o valores que começassem com 's'? No caso teriamos que criar outra função com essa nova condição.. Mas com funções de ordem superior, podemos abstrair essas condições para funções recebidas nos parametros, assim a função tem mais flexibilidade e menos regras impostas nela.

```js
function reduce(reducer, initial, arr) {
  let acc = initial
  for (let i = 0; i < arr.length; i++) {
    acc = reducer(acc, arr[i])
  }
  return acc
}

function filter(words, callback) {
  return reduce(
    (acc, curr) => (callback(curr) ? acc.concat([curr]) : acc),
    [],
    words
  )
}

const censor = (words) => filter(words, (word) => word.length !== 4)
```

Nesse exemplo, podemos perceber um bom nivel de abstração. A função censor está totalmente livre de suas responsabilidades e apenas filtra o que deseja com a função filter que pode ser usada em qualquer valor. A função de ordem superior <code>reduce</code> é uma abstração da responsabilidade de iterar sobre o array e selecionar os desejados, mas essa responsabilidade de saber quem deve ser selecionado fica por conta da função que recebe por argumentos <code>reducer</code>.

# Parcial Application

Uma aplicação parcial é uma função que foi aplicada a alguns, mas ainda não a todos os seus argumentos. Em outras palavras, é uma função que possui alguns argumentos fixos dentro de seu escopo de encerramento. Uma função com alguns de seus parâmetros fixos é considerada parcialmente aplicada .

Utilizando o exemplo de cima da para notar que quando você aplica <code>add(5)</code> já é uma aplicação parcial pois foi passado apenas um argumento. No caso, não confundem que aplicação parcial só pode ser usada em modelo curry.

Aplicativos parciais podem levar tantos ou poucos argumentos por vez, conforme desejado. As funções curried, por outro lado, sempre retornam uma função unária: uma função que recebe um argumento.

<em>Todas as funções curried retornam aplicativos parciais, mas nem todos os aplicativos parciais são o resultado de funções curried.</em>

# Point-free style

Esse conceito se basea na declaração de funções que não fazem referência aos seus parametros, ou seja, não tem "noção" de seus parametros.

```js
// declaração comum
function anything(/* parametros aqui*/) {}
const anything2 = (/* parametros aqui */) => {}
const anything3 = function (/* parametros aqui */) {}
```

Esse conceito é bastante usado junto a funções curried. Pegando nosso primeiro exemplo:

```js
const add = (a) => (b) => a + b
// Inc10 será uma função que sempre adicionará mais 10 aos seus argumentos.
// Nesse caso inc10 é uma função que foi declarada sem referência aos seus parametros.
const inc10 = add(10)
```

Com esse conceito, podemos criar funções especializadas, como a <code>inc10()</code> que se tornou uma função especializada da <code>add()</code>, assim podemos fazer quantas funções especializadas quisermos.

```js
const inc20 = add(20)
const inc30 = add(30)
//...
```

<em>Todas as funções curried são uma forma de função de ordem superior que permite criar versões especializadas da função original para o caso de uso específico em questão.</em>

# Composition function and Curry

Funções curried são funções que recebem um argumento por vez. Por exemplo:

```js
const add = (a) => (b) => a + b

console.log(add(5)(2)) // 7
```

Para fazer chamar essas funções você pode utilizar a syntax de <strong>application functions</strong>. Em JavaScript, os parênteses <code>()</code> após a referência de função acionam a invocação da função.

Esse estilo de declaração de função é extremamente atrelada com o conceito de <strong>clousures</strong> (Abordarei esse conceito em outro topico).

<hr>
<strong>Curry</strong> é otimo para composição de função. Além disso, fora do contexto de composição, <strong>curry</strong> é perfeito para <strong>point-free style</strong>, ou seja, especialização. Contudo, o real poder das <strong>function curry</strong> é na composição de função, pela simplificação.

Uma função pode receber qualquer número de entradas, mas só pode retornar uma única saída. Para que as funções sejam compostas, o tipo de saída deve estar alinhado com o tipo de entrada esperado:

```
f: a => b
g: b => c
h: a => c
```

Se a função <code>g</code> acima esperava dois parâmetros, a saída de <code>f</code> não se alinharia com a entrada para <code>g</code>:

```
f: a => b
g: (x, b) => c
h: a => c
```

Lembre-se de que a definição de uma função curried é uma função que usa vários parâmetros, um de cada vez, pegando o primeiro argumento e retornando uma série de funções, cada uma das quais leva o próximo argumento até que todos os parâmetros tenham sido coletados. Então para usar funções dentro de funções curry, sempre faça funções curry para utilizar o poder total desse estilo.

<code>
g: x => b => c
</code>

As palavras-chave dessa definição são “uma de cada vez”. A razão pela qual funções curried são tão convenientes para composição de função é que elas transformam funções que esperam vários parâmetros em funções que podem receber um único argumento, permitindo que elas se encaixem em um pipeline de composição de função. Tome a função <code>trace</code> como um exemplo[0]:

```js
const pipe =
  (...fns) =>
  (x) =>
    fns.reduce((y, f) => f(y), x)

const trace = (label) => (value) => {
  console.log(`${label}: ${value}`)
  return value
}

const g = (n) => n + 1
const f = (n) => n * 2

const h = pipe(g, trace("after g"), f, trace("after f"))

h(20)

/*
after g: 21
after f: 42
*/
```

A função <code>trace</code> define dois parâmetros, mas os leva um de cada vez, o que nos permite especializar a função embutida. Se <code>trace</code> não tivesse curry, não poderíamos usá-lo dessa forma. Teríamos que escrever o pipeline assim:

```js
const pipe =
  (...fns) =>
  (x) =>
    fns.reduce((y, f) => f(y), x)

const trace = (label, value) => {
  console.log(`${label}: ${value}`)
  return value
}

const g = (n) => n + 1
const f = (n) => n * 2

const h = pipe(
  g,
  // As chamadas trace () não são mais livres de pontos,
  // introduzindo a variável intermediária, `x`.
  (x) => trace("after g", x),
  f,
  (x) => trace("after f", x)
)

h(20)

/*
after g: 21
after f: 42
*/
```

# Data last

Curry não é a única coisa que você se deve se atentar, você também precisa se certificar de que a função está esperando parâmetros na ordem correta para especializá-los, se a ordem for inversa, ficaria assim:

```js
const pipe =
  (...fns) =>
  (x) =>
    fns.reduce((y, f) => f(y), x)

const trace = (value) => (label) => {
  console.log(`${label}: ${value}`)
  return value
}

const g = (n) => n + 1
const f = (n) => n * 2

const h = pipe(
  g,
  // As chamadas trace () não são mais livres de pontos,
  // por que a ordem dos argumentos está inversa e não dá para criar a especialização.
  (x) => trace(x)("after g"),
  f,
  (x) => trace(x)("after f")
)

h(20)

/*
after g: 21
after f: 42
*/
```

Uma boa abordagem para sempre criar funções que recebam os argumentos de forma ideal para a especialização, é pensar em **"dados por ultimo"** (data last), o que significa que você deve pegar os parâmetros de especialização primeiro e pegar os dados nos quais a função atuará por último.

# Resumo

<strong>Functions curried: </strong> Uma função que recebe vários argumentos um por vez e retornando funções para os proximos argumentos, ótimo para criar funções especializadas. <br />
<strong>Partial Application: </strong> É uma função que é aplicada a alguns, mas não a todos os argumentos. Os argumentos aos quais a função já foi aplicada são chamados de parâmetros fixos. <br />
<strong>Point-free style: </strong> É uma forma de definir uma função sem referência a seus argumentos. Geralmente, uma função sem ponto é criada chamando uma função que retorna uma função, como uma função curried. <br />
<strong>Data last functions: </strong> É uma forma de idealizar a criaçao de funções curried, para que sempre ocorra a especialização de forma correta.

## Extra

Funções curried são boas para composições, por que convertem facilmente funções n-ária em funções unárias, necessárias para pipelines de composição de funções.

[Aqui](https://github.com/JohnAnon9771/Concepts-fundamentals/blob/main/functional-programming/pratices/readme.md) está alguns exercicios usando uma linguagem **totalmente funcional (Haskell)** e uma de **multiparadigma (Javascript)**.
