---
title: Introducción
type: guia
order: 2
---

## ¿Qué es Vue.js?

Vue (pronunciado /vjuː/ en inglés, como **view**) es un **framework progresivo** para construir interfaces de usuario. A diferencia de otros _frameworks_ monolíticos, Vue está diseñado desde el inicio para ser adoptado incrementalmente. La biblioteca principal se enfoca solo en la capa de la vista, y es muy simple de utilizar e integrar con otros proyectos o bibliotecas existentes. Por otro lado, Vue también es perfectamente capaz de soportar aplicaciones de una sola página sofisticadas cuando se utiliza en combinación con [herramientas modernas](single-file-components.html) y [bibliotecas de soporte](https://github.com/vuejs/awesome-vue#libraries--plugins).

Si eres un desarrollador _frontend_ con experiencia y quieres saber como Vue se compara con otras bibliotecas/frameworks, revisa [esta comparación](comparison.html).

## Empezando

<p class="tip">La guia oficial asume un conocimiento intermedio de HTML, CSS y JavaScript. Si eres totalmente nuevo en el desarrollo _frontend_, puede no ser la mejor idea empezar a utilizar un _framework_ - ¡aprende los conceptos básicos y luego vuelve aquí! La experiencia previa con otros _frameworks_ ayuda, pero no es obligatoria.</p>

La manera más sencilla de probar Vue.js es usando el [ejemplo "hola mundo" en JSFiddle](https://jsfiddle.net/chrisvfritz/50wL7mdz/). Siéntete libre de abrilo en otra pestaña y revisarlo a medida que avanzamos con ejemlos básicos. Sino, puedes crear un archivo `.html` e incluye Vue:

``` html
<script src="https://unpkg.com/vue/dist/vue.js"></script>
```

La [página de instalación](installation.html) provee más opciones para instalar Vue. Nota que **no** recomendamos a los principiantes comenzar con `vue-cli`, especialmente si no estás familiarizado con las herramientas de trabajo basadas en Node.js.

## Renderizado declarativo

En el corazón de Vue.js se encuentra un sistema que nos permite renderizar declarativamente datos en el DOM utilizando una sintaxis de plantillas directa:

``` html
<div id="app">
  {{ message }}
</div>
```
``` js
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
```
{% raw %}
<div id="app" class="demo">
  {{ message }}
</div>
<script>
var app = new Vue({
  el: '#app',
  data: {
    message: 'Hello Vue!'
  }
})
</script>
{% endraw %}

¡Ya hemos creado nuestra primera aplicación Vue! Esto parece bastante similar a renderizar una plantilla de texto, pero internamente Vue ha hecho muchas cosas. Los datos y el DOM están enlazados, y todo es **reactivo**. ¿Cómo lo sabemos? Abre la consola de JavaScript en tu navegador (ahora mismo, en esta página) y cambia el valor de `app.message`. Deberías ver el ejemplo renderizado actualizarse acorde a lo que has ingresado.

Además de interpolación de texto, tambíen podemos enlazar atributos de un elemento, por ejemplo:

``` html
<div id="app-2">
  <span v-bind:title="message">
    ¡Deja tu mouse sobre este mensaje unos segundos para ver el atributo `title` enlazado dinámicamente!
  </span>
</div>
```
``` js
var app2 = new Vue({
  el: '#app-2',
  data: {
    message: 'You loaded this page on ' + new Date()
  }
})
```
{% raw %}
<div id="app-2" class="demo">
  <span v-bind:title="message">
    ¡Deja tu mouse sobre este mensaje unos segundos para ver el atributo `title` enlazado dinámicamente!
  </span>
</div>
<script>
var app2 = new Vue({
  el: '#app-2',
  data: {
    message: 'You loaded this page on ' + new Date()
  }
})
</script>
{% endraw %}

Aquí nos encontramos con algo nuevo. El atributo `v-bind` que estás viendo es conocido como una **directiva**. Las directivas llevan el prefijo `v-` para indicar que son atributos especiales provistos por Vue y, como debes haber adivinado, aplican un comportamiento reactivo especial al DOM renderizado. En este caso, básicamente esta diciendo "manten el atributo `title` de este elemento enlazado con la propiedad `message` en la instancia de Vue".

Si abres nuevamente tu consola JavaScript y escribes `app2.message = 'some new message'`, verás que el HTML enlazado (en este caso, el atributo `title`) ha sido actualizado.

## Condicionales y bucles

Es bastante sencillo alternar la presencia de un elemento:

``` html
<div id="app-3">
  <p v-if="seen">Now you see me</p>
</div>
```

``` js
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
})
```

{% raw %}
<div id="app-3" class="demo">
  <span v-if="seen">Now you see me</span>
</div>
<script>
var app3 = new Vue({
  el: '#app-3',
  data: {
    seen: true
  }
})
</script>
{% endraw %}

Adelante, escribe `app3.seen = false` en la consola. Deberías ver desaparecer el mensaje.

Este ejemplo demuestra que no solo podemos enlazar datos con texto y atributos, sino también con la **estructura** del DOM. Además, Vue provee un sistema de transiciones muy poderoso que puede aplicar automáticamente [efectos de trancisión](transitions.html) cuando los elementos son agregados/actualizados/removidos por Vue.

Hay unas cuantas otras directivas, cada una con una funcionalidad especial. Por ejemplo, la directiva `v-for` puede ser utilizada para mostrar una lista de elementos usando los datos de un arreglo:

``` html
<div id="app-4">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>
```
``` js
var app4 = new Vue({
  el: '#app-4',
  data: {
    todos: [
      { text: 'Learn JavaScript' },
      { text: 'Learn Vue' },
      { text: 'Build something awesome' }
    ]
  }
})
```
{% raw %}
<div id="app-4" class="demo">
  <ol>
    <li v-for="todo in todos">
      {{ todo.text }}
    </li>
  </ol>
</div>
<script>
var app4 = new Vue({
  el: '#app-4',
  data: {
    todos: [
      { text: 'Learn JavaScript' },
      { text: 'Learn Vue' },
      { text: 'Build something awesome' }
    ]
  }
})
</script>
{% endraw %}

En la consola, escribe `app4.todos.push({ text: 'New item' })`. Deberías ver un nuevo elemento agregado a la lista.

## Manejando entradas de usuario

Para permitir a los usuarios interactuar con tu aplicación, podemos usar la directiva `v-on` para añadir _listeners_ de eventos que invocan métodos en nuestras instancias de Vue:

``` html
<div id="app-5">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">Reverse Message</button>
</div>
```
``` js
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
```
{% raw %}
<div id="app-5" class="demo">
  <p>{{ message }}</p>
  <button v-on:click="reverseMessage">Reverse Message</button>
</div>
<script>
var app5 = new Vue({
  el: '#app-5',
  data: {
    message: 'Hello Vue.js!'
  },
  methods: {
    reverseMessage: function () {
      this.message = this.message.split('').reverse().join('')
    }
  }
})
</script>
{% endraw %}

Nota que en el método simplemente actualizamos el estado de nuestra aplicación sin modificar del DOM - todas las manipulaciones del mismo son manejadas por Vue, y el código que escribes se enfoca en la lógica subyacente.

Vue también provee la directiva `v-model` que hace muy sencillo el enlace de dos vías entre un `input` de un formulario y el estado de la aplicación:

``` html
<div id="app-6">
  <p>{{ message }}</p>
  <input v-model="message">
</div>
```
``` js
var app6 = new Vue({
  el: '#app-6',
  data: {
    message: 'Hello Vue!'
  }
})
```
{% raw %}
<div id="app-6" class="demo">
  <p>{{ message }}</p>
  <input v-model="message">
</div>
<script>
var app6 = new Vue({
  el: '#app-6',
  data: {
    message: 'Hello Vue!'
  }
})
</script>
{% endraw %}

## Componentes

El sistema de componentes es otro concepto importante en Vue, porque es una abstracción que nos permite construir aplicaciones de gran escala compuestas por componentes pequeños, autocontenidos y, normalmente, reutilizables. Si lo pensamos, casi cualquier tipo de interface de aplicación puede ser representada de manera abstracta como un árbol de componentes:

![Component Tree](/images/components.png)

En Vue, un componente es esencialmente una instancia de Vue con opciones predefinidas. Registrar un componente en Vue es directo y sencillo:

``` js
// Define un nuevo componente llamado todo-item
Vue.component('todo-item', {
  template: '<li>This is a todo</li>'
})
```

Ahora puedes utilizarlo en la plantilla de otro componente:

``` html
<ol>
  <!-- Crea una instancia del componente todo-item -->
  <todo-item></todo-item>
</ol>
```

Pero esto renderizaría el mismo texto para cada _todo_, lo cual no es muy interesante. Deberíamos ser capaces de pasar datos desde el padre a los componentes hijo. Modifiquemos la definición del componente para aceptar [propiedades](components.html#Props):

``` js
Vue.component('todo-item', {
  // El componente todo-item ahora acepta
  // "prop", el cual es similar a un atributo personalizado
  // La rpopiedad se llama _todo_.
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
```

Ahora podemos pasar _todo_ a cada componente repetido utilizando `v-bind`:

``` html
<div id="app-7">
  <ol>
    <!-- Ahora le pasamos a cada todo-item with el objeto todo    -->
    <!-- que representa, para que su contenido pueda ser dinámico -->
    <todo-item v-for="item in groceryList" v-bind:todo="item"></todo-item>
  </ol>
</div>
```
``` js
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})

var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { text: 'Vegetables' },
      { text: 'Cheese' },
      { text: 'Whatever else humans are supposed to eat' }
    ]
  }
})
```
{% raw %}
<div id="app-7" class="demo">
  <ol>
    <todo-item v-for="item in groceryList" v-bind:todo="item"></todo-item>
  </ol>
</div>
<script>
Vue.component('todo-item', {
  props: ['todo'],
  template: '<li>{{ todo.text }}</li>'
})
var app7 = new Vue({
  el: '#app-7',
  data: {
    groceryList: [
      { text: 'Vegetables' },
      { text: 'Cheese' },
      { text: 'Whatever else humans are supposed to eat' }
    ]
  }
})
</script>
{% endraw %}

Este es simplemente un ejemplo inventado, pero hemos logrado separar nuestra aplicación en porciones más pequeñas, y el hijo está razonablemente desacoplado del padre a través de la interface de propiedades. Podemos mejorar aún más nuestro componente `<todo-item>` con una plantilla más compleja o diferente lógica sin afectar a la aplicación padre.

En aplicaciones grandes, es necesario dividir la aplicación entera en componentes para un desarrollo manejable. Hablaremos mucho más acerca de los componentes [más adelante en la guia](components.html), pero aquí tienes un ejemplo (imaginario) de como luciría una plantilla de aplicación utilizando componentes:

``` html
<div id="app">
  <app-nav></app-nav>
  <app-view>
    <app-sidebar></app-sidebar>
    <app-content></app-content>
  </app-view>
</div>
```

### Relación con los elementos personalizados

Puedes haber notado que los componentes de Vue son muy similares a los **Elementos Personalizados**, los cuales son parte de la [especificación de Componentes Web](http://www.w3.org/wiki/WebComponents/). Esto es porque la sintaxis de componente de Vue está modelada basándose en ideas de la especificación. Por ejemplo, los componentes de Vue implementan la [API de Slot](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md) y el atributo especial `is`. Sin embargo, hay unas cuantas diferencias clave:

1. La especificación de Componentes Web está en un estado borrador todavía, y no está implementada nativamente el todos los navegadores. En comparación, los componentes de Vue no requieren ningún _polyfill_ y funcionan consistentemente in todos los navegadores soportados (IE9 y superiores). Cuando se necesite, los componentes de Vue pueden ser envueltos dentro de un elemento personalizado nativo.

2. Los componentes de Vue proveen características importantes que no están disponibles en los elementos personalizados, siendo las más notables el flujo de datos entre componentes, la comunicación con eventos personalizados y la integración con herramientas de desarrollo.

## ¿Listo para más?

Hemos introducido brevemente las características más básicas del corazón de Vue.js - el resto de esta guia cubrirá estas y otras características avanzadas con mucho más detalle, ¡asegúrate de leerla entera!
