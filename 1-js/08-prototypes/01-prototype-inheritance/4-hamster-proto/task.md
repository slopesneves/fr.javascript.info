importance: 5

---

# Pourquoi deux hamsters sont pleins?

Nous avons deux hamsters: `speedy` et `lazy` héritant de l'objet général `hamster`.

Lorsque nous nourrissons l'un d'eux, l'autre est également plein. Pourquoi? Comment y remédier?

```js run
let hamster = {
  stomach: [],

  eat(food) {
    this.stomach.push(food);
  }
};

let speedy = {
  __proto__: hamster
};

let lazy = {
  __proto__: hamster
};

// Celui-ci a trouvé la nourriture
speedy.eat("apple");
alert( speedy.stomach ); // apple

// Celui-ci l'a aussi, pourquoi? réparer s'il vous plaît.
alert( lazy.stomach ); // apple
```

