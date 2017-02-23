---
title: Directivas personalizadas
type: guia
order: 16
---

## Introducción

Además del conjunto por defecto de directivas en su núcleo (`v-model` y `v-show`), Vue también te permite registrar tus propias directivas personalizadas. Nota que en Vue 2.0, la forma básica de abstracción y reutilización de código es a través de componentes. Sin embargo, puede haber casos donde necesites acceso de bajo nivel al DOM con elementos puros, y es aquí donde las directivas personalizadas todavía serían útiles. Un ejemplo de ello sería hacer foco en un elemento `input` como el siguiente:

{% raw %}
<div id="simplest-directive-example" class="demo">
  <input v-focus>
</div>
<script>
Vue.directive('focus', {
  inserted: function (el) {
    el.focus()
  }
})
new Vue({
  el: '#simplest-directive-example'
})
</script>
{% endraw %}

Cuando la página cargue, ese elemento recibe el foco (nota: el autofoco no funciona en Safari). De hecho, si no has hecho clic en ningún lado desde que visitaste esta página, el campo de texto anterior debería tener el foco ahora mismo. Codifiquemos una directiva que lo realice:

``` js
// Registra una directiva personalizada global llamada v-focus
Vue.directive('focus', {
  // Cuando el elemento enlazado es insertado en el DOM
  inserted: function (el) {
    // Pon el foco en el elemento
    el.focus()
  }
})
```

Si en cambio quieres registrar una directiva localmente, los componentes también aceptan la opción `directives`:

``` js
directives: {
  focus: {
    // definición de la directiva
  }
}
```

Luego, dentro de una plantilla, puedes utilizar el nuevo atributo `v-focus` en cualquier elemento, por ejemplo:

``` html
<input v-focus>
```

## Funciones _hook_

Un objeto de definición de directivas puede proveer distintas funciones _hook_ (todas opcionales):

- `bind`: se ejecuta solo una vez, cuando la directiva se enlaza en primer término al elemento. Aquí es donde puedes realizar las configuraciones necesarias una vez.

- `inserted`: se ejecuta cuando el elemento enlazado ha sido insertado en su nodo padre (esto solo garantiza que el nodo padre exista, no que esté necesariamente dentro del documento).

- `update`: se ejecuta luego que el componente contenedor ha sido actualizado, __pero posiblemente antes que sus hijos hayan sido actualizados__. El valor de la directiva puede haber cambiado o no, pero puedes saltarte actualizaciones innecesarias comparando el valor actual del enlace con el anterior (más información debajo en parámetros _hook_).

- `componentUpdated`: se ejecuta luego de que el componente contenedor __y sus hijos__ han sido actualizados.

- `unbind`: se ejecuta solo una vez, cuando la directiva es desenlazada del elemento.

Exploraremos los parámetros pasados a estos _hooks_ (por ejemplo: `el`, `binding`, `vnode`, y `oldVnode`) en la siguiente sección.

## Parámetros de los _hooks_ de directivas

A los _hooks_ de directivas se les pasan estos parámetros:

- **el**: El elemento al cual está enlazada la directiva. Puede ser usado para manipular directamente el DOM.
- **binding**: Un objeto que contiene las siguientes propiedades.
  - **name**: El nombre de la directiva, sin el prefijo `v-`.
  - **value**: El valor pasado a la directiva. Por ejemplo en `v-my-directive="1 + 1"`, el valor sería `2`.
  - **oldValue**: El valor anterior, solo disponible en `update` y `componentUpdated`. El mismo existe haya o no cambiado.
  - **expression**: La expresión del enlace como una cadena de texto. Por ejemplo: en `v-my-directive="1 + 1"`, la expresión sería `"1 + 1"`.
  - **arg**: Los parámetros pasados a la directivas, si existen. Por ejemplo: en `v-my-directive:foo`, el parámetro sería `"foo"`.
  - **modifiers**: Un objeto que contiene modificadores, si existen. Por ejemplo: en `v-my-directive.foo.bar`, el objeto con los modificadores sería `{ foo: true, bar: true }`.
- **vnode**: El nodo virtual generado por el compilador de Vue. Ingresa a la [API de VNode](../api/#VNode-Interface) para más detalles.
- **oldVnode**: El nodo virtual anterior, solo disponible en los _hooks_ `update` y `componentUpdated`.

<p class="tip">Dejando a un lado `el`, deberías tratar estos argumentos como de solo lectura y nunca modificarlos. Si necesitas compartir información entre _hooks_, es recomendable hacerlo a través del [dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset) del elemento.</p>

Un ejemplo de una directiva personalizada utilizando algunas de estas propiedades:

``` html
<div id="hook-arguments-example" v-demo:hello.a.b="message"></div>
```

``` js
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})

new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})
```

{% raw %}
<div id="hook-arguments-example" v-demo:hello.a.b="message" class="demo"></div>
<script>
Vue.directive('demo', {
  bind: function (el, binding, vnode) {
    var s = JSON.stringify
    el.innerHTML =
      'name: '       + s(binding.name) + '<br>' +
      'value: '      + s(binding.value) + '<br>' +
      'expression: ' + s(binding.expression) + '<br>' +
      'argument: '   + s(binding.arg) + '<br>' +
      'modifiers: '  + s(binding.modifiers) + '<br>' +
      'vnode keys: ' + Object.keys(vnode).join(', ')
  }
})
new Vue({
  el: '#hook-arguments-example',
  data: {
    message: 'hello!'
  }
})
</script>
{% endraw %}

## Abreviación de funciones

En muchos casos, puedes querer el mismo comportamiento para `bind` y `update`, pero no te interesan los demás _hooks_. Por ejemplo:

``` js
Vue.directive('color-swatch', function (el, binding) {
  el.style.backgroundColor = binding.value
})
```

## Objetos literales

Si tus directivas necesitan múltiples valores, puedes pasar un objeto literal de JavaScript. Recuerda, las directivas pueden recibir cualquier expresión JavaScript válida.

``` html
<div v-demo="{ color: 'white', text: 'hello!' }"></div>
```

``` js
Vue.directive('demo', function (el, binding) {
  console.log(binding.value.color) // => "white"
  console.log(binding.value.text)  // => "hello!"
})
```
