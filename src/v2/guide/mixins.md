---
title: Mixins
type: guia
order: 17
---

## Lo básico

Los _mixins_ son una manera flexible de distribuir funcionalidades reusables para componentes de Vue. Un objeto _mixin_ puede contener cualquier opción de componente. Cuando un componente utiliza un _mixin_, todas las opciones en él serán fusionadas con las opciones propias del componente.

Oor ejemplo:

``` js
// Define un objeto _mixin_
var myMixin = {
  created: function () {
    this.hello()
  },
  methods: {
    hello: function () {
      console.log('hello from mixin!')
    }
  }
}

// Define un componente que use ese _mixin_
var Component = Vue.extend({
  mixins: [myMixin]
})

var component = new Component() // -> "hello from mixin!"
```

## Opciones de fusionado

Cuando un _mixin_ y el propio componente contienen opciones solapadas, serán "fusionadas" utilizando estrategias apropiadas. Por ejemplo, las funciones _hook_ serán añadidas a un arreglo para que todas ellas sean ejecutadas. Además, los _hooks_ del _mixin_ serán ejecutados antes que los propios del componente:

``` js
var mixin = {
  created: function () {
    console.log('mixin hook called')
  }
}

new Vue({
  mixins: [mixin],
  created: function () {
    console.log('component hook called')
  }
})

// -> "mixin hook called"
// -> "component hook called"
```

Las opciones que esperán valores de tipo objeto, por ejemplo `methods`, `components` y `directives`, serán fusionadas en un mismo objeto. Las opciones del componente tendrán mayor precedencia cuando haya llaves en conflicto en estos objetos:

``` js
var mixin = {
  methods: {
    foo: function () {
      console.log('foo')
    },
    conflicting: function () {
      console.log('from mixin')
    }
  }
}

var vm = new Vue({
  mixins: [mixin],
  methods: {
    bar: function () {
      console.log('bar')
    },
    conflicting: function () {
      console.log('from self')
    }
  }
})

vm.foo() // -> "foo"
vm.bar() // -> "bar"
vm.conflicting() // -> "from self"
```
 
Nota que las mismas estrategias de fusión son utilizadas en `Vue.extend()`.

## _Mixin_ globales

Puedes aplicar globalmente un _mixin_. ¡Ten cuidado! Una vez que aplicas globalmente un _mixin_, afectará a **cada** instancia de Vue que crees luego. Cuando se utiliza apropiadamente, puede ser utilizado para inyectar lógica de procesamiento para opciones personalizadas:

``` js
// inyecta una función controladora para la opción personalizada `myOption`
Vue.mixin({
  created: function () {
    var myOption = this.$options.myOption
    if (myOption) {
      console.log(myOption)
    }
  }
})

new Vue({
  myOption: 'hello!'
})
// -> "hello!"
```

<p class="tip">Utiliza _mixins_ globales con cuidado y no muy seguido, porque afecta a cada una de las instancias de Vue creadas, inclusive los componentes de terceros. La mayor parte del tiempo, deberías usarlos para el manejo de opciones personalizadas como la que mostramos en el ejemplo anterior. También es una buena idea empaquetarlos como [complementos](plugins.html) para evitar ejecutarlos más de una vez.</p>

## Estrategias de fusión de opciones personalizadas

Cuando fusionas opciones personalizadas, utilizan una estrategia por defecto, la cual simplemente sobreescribe el valor actual. Si deseas que una opción personalizada sea fusionada utilizando lógica propia, necesitas agregar una función a `Vue.config.optionMergeStrategies`:

``` js
Vue.config.optionMergeStrategies.myOption = function (toVal, fromVal) {
  // devuelve mergedVal
}
```

Para la mayoria de las opciones basadas en objetos, puedes utilizar la misma estrategia que en `methods`:

``` js
var strategies = Vue.config.optionMergeStrategies
strategies.myOption = strategies.methods
```

Puedes encontrar un ejemplo más avanzadoen la estrategia de fusionado de [Vuex 1.x](https://github.com/vuejs/vuex):

``` js
const merge = Vue.config.optionMergeStrategies.computed
Vue.config.optionMergeStrategies.vuex = function (toVal, fromVal) {
  if (!toVal) return fromVal
  if (!fromVal) return toVal
  return {
    getters: merge(toVal.getters, fromVal.getters),
    state: merge(toVal.state, fromVal.state),
    actions: merge(toVal.actions, fromVal.actions)
  }
}
```
