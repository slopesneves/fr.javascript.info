
# Closure

JavaScript est un langage très fonctionnel. Cela nous donne beaucoup de liberté. Une fonction peut être créée dynamiquement, copiée dans une autre variable ou transmise en tant qu'argument à une autre fonction et appelée ultérieurement à partir d'un endroit totalement différent.

Nous savons qu'une fonction peut accéder aux variables en dehors de celle-ci, cette fonctionnalité est utilisée assez souvent.

Mais que se passe-t-il quand une variable externe change ? Une fonction obtient-elle la valeur la plus récente ou celle qui existait lors de la création de la fonction ?

De plus, que se passe-t-il lorsqu'une fonction se déplace à un autre endroit du code et est appelée à partir de là - a-t-elle accès aux variables externes du nouvel endroit ?

Les différents langages se comportent différemment ici, dans ce chapitre, nous traitons du comportement de JavaScript en particulier.

## Quelques questions

Examinons d’abord deux situations, puis étudions la mécanique interne pièce par pièce pour pouvoir répondre aux questions suivantes et à des questions plus complexes à venir.

1. La fonction `sayHi` utilise une variable externe `name`. Lorsque la fonction est exécutée, quelle valeur va-t-elle utiliser ?

    ```js
    let name = "John";

    function sayHi() {
      alert("Hi, " + name);
    }

    name = "Pete";

    *!*
    sayHi(); // qu'est-ce que ça va afficher : "John" ou "Pete" ?
    */!*
    ```

    De telles situations sont courantes dans les développements côté navigateur et côté serveur. Une fonction peut être planifiée pour s'exécuter plus tard que sa création, par exemple après une action de l'utilisateur ou une requête du réseau.

    Donc, la question est : est-ce que cela prend en compte les dernières modifications ?


2. La fonction `makeWorker` crée une autre fonction et la renvoie. Cette nouvelle fonction peut être appelée ailleurs. Aura-t-elle accès aux variables externes à partir de son lieu de création, ou du lieu d'invocation, ou des deux ?

    ```js
    function makeWorker() {
      let name = "Pete";

      return function() {
        alert(name);
      };
    }

    let name = "John";

    // create a function
    let work = makeWorker();

    // appel
    *!*
    work(); // qu'est-ce que ça va afficher ? "Pete" (nom où elle a été créé) ou "John" (nom où elle a été appelée) ?
    */!*
    ```


## Environnement Lexical

Pour comprendre ce qui se passe, voyons d’abord ce qu’est une "variable".

En JavaScript, chaque fonction en cours d'exécution, le bloc de code `{...}` et le script dans son ensemble ont un objet associé interne (masqué) appelé *Environnement Lexical*.

L'objet Environnement Lexical comprend deux parties :

1. *Environnement Record (Enregistrement d'Environnement)* -- un objet qui stocke toutes les variables locales en tant que propriétés (et d'autres informations telles que la valeur de `this`).
2. Une référence à *l'environnement lexical externe*, celui associé au code externe.

**Une "variable" est simplement une propriété de l'objet interne spécial `Environment Record`. "obtenir ou modifier une variable" signifie "obtenir ou modifier une propriété de cet objet".**

Par exemple, dans ce code simple, il n’existe qu’un seul Environnement Lexical :

![lexical environment](lexical-environment-global.svg)

Appelé aussi Environnement Lexical global, il est associé à l'ensemble du script.

Sur l'image ci-dessus, le rectangle correspond à l'enregistrement de l'environnement (magasin de variables) et la flèche à la référence externe. L'environnement lexical global n'a pas de référence externe, il pointe donc sur `null`.

Et c'est comme ça que ça change quand une variable est définie et assignée :

![lexical environment](lexical-environment-global-2.svg)

Les rectangles de droite nous montrent comment l'environnement lexical global change au cours de l'exécution :

1. Lorsque le script démarre, l'environnement lexical est vide.
2. La déclaration de `let phrase` apparaît. Aucune valeur n’a été affectée, donc `undefined` est stocké.
3. Une valeur est assignée à `phrase`.
4. `phrase` change de valeur.

Tout a l'air simple pour l'instant, non ?

Pour résumer :

- Une variable est une propriété d'un objet interne spécial, associée au bloc / fonction / script en cours d'exécution.
- Travailler avec des variables, c'est travailler avec les propriétés de cet objet.

### Fonction Declaration

Jusqu'à présent, nous n'avons observé que des variables. Maintenant, entrons dans les Fonctions Declaration.

**Contrairement aux variables `let`, elles sont entièrement initialisées non pas lorsque l'exécution les atteint, mais plus tôt, lorsqu'un environnement lexical est créé.**

Pour les fonctions de niveau supérieur, cela signifie le moment où le script est lancé.

That is why we can call a function declaration before it is defined.

The code below demonstrates that the Lexical Environment is non-empty from the beginning. It has `say`, because that's a Function Declaration. And later it gets `phrase`, declared with `let`:

![lexical environment](lexical-environment-global-3.svg)


### Inner and outer Lexical Environment

Now let's go on and explore what happens when a function accesses an outer variable.

During the call, `say()` uses the outer variable `phrase`, let's look at the details of what's going on.

When a function runs, a new Lexical Environment is created automatically to store local variables and parameters of the call.

For instance, for `say("John")`, it looks like this (the execution is at the line, labelled with an arrow):

<!--
    ```js
    let phrase = "Hello";

    function say(name) {
     alert( `${phrase}, ${name}` );
    }

    say("John"); // Hello, John
    ```-->

![lexical environment](lexical-environment-simple.svg)

So, during the function call we have two Lexical Environments: the inner one (for the function call) and the outer one (global):

- The inner Lexical Environment corresponds to the current execution of `say`.

    It has a single property: `name`, the function argument. We called `say("John")`, so the value of `name` is `"John"`.
- The outer Lexical Environment is the global Lexical Environment.

    It has `phrase` variable and the function itself.

The inner Lexical Environment has a reference to the `outer` one.

**When the code wants to access a variable -- the inner Lexical Environment is searched first, then the outer one, then the more outer one and so on until the global one.**

If a variable is not found anywhere, that's an error in strict mode (without `use strict`, an assignment to a non-existing variable, like `user = "John"` creates a new global variable `user`, that's for backwards compatibility).

Let's see how the search proceeds in our example:

- When the `alert` inside `say` wants to access `name`, it finds it immediately in the function Lexical Environment.
- When it wants to access `phrase`, then there is no `phrase` locally, so it follows the reference to the enclosing Lexical Environment and finds it there.

![lexical environment lookup](lexical-environment-simple-lookup.svg)

Now we can give the answer to the first question from the beginning of the chapter.

**A function gets outer variables as they are now, it uses the most recent values.**

Old variable values are not saved anywhere. When a function wants a variable, it takes the current value from its own Lexical Environment or the outer one.

So the answer to the first question is `Pete`:

```js run
let name = "John";

function sayHi() {
  alert("Hi, " + name);
}

name = "Pete"; // (*)

*!*
sayHi(); // Pete
*/!*
```


The execution flow of the code above:

1. The global Lexical Environment has `name: "John"`.
2. At the line `(*)` the global variable is changed, now it has `name: "Pete"`.
3. When the function `sayHi()`, is executed and takes `name` from outside. Here that's from the global Lexical Environment where it's already `"Pete"`.


```smart header="One call -- one Lexical Environment"
Please note that a new function Lexical Environment is created each time a function runs.

And if a function is called multiple times, then each invocation will have its own Lexical Environment, with local variables and parameters specific for that very run.
```

```smart header="Lexical Environment is a specification object"
"Lexical Environment" is a specification object: it only exists "theoretically" in the [language specification](https://tc39.es/ecma262/#sec-lexical-environments) to describe how things work. We can't get this object in our code and manipulate it directly. JavaScript engines also may optimize it, discard variables that are unused to save memory and perform other internal tricks, as long as the visible behavior remains as described.
```


## Nested functions

A function is called "nested" when it is created inside another function.

It is easily possible to do this with JavaScript.

We can use it to organize our code, like this:

```js
function sayHiBye(firstName, lastName) {

  // helper nested function to use below
  function getFullName() {
    return firstName + " " + lastName;
  }

  alert( "Hello, " + getFullName() );
  alert( "Bye, " + getFullName() );

}
```

Here the *nested* function `getFullName()` is made for convenience. It can access the outer variables and so can return the full name. Nested functions are quite common in JavaScript.

What's much more interesting, a nested function can be returned: either as a property of a new object (if the outer function creates an object with methods) or as a result by itself. It can then be used somewhere else. No matter where, it still has access to the same outer variables.

For instance, here the nested function is assigned to the new object by the [constructor function](info:constructor-new):

```js run
// constructor function returns a new object
function User(name) {

  // the object method is created as a nested function
  this.sayHi = function() {
    alert(name);
  };
}

let user = new User("John");
user.sayHi(); // the method "sayHi" code has access to the outer "name"
```

And here we just create and return a "counting" function:

```js run
function makeCounter() {
  let count = 0;

  return function() {
    return count++; // has access to the outer "count"
  };
}

let counter = makeCounter();

alert( counter() ); // 0
alert( counter() ); // 1
alert( counter() ); // 2
```

Let's go on with the `makeCounter` example. It creates the "counter" function that returns the next number on each invocation. Despite being simple, slightly modified variants of that code have practical uses, for instance, as a [pseudorandom number generator](https://en.wikipedia.org/wiki/Pseudorandom_number_generator), and more.

How does the counter work internally?

When the inner function runs, the variable in `count++` is searched from inside out. For the example above, the order will be:

![](lexical-search-order.svg)

1. The locals of the nested function...
2. The variables of the outer function...
3. And so on until it reaches global variables.

In this example `count` is found on  step `2`. When an outer variable is modified, it's changed where it's found. So `count++` finds the outer variable and increases it in the Lexical Environment where it belongs. Like if we had `let count = 1`.

Here are two questions to consider:

1. Can we somehow reset the counter `count` from the code that doesn't belong to `makeCounter`? E.g. after `alert` calls in the example above.
2. If we call `makeCounter()` multiple times -- it returns many `counter` functions. Are they independent or do they share the same `count`?

Try to answer them before you continue reading.

...

All done?

Okay, let's go over the answers.

1. There is no way: `count` is a local function variable, we can't access it from the outside.
2. For every call to `makeCounter()` a new function Lexical Environment is created, with its own `count`. So the resulting `counter` functions are independent.

Here's the demo:

```js run
function makeCounter() {
  let count = 0;
  return function() {
    return count++;
  };
}

let counter1 = makeCounter();
let counter2 = makeCounter();

alert( counter1() ); // 0
alert( counter1() ); // 1

alert( counter2() ); // 0 (independent)
```


Hopefully, the situation with outer variables is clear now. For most situations such understanding is enough. There are few details in the specification that we omitted for brevity. So in the next section we cover even more details.

## Environments in detail

Here's what's going on in the `makeCounter` example step-by-step, follow it to make sure that you know things in the very detail.

Please note the additional `[[Environment]]` property is covered here. We didn't mention it before for simplicity.

1. When the script has just started, there is only global Lexical Environment:

    ![](lexenv-nested-makecounter-1.svg)

    At that starting moment there is only `makeCounter` function, because it's a Function Declaration. It did not run yet.

    **All functions "on birth" receive a hidden property `[[Environment]]` with a reference to the Lexical Environment of their creation.**

    We didn't talk about it yet, that's how the function knows where it was made.

    Here, `makeCounter` is created in the global Lexical Environment, so `[[Environment]]` keeps a reference to it.

    In other words, a function is "imprinted" with a reference to the Lexical Environment where it was born. And `[[Environment]]` is the hidden function property that has that reference.

2. The code runs on, the new global variable `counter` is declared and gets the result of `makeCounter()` call. Here's a snapshot of the moment when the execution is on the first line inside `makeCounter()`:

    ![](lexenv-nested-makecounter-2.svg)

    At the moment of the call of `makeCounter()`, the Lexical Environment is created, to hold its variables and arguments.

    As all Lexical Environments, it stores two things:
    1. An Environment Record with local variables. In our case `count` is the only local variable (appearing when the line with `let count` is executed).
    2. The outer lexical reference, which is set to the value of `[[Environment]]` of the function. Here `[[Environment]]` of `makeCounter` references the global Lexical Environment.

    So, now we have two Lexical Environments: the first one is global, the second one is for the current `makeCounter` call, with the outer reference to global.

3. During the execution of `makeCounter()`, a tiny nested function is created.

    It doesn't matter whether the function is created using Function Declaration or Function Expression. All functions get the `[[Environment]]` property that references the Lexical Environment in which they were made. So our new tiny nested function gets it as well.

    For our new nested function the value of `[[Environment]]` is the current Lexical Environment of `makeCounter()` (where it was born):

    ![](lexenv-nested-makecounter-3.svg)

    Please note that on this step the inner function was created, but not yet called. The code inside `function() { return count++; }` is not running.

4. As the execution goes on, the call to `makeCounter()` finishes, and the result (the tiny nested function) is assigned to the global variable `counter`:

    ![](lexenv-nested-makecounter-4.svg)

    That function has only one line: `return count++`, that will be executed when we run it.

5. When `counter()` is called, a new Lexical Environment is created for the call. It's empty, as `counter` has no local variables by itself. But the `[[Environment]]` of `counter` is used as the `outer` reference for it, that provides access to the variables of the former `makeCounter()` call where it was created:

    ![](lexenv-nested-makecounter-5.svg)

    Now when the call looks for `count` variable, it first searches its own Lexical Environment (empty), then the Lexical Environment of the outer `makeCounter()` call, where finds it.

    Please note how memory management works here. Although `makeCounter()` call finished some time ago, its Lexical Environment was retained in memory, because there's a nested function with `[[Environment]]` referencing it.

    Generally, a Lexical Environment object lives as long as there is a function which may use it. And only when there are none remaining, it is cleared.

6. The call to `counter()` not only returns the value of `count`, but also increases it. Note that the modification is done "in place". The value of `count` is modified exactly in the environment where it was found.

    ![](lexenv-nested-makecounter-6.svg)

7. Next `counter()` invocations do the same.

The answer to the second question from the beginning of the chapter should now be obvious.

The `work()` function in the code below gets `name` from the place of its origin through the outer lexical environment reference:

![](lexenv-nested-work.svg)

So, the result is `"Pete"` here.

But if there were no `let name` in `makeWorker()`, then the search would go outside and take the global variable as we can see from the chain above. In that case it would be `"John"`.

```smart header="Closures"
There is a general programming term "closure", that developers generally should know.

A [closure](https://en.wikipedia.org/wiki/Closure_(computer_programming)) is a function that remembers its outer variables and can access them. In some languages, that's not possible, or a function should be written in a special way to make it happen. But as explained above, in JavaScript, all functions are naturally closures (there is only one exclusion, to be covered in <info:new-function>).

That is: they automatically remember where they were created using a hidden `[[Environment]]` property, and all of them can access outer variables.

When on an interview, a frontend developer gets a question about "what's a closure?", a valid answer would be a definition of the closure and an explanation that all functions in JavaScript are closures, and maybe few more words about technical details: the `[[Environment]]` property and how Lexical Environments work.
```

## Code blocks and loops, IIFE

The examples above concentrated on functions. But a Lexical Environment exists for any code block `{...}`.

A Lexical Environment is created when a code block runs and contains block-local variables. Here are a couple of examples.

### If

In the example below, the `user` variable exists only in the `if` block:

<!--
    ```js run
    let phrase = "Hello";

    if (true) {
        let user = "John";

        alert(`${phrase}, ${user}`); // Hello, John
    }

    alert(user); // Error, can't see such variable!
    ```-->

![](lexenv-if.svg)

When the execution gets into the `if` block, the new "if-only" Lexical Environment is created for it.

It has the reference to the outer one, so `phrase` can be found. But all variables and Function Expressions, declared inside `if`, reside in that Lexical Environment and can't be seen from the outside.

For instance, after `if` finishes, the `alert` below won't see the `user`, hence the error.

### For, while

For a loop, every iteration has a separate Lexical Environment. If a variable is declared in `for(let ...)`, then it's also in there:

```js run
for (let i = 0; i < 10; i++) {
  // Each loop has its own Lexical Environment
  // {i: value}
}

alert(i); // Error, no such variable
```

Please note: `let i` is visually outside of `{...}`. The `for` construct is special here: each iteration of the loop has its own Lexical Environment with the current `i` in it.

Again, similarly to `if`, after the loop `i` is not visible.

### Code blocks

We also can use a "bare" code block `{…}` to isolate variables into a "local scope".

For instance, in a web browser all scripts (except with `type="module"`) share the same global area. So if we create a global variable in one script, it becomes available to others. But that becomes a source of conflicts if two scripts use the same variable name and overwrite each other.

That may happen if the variable name is a widespread word, and script authors are unaware of each other.

If we'd like to avoid that, we can use a code block to isolate the whole script or a part of it:

```js run
{
  // do some job with local variables that should not be seen outside

  let message = "Hello";

  alert(message); // Hello
}

alert(message); // Error: message is not defined
```

The code outside of the block (or inside another script) doesn't see variables inside the block, because the block has its own Lexical Environment.

### IIFE

In the past, there were no block-level lexical environment in JavaScript.

So programmers had to invent something. And what they did is called "immediately-invoked function expressions" (abbreviated as IIFE).

That's not a thing we should use nowadays, but you can find them in old scripts, so it's better to understand them.

IIFE looks like this:

```js run
(function() {

  let message = "Hello";

  alert(message); // Hello

})();
```

Here a Function Expression is created and immediately called. So the code executes right away and has its own private variables.

The Function Expression is wrapped with parenthesis `(function {...})`, because when JavaScript meets `"function"` in the main code flow, it understands it as the start of a Function Declaration. But a Function Declaration must have a name, so this kind of code will give an error:

```js run
// Try to declare and immediately call a function
function() { // <-- Error: Unexpected token (

  let message = "Hello";

  alert(message); // Hello

}();
```

Even if we say: "okay, let's add a name", that won't work, as JavaScript does not allow Function Declarations to be called immediately:

```js run
// syntax error because of parentheses below
function go() {

}(); // <-- can't call Function Declaration immediately
```

So, parentheses around the function is a trick to show JavaScript that the function is created in the context of another expression, and hence it's a Function Expression: it needs no name and can be called immediately.

There exist other ways besides parentheses to tell JavaScript that we mean a Function Expression:

```js run
// Ways to create IIFE

(function() {
  alert("Parentheses around the function");
}*!*)*/!*();

(function() {
  alert("Parentheses around the whole thing");
}()*!*)*/!*;

*!*!*/!*function() {
  alert("Bitwise NOT operator starts the expression");
}();

*!*+*/!*function() {
  alert("Unary plus starts the expression");
}();
```

In all the above cases we declare a Function Expression and run it immediately. Let's note again: nowadays there's no reason to write such code.

## Garbage collection

Usually, a Lexical Environment is cleaned up and deleted after the function run. For instance:

```js
function f() {
  let value1 = 123;
  let value2 = 456;
}

f();
```

Here two values are technically the properties of the Lexical Environment. But after `f()` finishes that Lexical Environment becomes unreachable, so it's deleted from the memory.

...But if there's a nested function that is still reachable after the end of `f`, then it has `[[Environment]]` property that references the outer lexical environment, so it's also reachable and alive:

```js
function f() {
  let value = 123;

  function g() { alert(value); }

*!*
  return g;
*/!*
}

let g = f(); // g is reachable, and keeps the outer lexical environment in memory
```

Please note that if `f()` is called many times, and resulting functions are saved, then all corresponding Lexical Environment objects will also be retained in memory. All 3 of them in the code below:

```js
function f() {
  let value = Math.random();

  return function() { alert(value); };
}

// 3 functions in array, every one of them links to Lexical Environment
// from the corresponding f() run
let arr = [f(), f(), f()];
```

A Lexical Environment object dies when it becomes unreachable (just like any other object). In other words, it exists only while there's at least one nested function referencing it.

In the code below, after `g` becomes unreachable, enclosing Lexical Environment (and hence the `value`) is  cleaned from memory;

```js
function f() {
  let value = 123;

  function g() { alert(value); }

  return g;
}

let g = f(); // while g is alive
// their corresponding Lexical Environment lives

g = null; // ...and now the memory is cleaned up
```

### Real-life optimizations

As we've seen, in theory while a function is alive, all outer variables are also retained.

But in practice, JavaScript engines try to optimize that. They analyze variable usage and if it's obvious from the code that an outer variable is not used -- it is removed.

**An important side effect in V8 (Chrome, Opera) is that such variable will become unavailable in debugging.**

Try running the example below in Chrome with the Developer Tools open.

When it pauses, in the console type `alert(value)`.

```js run
function f() {
  let value = Math.random();

  function g() {
    debugger; // in console: type alert(value); No such variable!
  }

  return g;
}

let g = f();
g();
```

As you could see -- there is no such variable! In theory, it should be accessible, but the engine optimized it out.

That may lead to funny (if not such time-consuming) debugging issues. One of them -- we can see a same-named outer variable instead of the expected one:

```js run global
let value = "Surprise!";

function f() {
  let value = "the closest value";

  function g() {
    debugger; // in console: type alert(value); Surprise!
  }

  return g;
}

let g = f();
g();
```

```warn header="See ya!"
This feature of V8 is good to know. If you are debugging with Chrome/Opera, sooner or later you will meet it.

That is not a bug in the debugger, but rather a special feature of V8. Perhaps it will be changed sometime.
You always can check for it by running the examples on this page.
```
