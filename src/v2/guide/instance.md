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

A pesar de no estar estrictamente asociado con el [patrón MVVM](https://en.wikipedia.org/wiki/Model_View_ViewModel), el diseño de Vue estuvo parcialmente inspirado en él. Por convención, normalmente utilizamos la variable `vm` (abreviación de _ViewModel_) para referirnos a instancias de Vue.

Cuando creas una nueva instancia de Vue, necesitas pasar un **objeto de opciones**, el cual puede contener opciones para datos, una plantilla, el elemento donde montarla, métodos, _callbacks_ para el ciclo de vida, etc. Puedes encontrar la lista completa de opciones en la [documentación de referencia de la API](../api).

El constructor de `Vue` puede ser extendido para crear **constructores de componentes** reutilizables con opciones predefinidas:

``` js
var MyComponent = Vue.extend({
  // opciones de extensión
})

// todas las instancias de `MyComponent` son creadas con
// las opciones de extensión predefinidas
var myComponentInstance = new MyComponent()
```

Aunque es posible crear instancias extendidas imperativamente, la mayor parte del tiempo es recomendable componerlas declarativamente en plantillas como elementos personalizados. Hablaremos de ello en detalle en [el sistema de componentes](components.html). Por ahora, solo necesitas saber que todos los componentes de Vue son, esencialmente, instancias de Vue extendidas.

## Propiedades y métodos

Cada instancia de Vue es un **proxy** de todas las propiedades que se encuentran en su objeto `data`:

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

Debe tenerse en cuenta que solo las propiedades que están en el objeto `data` son **reactivas**. Si agregas una propiedad a la instancia después de haberla creado, no va a desencadenar actualizaciones a la vista. Veremos el sistema de la reactividad más adelante con detalle.

Además de las propiedades de datos, las instancias de Vue exponen una serie de propiedades y métodos de la instancia muy útiles. Estas propiedades y métodos tienen el prefijo `$` para diferenciarlos de las propiedades del objeto `data` que pasan por el _proxy_ de la instancia Vue. Por ejemplo:

``` js
var data = { a: 1 }
var vm = new Vue({
  el: '#example',
  data: data
})

vm.$data === data // -> verdadero
vm.$el === document.getElementById('example') // -> verdadero

// $watch es un método de la instancia
vm.$watch('a', function (nuevoValor, viejoValor) {
  // este callback se llamará cuando `vm.a` cambie
})
```

<p class="tip">No uses _[arrow functions](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Functions/Arrow_functions)_ en las propiedades de una instancia o _callback_ (ej: `vm.$watch('a', nuevoValor => this.miMetodo())`). Las _arrow functions_ están unidas al contexto de su padre, en consecuencia `this` no será la instancia Vue como esperarías y `this.miMetodo` será _undefined_.</p>

Consulta la [documentación de referencia de la API](../api) para la lista completa de propiedades y métodos de la instancia.

## Hooks del ciclo de vida de una instancia

Cada instancia de Vue recorre una serie de pasos de inicialización cuando es creada. Por ejemplo, necesita configurar la observación de los datos, compilar la plantilla, montar la instancia en el DOM, y actualizar el DOM cuando los datos cambien. Durante este proceso, la instancia de Vue invocará _hooks_ (ganchos) duante el ciclo de vida del component, esto nos dará la oportunidad de ejecutar nuestra lógica. Por ejemplo, el hook _[`created`](../api/#created)_ (creado) se llamará después de que la instancia haya sido creada.

``` js
var vm = new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` apunta a la instancia vm
    console.log('a is: ' + this.a)
  }
})
// -> "a is: 1"
```

Hay otros _hooks_ que serán llamados en diferentes momentos del ciclo de vida de una instancia. Por ejemplo _[`mounted`](../api/#mounted)_ (montado), _[`updated`](../api/#updated)_ (actualizado), y _[`destroyed`](../api/#destroyed)_ (destruido). Todos estos _hooks_ son llamados con el contexto `this` apuntando a la instancia Vue que lo ha invocado. Quizás te hayas estado preguntando donde vive el concepto de controlador en el mundo Vue y la respuesta es: no hay controladores. Tu lógica para un componente se repartirá entre estos _hooks_.

## Diagrama de ciclo de vida

Más abajo hay un diagrama que muestra el ciclo de vida de una instancia. No es necesario que entiendas todo lo que muestra ahora mismo, pero este diagrama te servirá como referencia en el futuro.

![Ciclo de vida](/images/lifecycle.png)
