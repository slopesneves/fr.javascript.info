# Le Défilement


L'événement `scroll` permet de réagir sur le défilement d'une page ou d'un élément. Il y a pas mal de bonnes choses que nous pouvons faire ici.


Par exemple:
- Montrer/cacher des contrôles additionnelles ou information selon la ou se trouve l'utilisateur sur le document.
- Charger plus de données lorsque l'utilisateur défile vers le bas jusqu'à la fin de la page.

Voici une petite fonction pour montrer la position actuelle du défilement:

```js autorun
window.addEventListener('scroll', function() {
  document.getElementById('showScroll').innerHTML = pageYOffset + 'px';
});
```

```online
en action:

Current scroll = <b id="showScroll">Faites défiler la fenêtre</b>
```

L'évènement `scroll` fonctionne aussi bien avec `window` qu'avec les éléments defilables.

## Empêcher le défilement


Comment fait-on quelque chose de non-scrollable ?

Nous ne pouvons pas empêcher le défilement en utilisant `event.preventDefault()` dans l'écouteur `onscroll`, car il se déclenche *après* le défilement qui est déjà passé.

Mais nous pouvons empêcher le défilement avec `event.preventDefault()` sur un événement qui provoque le défilement, par exemple, l'événement `keydown` pour `key:pageUp` et `key:pageDown`.


Si nous ajoutons un gestionnaire d'évènement a ces évènements et à `event.preventDefault()` , alors le défilement ne pas se déclencher.


Il existe de nombreuses façons d’initialiser un défilement. Il est donc plus fiable d’utiliser la propriété CSS `overflow`.


Voici quelques taches que vous pouvez résoudre oubien regarder afin de voir une application de l'évènement `onscroll`.