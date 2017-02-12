---
title: La instancia de Vue
type: guia
order: 3
---

## Constructor

Cada vm Vue es iniciada creando una **instancia raíz de Vue** con la función constructora `Vue`:

``` js
var vm = new Vue({
  // opciones
})
```

A pesar de no estar estrictamente asociado con el [patrón MVVM](https://en.wikipedia.org/wiki/Model_View_ViewModel), el diseño de Vue estuvo parcialmente inspirado en él. Como convención, normalmente utilizamos la variable `vm` (abreviación de ViewModel) para referirnos a instancias de Vue.

Cuando crees una nueva instancia de Vue, necesitas pasar un **objeto de opciones** el cual puede contener opciones para datos, una plantilla, el elemento donde montarla, métodos, _callbacks_ para el ciclo de vida, etc. Puedes encontrar la lista completa de opciones en la [documentación de referencia de la API](../api).

El constructor de `Vue` puede ser extendido para crear **constructores de componentes** reutilizables con opciones predefinidas:

``` js
var MyComponent = Vue.extend({
  // opciones de extensión
})

// todas las instancias de `MyComponent` son creadas con
// las opciones de extensión predefinidas
var myComponentInstance = new MyComponent()
```

Aunque es posible crear instancias extendidas imperativamente, la mayor parte del tiempo es recomendable componerlas declarativamente in plantillas como elementos personalizados. Hablaremos de ello en detalle en [el sistema de componentes](components.html). Por ahora, solo necesitas saber que todos los componentes de Vue son, esencialmente, instancias de Vue extendidas.

## Propiedades y métodos

Cada instancia de Vue **proxies** todas las propiedades que se encuentran en su objeto `data`:

``` js
var data = { a: 1 }
var vm = new Vue({
  data: data
})

vm.a === data.a // -> verdadero

// modificar el valor de la propiedad también afecta a los datos originales
vm.a = 2
data.a // -> 2

// ... y viceversa
data.a = 3
vm.a // -> 3
```

Debe tenerse en cuenta 
It should be noted that only these proxied properties are **reactive**. If you attach a new property to the instance after it has been created, it will not trigger any view updates. We will discuss the reactivity system in detail later.

In addition to data properties, Vue instances expose a number of useful instance properties and methods. These properties and methods are prefixed with `$` to differentiate them from proxied data properties. For example:

``` js
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // -> true
vm.$el === document.getElementById('example') // -> true

// $watch is an instance method
vm.$watch('a', function (newVal, oldVal) {
  // this callback will be called when `vm.a` changes
})
```

<p class="tip">Don't use [arrow functions](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions) on an instance property or callback (e.g. `vm.$watch('a', newVal => this.myMethod())`). As arrow functions are bound to the parent context, `this` will not be the Vue instance as you'd expect and `this.myMethod` will be undefined.</p>

Consult the [API reference](../api) for the full list of instance properties and methods.

## Instance Lifecycle Hooks

Each Vue instance goes through a series of initialization steps when it is created - for example, it needs to set up data observation, compile the template, mount the instance to the DOM, and update the DOM when data changes. Along the way, it will also invoke some **lifecycle hooks**, which give us the opportunity to execute custom logic. For example, the [`created`](../api/#created) hook is called after the instance is created:

``` js
var vm = new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` points to the vm instance
    console.log('a is: ' + this.a)
  }
})
// -> "a is: 1"
```

There are also other hooks which will be called at different stages of the instance's lifecycle, for example [`mounted`](../api/#mounted), [`updated`](../api/#updated), and [`destroyed`](../api/#destroyed). All lifecycle hooks are called with their `this` context pointing to the Vue instance invoking it. You may have been wondering where the concept of "controllers" lives in the Vue world and the answer is: there are no controllers. Your custom logic for a component would be split among these lifecycle hooks.

## Lifecycle Diagram

Below is a diagram for the instance lifecycle. You don't need to fully understand everything going on right now, but this diagram will be helpful in the future.

![Lifecycle](/images/lifecycle.png)
