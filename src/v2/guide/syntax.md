---
title: Sintaxis de plantillas
type: guia
order: 4
---

Vue.js utiliza una sintaxis de plantilla basada en HTML lo que te permite enlazar declarativamente el DOM con los datos de la instancia de Vue subyacente. Todas las planitllas de Vue.js están compuestas por HTML válido que puede ser analizadas por navegadores compatibles con las especifiaciones o analizadores HTML.

Internamente, Vue compila las plantillas a funciones de renderizado de DOM Virtual. En combinación con el sistema de reactividad, Vue es capaz de descifrar inteligentemente cual es la cantidad mínima de componentes a re-renderizar y aplicar la menor cantidad posible de manipulaciones al DOM cuando el estado de la aplicación cambia.

Si estas familiarizado con los conceptos del DOM Virtual y prefieres el poder de JavaScript puro, puedes también [escribir directamente funciones de renderizado](render-function.html) en lugar de plantillas, con soporte opcional para JSX.

## Interpolations

### Text

The most basic form of data binding is text interpolation using the "Mustache" syntax (double curly braces):

``` html
<span>Message: {{ msg }}</span>
```

The mustache tag will be replaced with the value of the `msg` property on the corresponding data object. It will also be updated whenever the data object's `msg` property changes.

You can also perform one-time interpolations that do not update on data change by using the [v-once directive](../api/#v-once), but keep in mind this will also affect any binding on the same node:

``` html
<span v-once>This will never change: {{ msg }}</span>
```

### HTML Puro

Las llaves dobles interpretan los datos como texto plano, no HTML. Si deseas mostrar HTML real, necesitarás usar la directiva `v-html`:

``` html
<div v-html="rawHtml"></div>
```

El contenido es insertado como HTML puro - los enlaces de datos son ignorados. Nota que no puedes utilizar `v-html` para componer plantillas parciales, porque Vue no es un motor de plantillas basado en cadenas de texto. En su lugar, se prefiere utilizar a los componentes como unidad fundamental para la reutilización de UI y la composición.

<p class="tip">Renderizar dinámicamente HTML arbitrario en tu sitio web puede ser muy peligroso ya que conduce a [vulnerabilidades XSS](https://en.wikipedia.org/wiki/Cross-site_scripting). Utiliza interpolación HTML solo con contenido de confianza y **nunca** con contenido provisto por el usuario.</p>

### Atributos

Las llaves no deben ser utilizadas dentro de atributos HTML, en su lugar utiliza la [directiva v-bind](../api/#v-bind):

``` html
<div v-bind:id="dynamicId"></div>
```

También funciona para atributos booleanos - el atributo sera quitado si la condición se evalúa como falsa:

``` html
<button v-bind:disabled="someDynamicCondition">Button</button>
```

### Utilizando expresiones JavaScript

Hasta ahora, solo hemos estado enlazando a propiedades simples en nuestras plantillas. Pero Vue.js en realidad soporta todo el poder de las expresiones JavaScript dentro de cualquier enlace con los datos:

``` html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```

Estas expresiones serán evaluadas como JavaScript en el ámbito de datos de la instancia de Vue. Una restricción es que cada enlace puede contener solo **una expresión simple**, por lo que lo siguiente **NO** funcionará:

``` html
<!-- esto es una declaración, no una expresión: -->
{{ var a = 1 }}

<!-- el flujo de control tampoco funcionará, utiliza expresiones ternarias -->
{{ if (ok) { return message } }}
```

<p class="tip">Las expresiones en las plantillas y tiene acceso restringido a una lista blanca de variables globales como `Math` y `Date`. No debes intentar acceder a variables globales definidas por el usuario dentro de expresiones en las plantillas.</p>

## Directivas

Las directivas son atributos especiales identificadas con el prefijo `v-`. Los valores de los atributos de directivas deben ser **una sola expresión JavaScript** (con la excepción de `v-for`, el cual discutiremos luego). El trabajo de una directiva es aplicar reactivamente efectos secundarios al DOM cuando el valor de su expresión cambia. Veamos el ejemplo que utilizamos en la introducción:

``` html
<p v-if="seen">Now you see me</p>
```

Aquí, la directiva `v-if` removería/insertaría el elemento `<p>` basada en la veracidad del valor de la expresión `seen`.

### Argumentos

Algunas directivas pueden recibir un "argumento", indentificado con dos puntos luego del nombre de la directiva. Por ejemplo, la directiva `v-bind` se utiliza para actualizar reactivamente un atributo HTML:

``` html
<a v-bind:href="url"></a>
```

Aquí `href` es el argumento, el cual le indica a la directiva `v-bind` que enlace el atributo `href` del elemento con el valor de la expresión`url`.

Otro ejemplo es la directiva `v-on`, la cual escucha eventos del DOM:

``` html
<a v-on:click="doSomething">
```

Aquí el argumento es el nombre del evento que debe escuchar. Hablaremos acerca del manejo de enventos en detalle tambíen.

### Modificadores

Los modificadores son sufijos especiales identificados con un punto, los cuales indican que la directiva debe ser enlazada de alguna forma especial. Por ejemplo, el modificador `.prevent` indica a la directiva `v-on` que ejecute `event.preventDefault()` en el evento disparado:

``` html
<form v-on:submit.prevent="onSubmit"></form>
```

Veremos más usos de los modificadores cuando hablemos en detalle de `v-on` y `v-model`.

## Filtros

Vue.js te permite definir filtros que pueden ser usados para aplicar formatos de texto comunes. Pueden ser utilizados en dos lugares: **en la interpolación con llaves y las expresiones `v-bind`**. Los filtros deben ser agregados al final de las expresiones JavaScript, luego de un símbolo de tubería:

``` html
<!-- en interpolación de texto -->
{{ message | capitalize }}

<!-- en v-bind -->
<div v-bind:id="rawId | formatId"></div>
```

<p class="tip">Los filtros de Vue 2.x solo pueden ser usados dentro de interpolaciónes con llaves y expresiones `v-bind` (esto último soportado desde la versión 2.1.0) porque están diseñados principalmente para transformar texto. Para transformaciones de datos más complejas en otras directivas, deberías utilizar en su lugar [propiedades computadas](computed.html).</p>

Los filtros siempre reciben el valor de la expresión como primer parámetro.

``` js
new Vue({
  // ...
  filters: {
    capitalize: function (value) {
      if (!value) return ''
      value = value.toString()
      return value.charAt(0).toUpperCase() + value.slice(1)
    }
  }
})
```

Pueden ser encadenados:

``` html
{{ message | filterA | filterB }}
```

Son funciones JavaScript, por lo que pueden recibir parámetros:

``` html
{{ message | filterA('arg1', arg2) }}
```

Aquí, la cadena de texto `'arg1'` será pasada al filtro como segundo parámetro, y el valor de la expresión `arg2` será evaluado y pasado como tercer parámetro.

## Atajos

El prefijo `v-` sirve como ayuda visual para identificar atributos específicos de Vue en tus plantillas. Esto es útil cuando estás utilizando Vue.js para añadir comportamiento dinámico a una estructura existente, pero puede tornarse repetitivo para algunas directivas utilizadas frecuentemente. A la vez, la necesidad del prefijo `v-` se vuelve menos importante cuando estás construeyendo una [SPA](https://en.wikipedia.org/wiki/Single-page_application) donde Vue.js controla todas las plantillas. Por lo tanto, Vue.js provee atajos especiales para dos de las directivas más utilizadas, `v-bind` y `v-on`:

### Atajo para `v-bind`

``` html
<!-- sintaxis completa -->
<a v-bind:href="url"></a>

<!-- atajo -->
<a :href="url"></a>
```


### Atajo para `v-on`

``` html
<!-- sintaxis completa -->
<a v-on:click="doSomething"></a>

<!-- atajo -->
<a @click="doSomething"></a>
```

Pueden parecer un poco diferentes al HTML normal, pero `:` y `@` son carácteres válidos para nombres de atributo y todos los navegadores soportados por Vue.js los pueden analizar correctamente. Además, no aparecen en la estructura renderizada final. La sintaxis corta es totalmente opcional, pero seguramente te gustará cuando aprendas más acerca de su uso.
