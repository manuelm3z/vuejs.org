---
title: Renderizado de listas
type: guia
order: 8
---

## `v-for`

Podemos utilizar la directiva `v-for` para renderizar una lista de elementos basándonos en un arreglo. La directiva `v-for` requiere una sintaxis especial de la forma `item in items`, donde `items` es el arreglo de datos fuente e `item` es un **alias** para el elemento sobre el que se está iterando:

### uso básico

``` html
<ul id="example-1">
  <li v-for="item in items">
    {{ item.message }}
  </li>
</ul>
```

``` js
var example1 = new Vue({
  el: '#example-1',
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

Resultado:

{% raw %}
<ul id="example-1" class="demo">
  <li v-for="item in items">
    {{item.message}}
  </li>
</ul>
<script>
var example1 = new Vue({
  el: '#example-1',
  data: {
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  },
  watch: {
    items: function () {
      smoothScroll.animateScroll(null, '#example-1')
    }
  }
})
</script>
{% endraw %}

Dentro de los bloques `v-for` tenemos acceso total a las propiedades del ámbito del padre. `v-for` también soporta un segundo parámetro opcional para indicar el índice el elemento actual.

``` html
<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
```

``` js
var example2 = new Vue({
  el: '#example-2',
  data: {
    parentMessage: 'Parent',
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

Resultado:

{% raw%}
<ul id="example-2" class="demo">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
<script>
var example2 = new Vue({
  el: '#example-2',
  data: {
    parentMessage: 'Parent',
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  },
  watch: {
    items: function () {
      smoothScroll.animateScroll(null, '#example-2')
    }
  }
})
</script>
{% endraw %}

Puedes utilizar `of` como delimitador en lugar de `in`, para que la sintaxis sea más parecida a la utilizada en JavaScript para iteradores:

``` html
<div v-for="item of items"></div>
```

### `v-for` en <template>

Similar a `v-if`, puedes utilizar una etiqueta `<template>` con `v-for` para renderizar un bloque de múltiples elementos. Por ejemplo:

``` html
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider"></li>
  </template>
</ul>
```

### `v-for` con objetos

Puedes utilizar `v-for` para iterar a través de las propiedades de un objeto.

``` html
<ul id="repeat-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>
```

``` js
new Vue({
  el: '#repeat-object',
  data: {
    object: {
      firstName: 'John',
      lastName: 'Doe',
      age: 30
    }
  }
})
```

Resultado:

{% raw %}
<ul id="repeat-object" class="demo">
  <li v-for="value in object">
    {{ value }}
  </li>
</ul>
<script>
new Vue({
  el: '#repeat-object',
  data: {
    object: {
      firstName: 'John',
      lastName: 'Doe',
      age: 30
    }
  }
})
</script>
{% endraw %}

También puedes proveer un segundo argumento para la llave:

``` html
<div v-for="(value, key) in object">
  {{ key }} : {{ value }}
</div>
```

Y otro para el índice:

``` html
<div v-for="(value, key, index) in object">
  {{ index }}. {{ key }} : {{ value }}
</div>
```

<p class="tip">Cuando iteres sobre un objeto, el orden está basado en el orden de enumeración de `Object.keys()`, el cual **no** garantizamos que sea consistente en todas las implementaciones de motores JavaScript.</p>

### `v-for` con rangos

`v-for` puede recibir un entero. En este caso, repetirá la plantilla esa cantidad de veces.

``` html
<div>
  <span v-for="n in 10">{{ n }}</span>
</div>
```

Resultado:

{% raw %}
<div id="range" class="demo">
  <span v-for="n in 10">{{ n }} </span>
</div>
<script>
new Vue({ el: '#range' })
</script>
{% endraw %}

### Componentes y `v-for`

> Esta sección asume un conocimiento previo de los [componentes de Vue](components.html). Siéntete libre de saltearla y volver luego.

Puedes utilizar `v-for` directamente en un componente personalizado, como cualquier otro elemento normal:

``` html
<my-component v-for="item in items"></my-component>
```

Sin embargo, no pasará automáticamente ningún dato al componente, porque los componentes tienen ámbitos aislados propios. Para pasar los datos de la iteración al componente, debes utilizar propiedades:

``` html
<my-component
  v-for="(item, index) in items"
  v-bind:item="item"
  v-bind:index="index">
</my-component>
```

La razón para no inyectar automáticamente `item` al componente es que genera un acoplamiento estrecho entre el componente y el funcionamiento de `v-for`. Siendo explícito acerca del origen de los datos hace que el componente sea reutilizable en otras situaciones.

Aquí tienes un ejemplo completo de una lista de tareas pendientes:

``` html
<div id="todo-list-example">
  <input
    v-model="newTodoText"
    v-on:keyup.enter="addNewTodo"
    placeholder="Add a todo"
  >
  <ul>
    <li
      is="todo-item"
      v-for="(todo, index) in todos"
      v-bind:title="todo"
      v-on:remove="todos.splice(index, 1)"
    ></li>
  </ul>
</div>
```

``` js
Vue.component('todo-item', {
  template: '\
    <li>\
      {{ title }}\
      <button v-on:click="$emit(\'remove\')">X</button>\
    </li>\
  ',
  props: ['title']
})

new Vue({
  el: '#todo-list-example',
  data: {
    newTodoText: '',
    todos: [
      'Do the dishes',
      'Take out the trash',
      'Mow the lawn'
    ]
  },
  methods: {
    addNewTodo: function () {
      this.todos.push(this.newTodoText)
      this.newTodoText = ''
    }
  }
})
```

{% raw %}
<div id="todo-list-example" class="demo">
  <input
    v-model="newTodoText" v
    v-on:keyup.enter="addNewTodo"
    placeholder="Add a todo"
  >
  <ul>
    <li
      is="todo-item"
      v-for="(todo, index) in todos"
      v-bind:title="todo"
      v-on:remove="todos.splice(index, 1)"
    ></li>
  </ul>
</div>
<script>
Vue.component('todo-item', {
  template: '\
    <li>\
      {{ title }}\
      <button v-on:click="$emit(\'remove\')">X</button>\
    </li>\
  ',
  props: ['title']
})
new Vue({
  el: '#todo-list-example',
  data: {
    newTodoText: '',
    todos: [
      'Do the dishes',
      'Take out the trash',
      'Mow the lawn'
    ]
  },
  methods: {
    addNewTodo: function () {
      this.todos.push(this.newTodoText)
      this.newTodoText = ''
    }
  }
})
</script>
{% endraw %}

### `v-for` con `v-if`

Cuando existen en el mismo nodo, `v-for` tiene mayor prioridad que `v-if`. Esto significa que `v-if` será ejecutado en cada iteración del bucle separadamente. Esto es muy útil cuando quieres renderizar nodos solo para _algunos_ elementos, como:

``` html
<li v-for="todo in todos" v-if="!todo.isComplete">
  {{ todo }}
</li>
```

Lo anterior solo renderizará los _todos_ que no hayan sido completados.

Si, en cambio, tu intención es saltearte condicionalmente la ejecución del bucle, puedes utilizar `v-if` en un elemento envolvente (o [`<template>`](conditional.html#Conditional-Groups-with-v-if-on-lt-template-gt)). Por ejemplo:

``` html
<ul v-if="shouldRenderTodos">
  <li v-for="todo in todos">
    {{ todo }}
  </li>
</ul>
```

## `key`

Cuando Vue está actualizando una lista de elementos renderizados con `v-for`, por defecto utiliza una estrategia "in-place patch". Si el orden de los elementos en los datos ha cambiado, en lugar de mover los elementos del DOM para coincidir con el orden de los elementos en los datos, Vue simplemente actualizará cada elemento en su lugar y se asegurará que refleje lo que debería ser renderizado en ese índice en particular. Esto es similar al comportamiento de `track-by="$index"` en Vue 1.x.

Este modo por defecto es eficiente, pero solo conveniente **cuando la lista renderizada no depende del estado de componentes hijos o de estado temporario del DOM (por ejemplo, campos de texto de un formulario)**.

Para darle una pista a Vue de que debe llevar un seguimiento de la identidad de cada nodo, y entonces reusar y reordenar elementos existentes, necesitas proveer un atributo `key` único para cada elemento. Un valor ideal para `key` sería un id único. Este atributo especial es un equivalente aproximado a `track-by` en 1.x, pero funciona como un atributo, por lo que necesitas utilizar `v-bind` para enlazarlo con valores dinámicos (utilizamos la forma corta aquí):

``` html
<div v-for="item in items" :key="item.id">
  <!-- content -->
</div>
```

Es recomendable proveer un `key` para `v-for` cuando sea posible , a menos que el contenido del DOM iterado sea simple, o estarás confiando en el comportamiento por defecto para las ganancias de rendimiento.

Dado que es un mecanismo genérico para identificar nodos, `key` también tiene otros usos que no están relacionados específicamente con `v-for`, como veremos más adelante en la guia.

## Detección de cambios en un arreglo

### Métodos de mutación

Vue envuelve los métodos de mutacion de los arreglos observados por lo que también disparará actualizaciones de las vistas. Los métodos envueltos son: 

- `push()`
- `pop()`
- `shift()`
- `unshift()`
- `splice()`
- `sort()`
- `reverse()`

Puedes abrir la consola y jugar con el arreglo `items` de los ejemplos anteriores ejecutando sus métodos de mutación. Por ejemplo: `example1.items.push({ message: 'Baz' })`.

### Reemplazando un arreglo

Los métodos de mutación, como su nombre sugiere, modifican el arreglo original sobre el cual actúan. En comparación, hay también métodos no-mutantes, por ejemplo: `filter()`, `concat()` y `slice()`, los cuales no modifican el arreglo original sino que **siempre devuelven un nuevo arreglo**. Cuando trabajes con métodos no mutantes, puedes simplemente reemplazar el arreglo anterior por el nuevo:

``` js
example1.items = example1.items.filter(function (item) {
  return item.message.match(/Foo/)
})
```

Puedes pensar que esto hará que Vue tiré todo el DOM existente y re-renderice la lista entera. Por suerte, no es el caso. Vue implementa algunas heurísticas inteligentes para reutilizar al máximo los elementos del DOM, por lo que reemplazar un arreglo existente por otro que contiene objetos solapados es una operación muy eficiente.

### Advertencias

Debido a las limitaciones en JavaScript, Vue **no puede** detectar los siguientes cambios en un arreglo:

1. Cuando intentas establecer el valor de un elemento a través del índice: `vm.items[indexOfItem] = newValue`
2. Cuando intentas modificar el largo de un arreglo: `vm.items.length = newLength`

Para solucionar el problema 1, las siguientes dos opciones lograrán el mismo resultado que `vm.items[indexOfItem] = newValue`, pero también dispararán actualizaciones de estado en el sistema de reactividad:

``` js
// Vue.set
Vue.set(example1.items, indexOfItem, newValue)
```
``` js
// Array.prototype.splice`
example1.items.splice(indexOfItem, 1, newValue)
```

Para solucionar el problema 2, puedes utilizar también `splice`:

``` js
example1.items.splice(newLength)
```

## Mostrando resultados filtrados/ordenados

En ocasiones queremos mostrar una versión filtrada u ordenada de un arreglo sin tener que modificar o restablecer los datos originales. En este caso, puedes crear una propiedad computada que devuelva el arreglo filtrado u ordenado:

Por ejemplo:

``` html
<li v-for="n in evenNumbers">{{ n }}</li>
```

``` js
data: {
  numbers: [ 1, 2, 3, 4, 5 ]
},
computed: {
  evenNumbers: function () {
    return this.numbers.filter(function (number) {
      return number % 2 === 0
    })
  }
}
```

Como alternativa, puedes utilizar un método donde las propiedades computadas no son factibles (por ejemplo, dentro de bucles `v-for` anidados):

``` html
<li v-for="n in even(numbers)">{{ n }}</li>
```

``` js
data: {
  numbers: [ 1, 2, 3, 4, 5 ]
},
methods: {
  even: function (numbers) {
    return numbers.filter(function (number) {
      return number % 2 === 0
    })
  }
}
```
