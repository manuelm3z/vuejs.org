---
title: Componentes
type: guia
order: 11
---

## ¿Qué son los componentes?

Los componentes son una de las características más poderosas de Vue. Te permiten extender elementos HTML básicos para encapsular código reutilizable. A grandes rasgos, los componentes son elementos personalizados a los que el compilador de Vue les añade comportamiento. En algunos casos, pueden aparecer como elementos HTML nativos extendidos con el atributo especial `is`.

## Utilizando componentes

### Registro

Hemos aprendido en las secciones anteriores que podemos crear una nueva instancia de Vue con:

``` js
new Vue({
  el: '#some-element',
  // opciones
})
```

Para registrar un componente global, puedes utilizar `Vue.component(tagName, options)`. Por ejemplo:

``` js
Vue.component('my-component', {
  // opciones
})
```

<p class="tip">Nota que Vue no te obliga a utilizar las [reglas de W3C](http://www.w3.org/TR/custom-elements/#concepts) para nombres de etiquetas personalizadas (todo en minúscula, con un guión medio) aunque seguir esta convención es considerado una buena práctica.</p>

Una vez registrado, un componente puede ser utilizado en la plantilla de una instancia como un elemento personalizado `<my-component></my-component>`. Asegúrate que el componente esté registrado **antes** de crear la instancia principal de Vue. Aquí hay un ejemplo completo:

``` html
<div id="example">
  <my-component></my-component>
</div>
```

``` js
// registro
Vue.component('my-component', {
  template: '<div>¡Mi componente!</div>'
})

// crear la instancia principal
new Vue({
  el: '#example'
})
```

Lo cual renderizará:

``` html
<div id="example">
  <div>¡Mi componente!</div>
</div>
```

{% raw %}
<div id="example" class="demo">
  <my-component></my-component>
</div>
<script>
Vue.component('my-component', {
  template: '<div>¡Mi componente!</div>'
})
new Vue({ el: '#example' })
</script>
{% endraw %}

### Registro local

No tienes que registrar cada componente globalmente. Puede hacer que un componente este disponible solo en el ámbito de otro componente/instancia registrándolo en la opción `components`:

``` js
var Child = {
  template: '<div>¡Mi componente!</div>'
}

new Vue({
  // ...
  components: {
    // <my-component> solo estará disponible en la plantilla del padre
    'my-component': Child
  }
})
```

El mismo encapsulamiento aplica para otras características registrables de Vue, como las directivas.

### Advertencias en el análisis de plantillas del DOM

Cuando utilizas el DOM como tu plantilla (por ejemplo, utilizando la opción `el` para montar un elemento con contenido existente), estarás sujeto a algunas restricciones que son inherentes a como trabaja el HTML, porque Vue solo puede recuperar el contenido de la plantilla **después** de que el navegador lo haya analizado y normalizado. Incluso, algunos elementos como `<ul>`, `<ol>`, `<table>` y `<select>` tienen restricciones acerca de qué elementos pueden contener, y otros como `<option>` solo pueden aparecer dentro de ciertos otros.

Esto conducirá a problemas cuando se utilicen componentes personalizados con elementos que tengas esas restricciones, por ejemplo:

``` html
<table>
  <my-row>...</my-row>
</table>
```

El componente personalizado `<my-row>` será marcado como contenido inválido, causando por ende errores en la salida renderizada. Una solución alternativa es utilizar el atributo especial `is`:

``` html
<table>
  <tr is="my-row"></tr>
</table>
```

**Debe notarse que estas limitaciones no aplican si estás utilizando plantillas de texto de alguna de las siguientes fuentes**:

- `<script type="text/x-template">`
- Plantillas de texto JavaScript en línea
- Componentes `.vue`

Por lo tanto, es preferible utilizar plantillas de texto siempre que sea posible.

### `data` debe ser una función

La mayoría de las opciones que pueden ser pasadas a un constructor de Vue pueden ser utilizadas en un componente, con un caso especial: `data` debe ser una función. De hecho, si intentas esto:

``` js
Vue.component('my-component', {
  template: '<span>{{ message }}</span>',
  data: {
    message: 'hello'
  }
})
```

Vue se detendrá y emitirá advertencias en la consola, diciéndote que `data` debe ser una función para instancias de componentes. Es bueno entender por qué existe la regla, así que hagamos algo de trampa:

``` html
<div id="example-2">
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
</div>
```

``` js
var data = { counter: 0 }

Vue.component('simple-counter', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  // 'data' es técnicamente una función, por lo que Vue no
  // se quejará, pero estamos devolviendo la misma referencia
  // de objeto en cada instancia de componente
  data: function () {
    return data
  }
})

new Vue({
  el: '#example-2'
})
```

{% raw %}
<div id="example-2" class="demo">
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
  <simple-counter></simple-counter>
</div>
<script>
var data = { counter: 0 }
Vue.component('simple-counter', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  data: function () {
    return data
  }
})
new Vue({
  el: '#example-2'
})
</script>
{% endraw %}

Dado que las tres instancias del componente comparten el mismo objeto `data`, ¡Al incrementar un contador se incrementa en todos! _Ouch_. Arreglemos esto retornando en su lugar un objeto de datos nuevo:

``` js
data: function () {
  return {
    counter: 0
  }
}
```

Ahora todos nuestros contadores tienen su propio estado interno:

{% raw %}
<div id="example-2-5" class="demo">
  <my-component></my-component>
  <my-component></my-component>
  <my-component></my-component>
</div>
<script>
Vue.component('my-component', {
  template: '<button v-on:click="counter += 1">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  }
})
new Vue({
  el: '#example-2-5'
})
</script>
{% endraw %}

### Componiendo componentes

Los componentes están pensados para ser utilizados en conjunto, comunmente en relaciones padre-hijo: el componente A puede utilizar al componente B en su propiar plantilla. Necesitarán inevitablemente comunicarse entre ellos: el padre puede necesitar pasar datos hacia el hijo, mientras que el hijo puede necesitar informar de algo que ha ocurrido al padre. Sin embargo, también es muy importante mantener desacopados al padre e hijo todo lo que sea posible a través de una interfaz definida claramente. Esto asegura que el código de cada componente puede ser escrito y razonadode forma independiente, haciéndolos más mantenibles y potencialmente fáciles de reutilizar.

En Vue.js, la relación padre-hijo entre componentes puede ser resumida como **propiedades hacia abajo, eventos hacia arriba**. El padre pasa datos al hijo a través de **propiedades** y el hijo envía mensajes al padre a través de **eventos**. Veamos como trabajan.

<p style="text-align: center">
  <img style="width:300px" src="/images/props-events.png" alt="props abajo, eventos arriba">
</p>

## Propiedades

### Pasando datos a través de propiedades

Cada instancia de un componente tiene su propio **ámbito aislado**. Esto significa que no puedes (y no debes) referenciar directamente datos del padre en la plantilla del componente hijo. Los datos pueden ser pasados hacia el hijo utilizando **propiedades**.

Una propiedad es un atributo personalizado para pasar información desde componentes padres. Un componente hijo necesita declarar explícitamente las propiedades que espera recibir utilizando la [opción `props`](../api/#props):

``` js
Vue.component('child', {
  // declara las propiedades
  props: ['message'],
  // como 'data', las propiedades pueden ser utilizadas dentro de las
  // plantillas y está disponibles en la vm como this.message
  template: '<span>{{ message }}</span>'
})
```

Entonces, puedes pasar una cadena de texto plana como:

``` html
<child message="hello!"></child>
```

Resultado:

{% raw %}
<div id="prop-example-1" class="demo">
  <child message="hello!"></child>
</div>
<script>
new Vue({
  el: '#prop-example-1',
  components: {
    child: {
      props: ['message'],
      template: '<span>{{ message }}</span>'
    }
  }
})
</script>
{% endraw %}

### camelCase vs. kebab-case

Los atributos HTML no distinguen entre mayúsculas y minúsculas, por lo que cuando utilices plantillas que no sean de texto, los nombres de propiedades en formato _camelCase_ necesitan ser escritas con su equivalente _kebab-case_:

``` js
Vue.component('child', {
  // camelCase en JavaScript
  props: ['myMessage'],
  template: '<span>{{ myMessage }}</span>'
})
```

``` html
<!-- kebab-case en HTML -->
<child my-message="hello!"></child>
```

De nuevo, si estás utilizando plantillas de texto está limitación no aplica.

### Propiedades dinámicas

Similar a enlazar atributos normales a una expresión, podemos utilizar `v-bind` para enlazar dinámicamente propiedades con datos en el padre. Cuando los datos sean actualizados en el padre, los cambios fluirán hacia el hijo:

``` html
<div>
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
```

Normalmente es más sencillo utilizar la forma corta de `v-bind`:

``` html
<child :my-message="parentMsg"></child>
```

Resultado:

{% raw %}
<div id="demo-2" class="demo">
  <input v-model="parentMsg">
  <br>
  <child v-bind:my-message="parentMsg"></child>
</div>
<script>
new Vue({
  el: '#demo-2',
  data: {
    parentMsg: 'Message from parent'
  },
  components: {
    child: {
      props: ['myMessage'],
      template: '<span>{{myMessage}}</span>'
    }
  }
})
</script>
{% endraw %}

### Literal vs dinámico

Un error común que los principiantes suelen cometer es tratar de pasar un número utilizando la sintaxis literal:

``` html
<!-- esto pasa la cadena de texto "1" -->
<comp some-prop="1"></comp>
```

Sin embargo, dado que esta es una propiedad literal, su valor se pasa como la cadena de texto `"1"` en lugar de un número. Si en realidad queremos pasar un número JavaScript, necesitamos utilizar `v-bind` para que su valor sea evaluado como una expresión JavaScript:

``` html
<!-- esto pasa el numero 1 -->
<comp v-bind:some-prop="1"></comp>
```

### Flujo de datos en un solo sentido

Todas las propiedades establecen un enlace de **un solo sentido** (_one-way data binding_) entre la propiedad del hijo y la del padre: cuando la propiedad del padre cambia, fuirá hacia el hijo, pero no en el sentido inverso. Esto previene que los componentes hijo modifiquen accidentalmente el estado del padre, lo cual puede hacer que el fujo de tu aplicación sea díficil de razonar.

Además, cada vez que el componente padre es actualizado, todas las propiedades en el componente hijo serán refrescadas con el último valor. Esto significa que **no** deberías mutar una propiedad en un componente hijo. Si lo haces, Vue te advertirá en la consola.

Hay usualmente dos casos en los que puede ser tentador mutar una propiedad:

1. La propiedad es utilizada solo para dar un valor inicial, el componente hijo solo quiere utilizarla como una propiedad de datos local luego.

2. La propiedad es pasada como un valor en bruto que luego necesita ser transformado.

Las respuestas apropiadas a estos casos de uso son:

1. Define una propiedad de datos local que utilice el valor inicial de la propiedad como su valor inicial:

  ``` js
  props: ['initialCounter'],
  data: function () {
    return { counter: this.initialCounter }
  }
  ```

2. Define una propiedad computada (`computed`) que sea calculada en base al valor de la propiedad (`props`):

  ``` js
  props: ['size'],
  computed: {
    normalizedSize: function () {
      return this.size.trim().toLowerCase()
    }
  }
  ```

<p class="tip">Nota que los objetos y _arrays_ en JavaScript son pasados por referencia, por lo que si la propiedad es un _array_ u objeto, modificarlos dentro del hijo **afectará** al estado del padre.</p>

### Validación de propiedades

Es posible especificar requerimientos para las propiedades que recibe un componente. Si no se cumple con un requerimiento, Vue emitirá advertencias. Esto es especialmente útil cuando estás creando componentes con la intención que sean utlizados por otros.

En lugar de definir las propiedades como un _array_ de cadenas de texto, puedes utilizar un objeto con los requerimientos de validación:

``` js
Vue.component('example', {
  props: {
    // verificación de tipo básica (`null` significa que acepta cualquier tipo)
    propA: Number,
    // múltiples tipos posibles
    propB: [String, Number],
    // cadena de texto requerida
    propC: {
      type: String,
      required: true
    },
    // un número con valor por defecto
    propD: {
      type: Number,
      default: 100
    },
    // Los valores por defecto de objectos/arrays deben ser devueltos
    // desde una función fábrica
    propE: {
      type: Object,
      default: function () {
        return { message: 'hello' }
      }
    },
    // función de validación personalizada
    propF: {
      validator: function (value) {
        return value > 10
      }
    }
  }
})
```

El `type` puede ser uno de los siguientes constructores nativos:

- String
- Number
- Boolean
- Function
- Object
- Array

Además, `type` puede ser una función constructora personalizada y la verificación será hecha con `instanceof`.

Cuando una validación de propiedad falla, Vue producirá una advertencia en la consola (si estás utilizando la versión de desarrollo).

## Eventos personalizados

Hemos aprendido que los padres pueden pasar datos hacia los hijos utilizando propiedades. Pero, ¿cómo nos comunicamos con el padre cuando algo sucede? Aquí es donde el sistema de eventos personalizados de Vue entra en juego.

### Utilizando `v-on` con eventos personalizados

Cada instancia de Vue implementa una [interfaz de eventos](../api/#Instance-Methods-Events), lo cual significa que puede:

- Escuchar un evento utilizando `$on(eventName)`
- Emitir un evento utilizando `$emit(eventName)`

<p class="tip">Nota que el sistema de eventos de Vue está separado de la [API EventTarget](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget) de los navegadores. Aunque funcionan de forma similar, `$on` y `$emit` **no** son alias para `addEventListener` y `dispatchEvent`.</p>

Además, un componente padre puede escuchar los eventos emitidos por un componente hijo utilizando `v-on` directamente en la plantilla donde el componente hijo está siendo utilizado.

<p class="tip">No puedes utilizar `$on` para escuchar eventos emitidos por hijos. Debes utilizar `v-on` directamente en la plantilla, como en el ejemplo debajo.</p>

Aquí hay un ejemplo:

``` html
<div id="counter-event-example">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
```

``` js
Vue.component('button-counter', {
  template: '<button v-on:click="increment">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})

new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
```

{% raw %}
<div id="counter-event-example" class="demo">
  <p>{{ total }}</p>
  <button-counter v-on:increment="incrementTotal"></button-counter>
  <button-counter v-on:increment="incrementTotal"></button-counter>
</div>
<script>
Vue.component('button-counter', {
  template: '<button v-on:click="increment">{{ counter }}</button>',
  data: function () {
    return {
      counter: 0
    }
  },
  methods: {
    increment: function () {
      this.counter += 1
      this.$emit('increment')
    }
  },
})
new Vue({
  el: '#counter-event-example',
  data: {
    total: 0
  },
  methods: {
    incrementTotal: function () {
      this.total += 1
    }
  }
})
</script>
{% endraw %}

En este ejemplo, es importante notar que el componente hijo todavía está completamente desacoplado de lo que pasa fuera de él. Todo lo que hace es reportar información acerca de su propia actividad, en caso de que a un componente padre pueda importarle.

#### Enlazando eventos nativos a componentes

Puede haber momentos en los que quieras escuchar un evento nativo en el elemento raíz de un componente. En estos casos, puedes utilizar el modificador `.native` para `v-on`. Por ejemplo:

``` html
<my-component v-on:click.native="doTheThing"></my-component>
```

### Componentes de campos de formularios utilizando eventos personalizados

Los eventos personalizados también pueden ser utilizados para crear campos personalizados que funcionen con `v-model`. Recuerda:

``` html
<input v-model="something">
```

es simplemente azúcar sintáctico para:

``` html
<input v-bind:value="something" v-on:input="something = $event.target.value">
```

Cuando se utiliza con un componente, se simplifica a:

``` html
<custom-input v-bind:value="something" v-on:input="something = arguments[0]"></custom-input>
```

Entonces, para que un componente trabaje con `v-model`, debe:

- aceptar una propiedad `value`
- emitir un evento `input` con el nuevo valor

Veámoslo en acción con un ejemplo muy sencillo de un campo de entrada de dinero:

``` html
<currency-input v-model="price"></currency-input>
```

``` js
Vue.component('currency-input', {
  template: '\
    <span>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)"\
      >\
    </span>\
  ',
  props: ['value'],
  methods: {
    // En lugar de actualizar el valor directamente, este
    // método es utilizado para dar formato y aplicar restricciones
    // al valor de la entrada
    updateValue: function (value) {
      var formattedValue = value
        // Remueve espacios en blanco de ambos lados
        .trim()
        // Acorta a dos decimales
        .slice(0, value.indexOf('.') + 3)
      // Si el valor no estaba normalizado aún,
      // lo sobrescribimos manualmente
      if (formattedValue !== value) {
        this.$refs.input.value = formattedValue
      }
      // Emite el valor numérico a través del evento 'input'
      this.$emit('input', Number(formattedValue))
    }
  }
})
```

{% raw %}
<div id="currency-input-example" class="demo">
  <currency-input v-model="price"></currency-input>
</div>
<script>
Vue.component('currency-input', {
  template: '\
    <span>\
      $\
      <input\
        ref="input"\
        v-bind:value="value"\
        v-on:input="updateValue($event.target.value)"\
      >\
    </span>\
  ',
  props: ['value'],
  methods: {
    updateValue: function (value) {
      var formattedValue = value
        .trim()
        .slice(0, value.indexOf('.') + 3)
      if (formattedValue !== value) {
        this.$refs.input.value = formattedValue
      }
      this.$emit('input', Number(formattedValue))
    }
  }
})
new Vue({
  el: '#currency-input-example',
  data: { price: '' }
})
</script>
{% endraw %}

La implementación de arriba es bastante ingenua. Por ejemplo, los usuarios pueden ingresar múltiples puntos e incluso letras algunas veces, ¡Aagh! Para aquellos que quieran ver un ejemplo no tan trivial aquí hay otro más robusto de un campo de entrada de dinero:

<iframe width="100%" height="300" src="https://jsfiddle.net/chrisvfritz/1oqjojjx/embedded/result,html,js" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

La interfaz de eventos puede ser usada para crear campos de entrada poco comunes. Por ejemplo, imagina estas posibilidades:

``` html
<voice-recognizer v-model="question"></voice-recognizer>
<webcam-gesture-reader v-model="gesture"></webcam-gesture-reader>
<webcam-retinal-scanner v-model="retinalImage"></webcam-retinal-scanner>
```

### Comunicación entre componentes sin relación padre/hijo

En ocasiones dos componentes pueden necesitar comunicarse entre ellos pero no son padre/hijo. En escenarios similares, puedes utilizar una instancia de Vue vacía como un bus central de eventos:

``` js
var bus = new Vue()
```
``` js
// en un método del componente A
bus.$emit('id-selected', 1)
```
``` js
// en el _hook created_ del componente B
bus.$on('id-selected', function (id) {
  // ...
})
```

En escenarios más complejos, deberías considerar emplear un [patrón para manejo dedicado del estado](state-management.html).

## Distribución de contenido con _slots_

Cuando se utilizan componentes, es usual querer componerlos como:

``` html
<app>
  <app-header></app-header>
  <app-footer></app-footer>
</app>
```

Hay dos cosas a notar aquí:

1. El componente `<app>` no sabe qué contenido puede ser necesario en el elemento donde será montado. Eso lo decide cualquier componente que esté utilizando `<app>`.

2. El componente `<app>` probablemente tenga su propia plantilla.

Para lograr que la composición funcione, necesitamos una manera de entrelazar el "contenido" del padre y la propia plantilla del componente. Este es un proceso llamado **distribución de contenido** (o "transclusión" si estás familiarizado con Angular). Vue.js implementa una API de distribución de contenido que está modelada basada en el [borrador actual de la especificación de componentes web](https://github.com/w3c/webcomponents/blob/gh-pages/proposals/Slots-Proposal.md), utilizando el elemento especial `<slot>` para trabajar como contenedor de distribución para el contenido original.

### Ámbito de compilación

Antes de sumergirnos en la API, clarifiquemos primero en qué ámbito son compilados los contenidos. Imagina una plantilla como esta:

``` html
<child-component>
  {{ message }}
</child-component>
```

¿`message` debería estar enlazado a los datos del padre o del hijo? La respuesta es al padre. Una regla sencilla para recordar el ámbito de los componentes es:

> Todo lo que se encuentre en la plantilla del padre es compilado en el ámbito del padre; todo lo que se encuentra en la plantilla del hijo, es compilado en el ámbito del hijo.

Un error común es tratar de enlazar una directiva en la plantilla del padre a un método/propiedad del hijo:

``` html
<!-- NO funciona -->
<child-component v-show="someChildProperty"></child-component>
```

Asumiendo que `someChildProperty` es una propiedad en el componenten hijo, los ejemplos anteriores no funcionarán. La plantilla del padre no tiene conocimiento del estado de un componente hijo.

Si necesitas enlazar directivas en el ámbito de un hijo en el nodo raíz de un componente, debes hacerlo en la plantilla propia de ese componente:

``` js
Vue.component('child-component', {
  // esto funciona, porque estamos en el ámbito correcto
  template: '<div v-show="someChildProperty">Child</div>',
  data: function () {
    return {
      someChildProperty: true
    }
  }
})
```

De manera similar, el contenido distribuido será compilado en el ámbito padre.

### _Slot_ único

El contenido del padre será **descartado** a menos que la plantilla del componente hijo contenga por lo menos un contenedor `<slot>`. Cuando haya solo un _slot_ sin atributos, el contenido completo será insertado en su posición del DOM, reemplazándolo.

Cualquier contenido originalmente dentro de la etiqueta `<slot>` es considerado **por defecto**. El contenido por defecto es compilado en el ámbito del hijo y solo será mostrado si el elemento de alojamiento está vacío y no tiene contenido para ser insertado.

Imagina que tenemos un componente llamado `my-component` con la siguiente plantilla:

``` html
<div>
  <h2>Título del hijo</h2>
  <slot>
    Esto solo se mostrará si no hay contenido para mostrar.
  </slot>
</div>
```

Y el padre que utiliza el componente:

``` html
<div>
  <h1>Título del padre</h1>
  <my-component>
    <p>Algo de contenido</p>
    <p>Más contenido</p>
  </my-component>
</div>
```

El resultado renderizado será:

``` html
<div>
  <h1>Título del padre</h1>
  <div>
    <h2>Título del hijo</h2>
    <p>Algo de contenido</p>
    <p>Más contenido</p>
  </div>
</div>
```

### _Slots_ con nombre

Los elementos `<slot>` tienen un atributo especial, `name`, el cual puede ser utilizado para personalizar aún más como debe ser distribuido el contenido. Puedes tener múltiples _slots_ con diferentes nombres. Un _slot_ con nombre se emparejará con cualquier elemento que tenga su correspondiente atributo `slot` en el fragmento del contenido.

Aún así, todavía puede haber un _slot_ sin nombre, el cual es el **_slot_ por defecto** que captura cualquier contenido que no haya coincidido anteriormente. Si no hay un _slot_ por defecto, el contenido que no haya coincidido será descartado.

Por ejemplo, imagina que tenemos un componente `app-layout` con la siguiente plantilla:

``` html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

La estructura del padre:

``` html
<app-layout>
  <h1 slot="header">Aquí podría ir el título de la página</h1>

  <p>Un párrafo para el contenido.</p>
  <p>Y otro párrafo.</p>

  <p slot="footer">Información de contacto</p>
</app-layout>
```

El renderizado resultante será:

``` html
<div class="container">
  <header>
    <h1>Aquí podría ir el título de la página</h1>
  </header>
  <main>
    <p>Un párrafo para el contenido.</p>
    <p>Y otro párrafo.</p>
  </main>
  <footer>
    <p>Información de contacto</p>
  </footer>
</div>
```

La API de distribución de contenido es un mecanismo muy útil cuando se diseñan componentes pensados para utilizarse compuestos con otros.

### _Slots_ con ámbito

> Nuevo en 2.1.0

Un _slot_ con ámbito es un tipo especial de _slot_ que funciona como una plantilla reutilizable (a la que se pueden pasar datos) en lugar de elementos ya renderizados.

En un componente hijo, simplemente pasa datos a un _slot_ como si estuvieses pasando propiedades a un componente:

``` html
<div class="child">
  <slot text="hola desde el hijo"></slot>
</div>
```

En el padre, un elemento `<template>` con el atributo especial `scope` indica que es una plantilla para un _slot_ con ámbito. El valor de `scope` es el nombre de una variable temporal que mantiene el objeto de propiedades pasado desde el hijo:

``` html
<div class="parent">
  <child>
    <template scope="props">
      <span>hola desde el padre</span>
      <span>{{ props.text }}</span>
    </template>
  </child>
</div>
```

Si renderizamos lo anterior, el resultado será:

``` html
<div class="parent">
  <div class="child">
    <span>hola desde el padre</span>
    <span>hola desde el hijo</span>
  </div>
</div>
```

Un caso de uso típico para _slots_ con ámbito sería un componente lista que permita al consumidor del componente personalizar como debería ser renderizado cada elemento en la lista:

``` html
<my-awesome-list :items="items">
  <!-- los 'slots' con ámbito pueden también tener nombre -->
  <template slot="item" scope="props">
    <li class="my-fancy-item">{{ props.text }}</li>
  </template>
</my-awesome-list>
```

Y la plantilla para el componente de lista:

``` html
<ul>
  <slot name="item"
    v-for="item in items"
    :text="item.text">
    <!-- contenido por defecto aquí -->
  </slot>
</ul>
```

## Componentes dinámicos

Puedes utilizar el mismo punto de montaje e intercambiar dinámicamente múltiples componentes utilizando el elemento reservado `<component>` y enlazarlo dinámicamente a su atributo `is`:

``` js
var vm = new Vue({
  el: '#example',
  data: {
    currentView: 'home'
  },
  components: {
    home: { /* ... */ },
    posts: { /* ... */ },
    archive: { /* ... */ }
  }
})
```

``` html
<component v-bind:is="currentView">
  <!-- ¡el componente cambia cuando vm.currentView cambia! -->
</component>
```

Si prefieres, puedes enlazar directamente a objetos de componente:

``` js
var Home = {
  template: '<p>Welcome home!</p>'
}

var vm = new Vue({
  el: '#example',
  data: {
    currentView: Home
  }
})
```

### `keep-alive`

Si quieres mantener en memoria los componentes que han sido sacados para preservar su estado o evitar un re-renderizado, puedes envolver un componente dinámico con un elemento `<keep-alive>`:

``` html
<keep-alive>
  <component :is="currentView">
    <!-- ¡los componentes inactivos serán guardados en una memoria caché! -->
  </component>
</keep-alive>
```

Más detalles acerca de `<keep-alive>` en la [referencia de la API](../api/#keep-alive).

## Miscelánea

### Creando componentes reutilizables

Cuando crees componentes, es bueno tener en cuenta si tu intención es reutilizarlo en algún otro lugar más adelante. Está bien que los componentes de un solo uso estén estrechamente acoplados, pero los componentes reutilizables deben definir una interfaz pública limpia y no suponer nada acerca del contexto en el que está siendo utilizado.

La API para un componente de Vue se divide en tres partes: propiedades, eventos y _slots_:

- Las **propiedades** (_props_) permiten al ambiente externo pasar datos al componente

- Los **eventos** permiten al componente disparar efectos secundarios en el ambiente externo

- Los **slots** permiten al ambiente externo componer al componente con contenido extra

Con la sintaxis corta para `v-bind` y `v-on`, las intenciones pueden ser comunicadas clara y exitosamente a la plantilla:

``` html
<my-component
  :foo="baz"
  :bar="qux"
  @event-a="doThis"
  @event-b="doThat"
>
  <img slot="icon" src="...">
  <p slot="main-text">Hello!</p>
</my-component>
```

### Referencia a componentes hijo

A pesar de la existencia de propiedades y eventos, a veces puede que necesites acceder directamente a un componente hijo en JavaScript. Para lograr esto tienes que asignarle un ID de referencia utilizando `ref`. Por ejemplo:

``` html
<div id="parent">
  <user-profile ref="profile"></user-profile>
</div>
```

``` js
var parent = new Vue({ el: '#parent' })
// accede a la instancia del componente hijo
var child = parent.$refs.profile
```

Cuando `ref` se utiliza en conjunto con `v-for`, la referencia que obtendrás será un _array_ de objetos que contienen componentes hijos espejados con la fuente de datos.

<p class="tip">`$refs` son asignadas luego de que el componente haya sido renderizado, y no es reactivo. Está pensado como una vía de escape para la manipulación directa de hijos. Debes evitar utilizar `$refs` en las plantillas o propiedades computadas.</p>

### Componentes asíncronos

En aplicaciones grandes, puede que necesitemos dividir la aplicación en porciones mas pequeñas y cargar un componente desde el servidor solo cuando es realmente utilizado. Para hacer esto más sencillo, Vue te permite definir tus componentes como una función factoría que resuelve asíncronicamente la definición de ese componente. Vue ejecutará la función factoría solo cuando el componente necesite ser renderizado y guardará en memoria caché el resultado para futuras re-renderizaciones. Por ejemplo:

``` js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // Pasa la definición del componente a la función callback de resolución
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

La función factoría recibe una función _callback_ `resolve`, la cual debe ser llamada cuando hayas recuperado la definición del componente desde tu servidor. Puedes también ejecutar `reject(reason)` para indicar que la carga ha fallado. `setTimeout` aquí está simplemente como demostración. Cómo se recupere el componente es una decisión totalmente a tu criterio. Un enfoque recomendado es utilizar los componentes asíncronos junto con el [_code splitting_ de Webpack](https://webpack.js.org/guides/code-splitting-require/):

``` js
Vue.component('async-webpack-example', function (resolve) {
  // Esta sintaxis 'require' especial indicará a Webpack que
  // automáticamente divida el código de tu módulos en diferentes módulos
  // los cuales serán cargados a través de peticiones Ajax.
  require(['./my-async-component'], resolve)
})
```

Puedes devolver también un `Promise` en la función factoría, por lo que con la sintaxis de Webpack 2 + ES2015 puedes hacer:

``` js
Vue.component(
  'async-webpack-example',
  () => import('./my-async-component')
)
```

<p class="tip">Si eres un usuario de <strong>Browserify</strong> y te gustaría utilizar componentes asíncronos, desafortunadamente su creador [ha dejado muy claro](https://github.com/substack/node-browserify/issues/58#issuecomment-21978224) que la carga asíncrona "no es algo que Browserify vaya a soportar". Oficialmente al menos. La comunidad ha encontrado [algunas soluciones alternativas](https://github.com/vuejs/vuejs.org/issues/620), las cuales pueden ser útiles para aplicaciones complejas ya existentes. Para cualquier otro escenario, recomendamos utilizar Webpack para tener soporte asíncrono de primera clase incorporado.</p>

### Convención de nombres de componentes

Cuando registres componentes (o propiedades), puedes utilizar kebab-case, camelCase, o TitleCase. A Vue no le importa.

``` js
// en una definición de componente
components: {
  // registra utilizando kebab-case
  'kebab-cased-component': { /* ... */ },
  // registra utilizando camelCase
  'camelCasedComponent': { /* ... */ },
  // registra utilizando TitleCase
  'TitleCasedComponent': { /* ... */ }
}
```

Dentro de las plantillas HTML, debes utilizar kebab-case:

``` html
<!-- siempre utiliza kebab-case en plantillas HTML -->
<kebab-cased-component></kebab-cased-component>
<camel-cased-component></camel-cased-component>
<title-cased-component></title-cased-component>
```

Cuando utilices plantillas de texto, no estás limitado por las restricciones de HTML. Eso significa que incluso en las plantillas, puedes referenciar a tus componentes y propiedades utilizando camelCase, TitleCase, o kebab-case:

``` html
<!-- ¡utiliza lo que quieras en plantillas de texto! -->
<my-component></my-component>
<myComponent></myComponent>
<MyComponent></MyComponent>
```

Si tu componente no recibe contenido a través de elementos `slot`, puedes incluso hacer que se auto-cierre con una `/` luego del nombre:

``` html
<my-component/>
```

De nuevo, esto _solo_ funciona con plantillas de texto, dado que los elementos personalizados auto-cerrados no son HTML válido y el analizador nativo de tu navegador no los entenderá.

### Componentes recursivos

Los componentes pueden invocarse recursivamente en su propia plantilla. Sin embargo, solo pueden hacerlo con la opción `name`:

``` js
name: 'unique-name-of-my-component'
```

Cuando registres un componente globalmente utilizando `Vue.component`, el ID global es asignado automáticamente como la opción `name` del componente.

``` js
Vue.component('unique-name-of-my-component', {
  // ...
})
```

Si no eres cuidadoso, los componentes recursivos pueden conducir a bucles infinitos:

``` js
name: 'stack-overflow',
template: '<div><stack-overflow></stack-overflow></div>'
```

Un componente como el anterior resultará en un error _"max stack size exceeded"_, así que asegúrate que la invocación recursiva sea condicional (ej: utiliza un `v-ìf` que en algún momento sea `false`).

### Referencias circulares entre componentes

Digamos que estas construyendo un árbol de directorios, como en Finder o Explorador de archivos. Puede que tengas un componente `tree-folder` con esta plantilla:

``` html
<p>
  <span>{{ folder.name }}</span>
  <tree-folder-contents :children="folder.children"/>
</p>
```

Luego un componente `tree-folder-contents` con esta otra plantilla:

``` html
<ul>
  <li v-for="child in children">
    <tree-folder v-if="child.children" :folder="child"/>
    <span v-else>{{ child.name }}</span>
  </li>
</ul>
```

Si lo piensas, verás que estos componentes serán en realidad descendientes _y_ ancestros uno del otro en el árbol, ¡una paradoja! Cuando registras componentes globalmente con `Vue.component`, esta paradoja se resuelve automáticamente por ti. Si es tu caso, deja de leer aquí.

Sin embargo, si estas requiriendo/importando componentes utilizando un __sistema de módulos__, por ejemplo Webpack o Browserify, obtendrás un error:

```
Failed to mount component: template or render function not defined.
```

Para explicar lo que está sucediendo, llamaré a nuestros componentes A y B. El sistema de módulos ve que necesita a A, pero primero A necesita a B, pero B necesita a A, pero A necesita a B, etc. Queda trabado en un bucle, no sabiendo como resolver adecuadamente cada componente sin resolver primero el otro. Para arreglar esto, necesitamos indicarle al sistema de módulos un punto en el cual pueda decir: "A necesitará a B _en algún momento_, pero no es necesario resolver B primero".

En nuestro caso, haré que ese punto sea el componente `tree-folder`. Sabemos que el hijo que crea la paradoja es el componente `tree-folder-contents`, por lo que esperaremos al _hook_ del ciclo de vida `beforeCreate` para registrarlo:

``` js
beforeCreate: function () {
  this.$options.components.TreeFolderContents = require('./tree-folder-contents.vue')
}
```

¡Problema resuelto!

### Plantillas en línea

Cuando el atributo especial `inline-template` está presente en un componente hijo, el componente utilizará su propio contenido interno como su plantilla, en lugar de tratarlo como contenido distribuido. Esto permite una mayor flexibilidad en la creación de plantillas.

``` html
<my-component inline-template>
  <div>
    <p>Estos serán compilados como la plantilla del mismo componente.</p>
    <p>No como la transclusion del contenido del padre.</p>
  </div>
</my-component>
```

Sin embargo, `inline-template` hace más difícil razonar acerca del ámbito de nuestra plantilla. Como buena práctica, es preferible definir plantillas dentro de componentes utilizando la opción `template` o en un elemento `template` dentro de un archivo `.vue`.

### X-Templates

Otra forma de definir plantillas es dentro de un elemento `script` con el tipo `text/x-template`, y luego referenciando la plantilla con un id. Por ejemplo:

``` html
<script type="text/x-template" id="hello-world-template">
  <p>Hello hello hello</p>
</script>
```

``` js
Vue.component('hello-world', {
  template: '#hello-world-template'
})
```

Esto puede ser útil para demostraciones con plantillas grandes o en aplicaciones extremadamente pequeñas, pero generalmente debe ser evitado porque separan las plantillas del resto de la definición del componente.

### Componentes estáticos baratos con `v-once`

Renderizar HTML plano es muy rápido en Vue, pero en ocasiones puede que tengas un componente que contiene **mucho** contenido estático. En estos casos, puedes asegurarte que sea evaluado solo una vez y agregado a la memoria caché añadiéndole la directiva `v-once` al elemento raíz, de la siguiente forma:

``` js
Vue.component('terms-of-service', {
  template: '\
    <div v-once>\
      <h1>Terms of Service</h1>\
      ... mucho contenido estático ...\
    </div>\
  '
})
```
