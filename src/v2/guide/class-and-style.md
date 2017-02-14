---
title: Enlace de estilos y clases
type: guia
order: 6
---

Una necesidad común cuando se enlazan datos es manipular la lista de clases de un elemento y sus estilos en línea. Dado que ambos son atributos, podemos utilizar `v-bind` para manejarlos: solo necesitamos calcular la cadena de texto final con nuestras expresiones. Sin embargo, lidiar con la concatenación de texto es molesto y propenso a errores. Por este motivo, Vue provee mejoras especiales cuando `v-bind` se utiliza en conjunto con `class` y `style`. Además de cadenas de texto, las expresiones peuden evaluar también objetos o arreglos:

## Enlazando clases HTML

### Sintaxis de objeto

Podemos pasar un objeto a `v-bind:class` para intercambiar clases dinámicamente:

``` html
<div v-bind:class="{ active: isActive }"></div>
```

La sintaxis anterior indica que la presencia de la clase `active` será determinada por [el valor de verdad](https://developer.mozilla.org/en-US/docs/Glossary/Truthy) de la propiedad `isActive`.

Puedes intercambiar múltiples clases agregando más campos al objeto. Además, la directiva `v-bind:class` puede co-existir con el atributo `class`. Entonces dada la siguiente plantilla:

``` html
<div class="static"
     v-bind:class="{ active: isActive, 'text-danger': hasError }">
</div>
```

Y los siguientes datos:

``` js
data: {
  isActive: true,
  hasError: false
}
```

Renderizará:

``` html
<div class="static active"></div>
```

Cuando `isActive` o `hasError` cambien, la lista de clases será actualizada en consecuencia. Por ejemplo, si `hasError` pasa a valer `true`, la lista de clases se convertirá en `"static active text-danger"`.

El objeto enlazado no tiene que ser declarado en línea:

``` html
<div v-bind:class="classObject"></div>
```
``` js
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```

Esto renderizará el mismo resultado. Podemos también enlazar a una [propiedad computada](computed.html) que devuelve un objeto. Este es un patrón muy común y poderoso:

``` html
<div v-bind:class="classObject"></div>
```
``` js
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal',
    }
  }
}
```

### Sintaxis de arreglo

Podemos pasar un arreglo a `v-bind:class` para aplicar una lista de clases:

``` html
<div v-bind:class="[activeClass, errorClass]">
```
``` js
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

Lo cual renderizará:

``` html
<div class="active text-danger"></div>
```

Si quisieras alternar una clase en la lista condicionalmente, puedes hacerlo con una expresión ternaria:

``` html
<div v-bind:class="[isActive ? activeClass : '', errorClass]">
```

Esto siempre aplicará la clase `errorClass`, pero solo `activeClass` cuando `isActive` valga `true`.

Sin embargo, esto puede resultar engorroso si tienes múltiples clases condicionales. Es por eso que también es posible utilizar la sintaxis de objetos dentro de la sintaxis de arreglos:

``` html
<div v-bind:class="[{ active: isActive }, errorClass]">
```

### Dentro de componentes

> Esta sección asume un conocimiento previo de los [componentes de Vue](components.html). Siéntete libre de saltearla y volver luego.

Cuando utilizas el atributo `class` en un componente personalizado, esas clases serán agregadas al elemento raíz del componente. Las clases existentes en el elemento no serán sobre escritas.

Por ejemplo, si declaras este componente:

``` js
Vue.component('my-component', {
  template: '<p class="foo bar">Hi</p>'
})
```

Y luego agregas algunas clases cuando lo utilizas:

``` html
<my-component class="baz boo"></my-component>
```

El HTML renderizado será:

``` html
<p class="foo bar baz boo">Hi</p>
```

Lo mismo aplica a clases enlazadas:

``` html
<my-component v-bind:class="{ active: isActive }"></my-component>
```

Cuando `isActive` sea verdadero, el HTML renderizado será:

``` html
<p class="foo bar active">Hi</p>
```

## Enlazando estilos en línea

### Sintaxis de objeto

La sintaxis de objeto para `v-bind:style` es bastante directa - es similar al CSS puro, excepto que es un objeto JavaScript. Puedes utilizar tanto _camelCase_ como _kebab-case_ (utiliza comillas con _kebab-case_) para los nombres de las propiedades CSS:

``` html
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```
``` js
data: {
  activeColor: 'red',
  fontSize: 30
}
```

Normalmente es una buena idea enlazar a un objeto de estilo directamente para que la plantilla sea más clara:

``` html
<div v-bind:style="styleObject"></div>
```
``` js
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
}
```

Nuevamente, la sintaxis de objeto es utilizada normalmente en conjunto con propiedades computadas que devuelven objetos.

### Sintaxis de arreglo

La sintaxis de arreglo para `v-bind:style` te permite aplicar múltiples objetos de estilo al mismo elemento:

``` html
<div v-bind:style="[baseStyles, overridingStyles]">
```

### Prefijos automáticos

Cuando utilizas una propiedad CSS en `v-bind:style` que requiere prefijos, por ejemplo `transform`, Vue lo detectará automáticamente y agregará los prefijos apropiados a los estilos aplicados.
