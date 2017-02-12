---
title: Complementos
type: guia
order: 18
---

## Desarrollando un complemento

Los complementos normalmente añaden funcionalidad a nivel global a Vue. No hay un ámbito estrictamente definido para un complemento - hay diferentes tipos típicos de complemento que puedes desarrollar:

1. Añadir algunos métodos o propiedades globales, por ejemplo [vue-element](https://github.com/vuejs/vue-element)

2. Añadir uno o más recursos globales: directivas/filtros/transiciones, por ejemplo [vue-touch](https://github.com/vuejs/vue-touch)

3. Añadir opciones de componentes a través de _mixin_ globales, por ejemplo [vuex](https://github.com/vuejs/vuex)

4. Añadir métodos de instancia de Vue agregándolos a Vue.prototype.

5. Una biblioteca que provea una API propiar, a la vez que inyecta una combinación de las anteriores, por ejemplo [vue-router](https://github.com/vuejs/vue-router)

Un complemento de Vue.js debe exponer un método `install`. Este será ejecutado con el constructor de `Vue` como primer parámetro, junto con las opciones posibles:

``` js
MyPlugin.install = function (Vue, options) {
  // 1. agrega un método o propiedad global
  Vue.myGlobalMethod = function () {
    // algo de lógica
  }

  // 2. agrega un recurso global
  Vue.directive('my-directive', {
    bind (el, binding, vnode, oldVnode) {
      // algo de lógica
    }
    ...
  })

  // 3. inyecta algunas opciones de componente
  Vue.mixin({
    created: function () {
      // algo de lógica
    }
    ...
  })

  // 4. agrega un método de instancia
  Vue.prototype.$myMethod = function (options) {
    // algo de lógica
  }
}
```

## Utilizando un complemento

Utiliza un plugin ejecutando el método global `Vue.use()`:

``` js
// ejecuta `MyPlugin.install(Vue)`
Vue.use(MyPlugin)
```

Puedes pasar algunas opciones si lo necesitas:

``` js
Vue.use(MyPlugin, { someOption: true })
```

`Vue.use` automáticamente previene que utilices un complemento más de una vez, por lo que ejecutarlo repetidamente con el mismo complemento como parámetro solo lo instalará una vez.

Algunos complementos oficiales provistos por Vue.js como `vue-router` ejecutan automáticamente `Vue.use()` si `Vue` está disponible como una variable global. Sin embargo, en un ambiente modular como CommonJS, necesitas explícitamente ejecutar `Vue.use()`:

``` js
// Utilizando CommonJS a través de Browserify o Webpack
var Vue = require('vue')
var VueRouter = require('vue-router')

// ¡No te olvides de ejecutar esto!
Vue.use(VueRouter)
```

Échale un vistazo a [awesome-vue](https://github.com/vuejs/awesome-vue#libraries--plugins) para obtener un listado enorme de bibliotecas y complementos desarrollados por la comunidad.
