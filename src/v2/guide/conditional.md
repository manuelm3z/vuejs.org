---
title: Renderizado condicional
type: guia
order: 7
---

## `v-if`

En plantillas de cadena de texto, por ejemplo Handlebars, escribiríamos un bloque condicional de la siguiente manera:

``` html
<!-- Plantilla de Handlebars -->
{{#if ok}}
  <h1>Yes</h1>
{{/if}}
```

En Vue, utilizamos la directiva `v-if` para lograr el mismo resultado:

``` html
<h1 v-if="ok">Yes</h1>
```

También es posible agregar un bloque "else" con `v-else`:

``` html
<h1 v-if="ok">Yes</h1>
<h1 v-else>No</h1>
```

### Grupos condicionales con `v-if` dentro de `<template>`

Debido a que `v-if` es una directiva, tiene que ser añadida a un elemento en particular. Pero, ¿qué sucede si queremos mostrar/esconder más de un elemento? En este caso utilizamos `v-if` en un elemento `<template>`, el cual sirve como elemento envolvente invisible. El resultado del renderizado final no lo incluíra.

``` html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

### `v-else`

Puedes utilizar la directiva `v-else` para indicar un "bloque else" para `v-if`:

``` html
<div v-if="Math.random() > 0.5">
  Now you see me
</div>
<div v-else>
  Now you don't
</div>
```

Un elemento `v-else` debe encotrarse inmediatamente después de un elemento `v-if` o `v-else-if` - de otra forma, no será reconocido.

### `v-else-if`

> Nuevo en 2.1.0

`v-else-if`, como su nombre lo indica, sirve como un "bloque else if" para `v-if`. Puede ser encadenado varias veces seguidas:

```html
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```

Similar a `v-else`, un elemento `v-else-if` debe ubicarse inmediatamente luego de un elemento `v-if` o `v-else-if`.

### Controlando elementos reusables con `key`

Vue intenta renderizar elementos de la manera más eficiente posible, normalmente reutilizándolos en lugar de renderizarlos de cero. Más allá de ayudar a Vue a ser muy rápido, esto puede tener algunas ventajas muy útiles. Por ejemplo, si un usuario intenta alternar entre diferentes tipos de inicio de sesión:

``` html
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address">
</template>
```

Intercambiando `loginType` en el código anterior no borarrá lo que el usuario ya haya escrito. Dado que ambas plantillas utilizan los mismos elementos, `<input>` no es reemplazado, solo su `placeholder`.

Verifícalo tu mismo escribiendo algo en el campo de texto y luego presionando el botón "Toggle":

{% raw %}
<div id="no-key-example" class="demo">
  <div>
    <template v-if="loginType === 'username'">
      <label>Username</label>
      <input placeholder="Enter your username">
    </template>
    <template v-else>
      <label>Email</label>
      <input placeholder="Enter your email address">
    </template>
  </div>
  <button @click="toggleLoginType">Toggle login type</button>
</div>
<script>
new Vue({
  el: '#no-key-example',
  data: {
    loginType: 'username'
  },
  methods: {
    toggleLoginType: function () {
      return this.loginType = this.loginType === 'username' ? 'email' : 'username'
    }
  }
})
</script>
{% endraw %}

Sin embargo, este no siempre es el comportamiento deseado, por lo que Vue te ofrece una manera de decir: "Estos dos elementos están completamente separados, no los reutilices". Simplemente agrega el atributo `key` con un valor único:

``` html
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
```

Ahora esos campos de texto serán renderizados desde cero cada vez que los intercambies. Verifícalo:

{% raw %}
<div id="key-example" class="demo">
  <div>
    <template v-if="loginType === 'username'">
      <label>Username</label>
      <input placeholder="Enter your username" key="username-input">
    </template>
    <template v-else>
      <label>Email</label>
      <input placeholder="Enter your email address" key="email-input">
    </template>
  </div>
  <button @click="toggleLoginType">Toggle login type</button>
</div>
<script>
new Vue({
  el: '#key-example',
  data: {
    loginType: 'username'
  },
  methods: {
    toggleLoginType: function () {
      return this.loginType = this.loginType === 'username' ? 'email' : 'username'
    }
  }
})
</script>
{% endraw %}

Nota que los elementos `<label>` están siendo reutilizados eficientemente, porque no tienen el atributo `key`.

## `v-show`

Otra opción para mostrar condicionalmente un elemento es la directiva `v-show`. El uso es prácticamente el mismo:

``` html
<h1 v-show="ok">Hello!</h1>
```

La diferencia es que un elemento con `v-show` siempre será renderizado y permanecerá en el DOM. `v-show` simplemente alterna el valor de la propiedad CSS `display` del elemento.

<p class="tip">Nota que `v-show` no soporta la sintaxis `<template>` ni funciona con `v-else`.</p>

## `v-if` vs `v-show`

`v-if` es renderizado condicional "real" porque se asegura que los _listeners_ de eventos y componentes hijo dentro del bloque condicional sean destruidos y recreados apropiadamente durante los cambios de condición.

`v-if` es también **lazy**: si la condición es falsa durante el renderizado inicial, no hará nada. El bloque condicional no será renderizado hasta que la condición sea verdadera por primera vez.

En comparación. `v-show` es mucho más sencillo: el elemento siempre es renderizado sin importar el estado inicial de la condición, con una alternancia basada en CSS.

Generalmente, `v-if` tiene un costo de alternancia mayor mientras que `v-show` tiene un costo de renderizado inicial mayor. Entonces escoge`v-show` si necesitas alternar algo muy frecuentemente o `v-if` si es poco probable que la condición cambie durante la ejecución.

## `v-if` with `v-for`

Cuando se utiliza en conjunto con `v-for`, `v-for` tiene una prioridad mayor que `v-if`. Lee la <a href="../guide/list.html#V-for-and-v-if">guia de renderizado de listas</a> para más detalles.
