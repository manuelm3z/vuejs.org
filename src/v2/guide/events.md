---
title: Manejo de eventos
type: guia
order: 9
---

## Escuchando eventos

Podemos utilizar la directiva `v-on` para escuchar eventos del DOM y ejecutar algo de JavaScript cuando son disparados.

Por ejemplo:

``` html
<div id="example-1">
  <button v-on:click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
```
``` js
var example1 = new Vue({
  el: '#example-1',
  data: {
    counter: 0
  }
})
```

Resultado:

{% raw %}
<div id="example-1" class="demo">
  <button v-on:click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
<script>
var example1 = new Vue({
  el: '#example-1',
  data: {
    counter: 0
  }
})
</script>
{% endraw %}

## Métodos para controladoras de eventos

La lógica para muchas funciones controladoras de eventos puede ser un tanto compleja, por lo que mantener el código JavaScript en el valor del atributo `v-on` simplemente no es factible. Por esto es que `v-on` acepta el nombre del método que te gustaría ejecutar.

Por ejemplo:

``` html
<div id="example-2">
  <!-- `greet` is the name of a method defined below -->
  <button v-on:click="greet">Greet</button>
</div>
```

``` js
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // define métodos dentro del objeto `methods`
  methods: {
    greet: function (event) {
      // `this` dentro de los métodos apunta a la instancia de Vue
      alert('Hello ' + this.name + '!')
      // `event` es el evento nativo del DOM
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})

// puedes invocar métodos en JavaScript tamibién
example2.greet() // -> 'Hello Vue.js!'
```

Resultado:

{% raw %}
<div id="example-2" class="demo">
  <button v-on:click="greet">Greet</button>
</div>
<script>
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  methods: {
    greet: function (event) {
      alert('Hello ' + this.name + '!')
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})
</script>
{% endraw %}

## Métodos dentro de controladoras en línea

En lugar de enlazar directamente con el nombre de un método, podemos utilizar métodos en una declaración JavaScript en línea:

``` html
<div id="example-3">
  <button v-on:click="say('hi')">Say hi</button>
  <button v-on:click="say('what')">Say what</button>
</div>
```
``` js
new Vue({
  el: '#example-3',
  methods: {
    say: function (message) {
      alert(message)
    }
  }
})
```

Resultado:
{% raw %}
<div id="example-3" class="demo">
  <button v-on:click="say('hi')">Say hi</button>
  <button v-on:click="say('what')">Say what</button>
</div>
<script>
new Vue({
  el: '#example-3',
  methods: {
    say: function (message) {
      alert(message)
    }
  }
})
</script>
{% endraw %}

A veces necesitamos acceder al evento original del DOM en una declaración JavaScript en línea. Puedes pasarlo al método utilizando la variable especial `$event`:

``` html
<button v-on:click="warn('Form cannot be submitted yet.', $event)">Submit</button>
```

``` js
// ...
methods: {
  warn: function (message, event) {
    // ahora tenemos acceso al evento nativo
    if (event) event.preventDefault()
    alert(message)
  }
}
```

## Modificadores de eventos

Es una necesidad muy común llamar a `event.preventDefault()` o `event.stopPropagation()` dentro de las funciones controladoras. Aunque podemos hacerlo fácilmente dentro de los métodos, sería mejor si estos tuvieran solo la lógica acerca de los datos en lugar de lidiar con detalles del evento del DOM.

Para solucionar este problema, Vue provee **modificadores de eventos** para `v-on`. Recuerda que esos modificadores son sufijos en las directivas indicados con un punto.

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`

``` html
<!-- la propagación del evento 'click' será detenida -->
<a v-on:click.stop="doThis"></a>

<!-- el evento 'submit' no recargará la página -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- los modificadores pueden encadenarse -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- solo el modificador -->
<form v-on:submit.prevent></form>

<!-- usa el modo captura cuando agregas _listener_ de eventos -->
<div v-on:click.capture="doThis">...</div>

<!-- solo dispara la función controladora si event.target es este elemento -->
<!-- quiere decir, no un elemento hijo -->
<div v-on:click.self="doThat">...</div>
```

> Nuevo en 2.1.4

``` html
<!-- el evento 'click' será disparado una vez como máximo -->
<a v-on:click.once="doThis"></a>
```

A diferencia de otros modificadores, los cuales son exclusivos de los eventos nativos del DOM, el modificador `.once` puede ser usado también en [eventos de componentes](components.html#Using-v-on-with-Custom-Events). Si todavía no has leído acerca de los componentes, por ahroa no te preocupes por esta característica.

## Modificadores de teclas

Cuando escuchamos eventos del teclado, muchas veces necesitamos verificar códigos de teclas comunes. Vue también permite modificadores de teclas para `v-on` cuando escuchamos eventos de teclas:

``` html
<!-- solo ejecuta vm.submit() cuando keyCode es 13 -->
<input v-on:keyup.13="submit">
```

Recordar todos los códigos de teclas es una molestia, por lo que Vue provee alias para los más utilizados:

``` html
<!-- lo mismo que el ejemplo anterior -->
<input v-on:keyup.enter="submit">

<!-- también funciona en la forma corta -->
<input @keyup.enter="submit">
```

Aquí hay una lista completa de alias de modificadores de teclas:

- `.enter`
- `.tab`
- `.delete` (captura tanto las teclas "Delete" como "Backspace")
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

Puedes definir [alias personalizados para modificadores de teclas](../api/#keyCodes) a través del objeto global `config.keyCodes`:

``` js
// habilita v-on:keyup.f1
Vue.config.keyCodes.f1 = 112
```

## Teclas modificadoras

> Nuevo en 2.1.0

Puedes utilizar los siguientes modificadores para disparar _listener_ de eventos del teclado o el ratón solo cuando la correspondiente tecla modificadora es presionada:

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

> Nota: En los teclados Macintosh, meta es la tecla _comando_ (⌘). En teclados Windows, meta es la tecla windows (⊞). En teclados Sun Microsystems, meta es la tecla identificada con un diamante sólido (◆). En algunos teclados, específicamente en máquinas Lisp y MIT o sucesoras, como el teclado _Knight_ o _space-cadet_, meta está etiquetada como “META”. En teclados Symbolics, meta está etiquetada como “META” o “Meta”.

Por ejemplo:

```html
<!-- Alt + C -->
<input @keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
```

## ¿Por qué _listeners_ en el HTML?

Debes estar preocupado porque todo este enfoque de escucha de eventos viola las viejas y buenas reglas acerca de la "separación de intereses". Ten la garantía: dado que todas las funciones controladoras y expresiones en Vue están estrictamente enlazadas con el _ViewModel_ que controla la vista actual, no causará ningún problema de mantenimiento. De hecho, hay varios beneficios por utilizar `v-on`:

1. Es más sencillo localizar las implementaciones de las funciónes controladoras dentro de tu código JS simplemente echando un vistado a la plantilla HTML.

2. Dado que no tienes que agregar manualmente _listeners_ de eventos en JS, el código de tu _ViewModel_ puede ser lógica pura y estar libre del DOM. Esto lo hace más sencillo de probar.

3. Cuando un _ViewModel_ es destruido, todos los _listeners_ de eventos son removidos automáticamente. No necesitas preocuparte por quitarlos tú mismo.
