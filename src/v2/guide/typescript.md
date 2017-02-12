---
title: Soporte para TypeScript
type: guia
order: 25
---

## Archivos oficiales de declaración

Un sistema de tipos estáticos puede ayudar a prevenir varios errores potenciales en tiempo de ejecución, especialmente cuando la aplicación crece. Por esto es que Vue incluye [declaraciones oficiales de tipo](https://github.com/vuejs/vue/tree/dev/types) para [TypeScript](https://www.typescriptlang.org/) - no solo para Vue, sino también [para Vue Router](https://github.com/vuejs/vue-router/tree/dev/types) y [para Vuex](https://github.com/vuejs/vuex/tree/dev/types).

Dado que son publicadas en [NPM](https://unpkg.com/vue/types/), ni siquiera necesitas herramientas externas como `Typings`, ya que las declaraciones son importadas automáticamente con Vue. Esto significa que todo lo que necesitas es simplemente:

``` ts
import Vue = require('vue')
```

Entonces se verificarán los tipos de todos los métodos, propiedades y parámetros. Por ejemplo, si por un error de tipeo escribes `template` como `tempate` (sin la `l`), el compilador de TypeScript imprimirá un mensaje de error en tiempo de compilación. Si estás utilizando un editor que pueda _lintear_ TypeScript, como [Visual Studio Code](https://code.visualstudio.com/), será posible encontrar estos errores incluso antes de compilar:

![TypeScript Type Error in Visual Studio Code](/images/typescript-type-error.png)

### Opciones de compilación

Los archivos de declaración de Vue requieren [la opción de compilador](https://www.typescriptlang.org/docs/handbook/compiler-options.html) `--lib DOM,ES5,ES2015.Promise`. Puedes pasar esta opción al comando `tsc` o agregar el equivalente en un archivo `tsconfig.json`.

### Accediendo a las declaraciones de tipo de Vue

Si quieres _annotar_ a tu propio código con los tipos de Vue, puedes acceder a ellos en el objeto exportado de Vue. Por ejemplo, para _annotar_ un objeto de opciones de componente exportado (en un archivo `.vue`):

``` ts
import Vue = require('vue')

export default {
  props: ['message'],
  template: '<span>{{ message }}</span>'
} as Vue.ComponentOptions<Vue>
```

## Componentes de Vue como clases JavaScript

Las opciones de componentes de Vue pueden ser _annotadas_ fácilmente con tipos:

``` ts
import Vue = require('vue')

// Declara el tipo del componente
interface MyComponent extends Vue {
  message: string
  onClick (): void
}

export default {
  template: '<button @click="onClick">Click!</button>',
  data: function () {
    return {
      message: 'Hello!'
    }
  },
  methods: {
    onClick: function () {
      // TypeScript sabe que `this` es de tipo MyComponent
      // y que `this.message` será una cadena de texto
      window.alert(this.message)
    }
  }
// Necesitamos _annotar_ explícitamente el objeto de
// opciones exportado con el tipo MyComponent
} as Vue.ComponentOptions<MyComponent>
```

Desafortunadamente, hay algunas limitaciones aquí:

- __TypeScript no puede inferir todos los tipos de la API de Vue.__ Por ejemplo, no sabe que la propiedad `message` devuelta en nuestra función `data` será agregada a la instancia de `MyComponent`. Eso significa que si asignamos un valor númerico o booleano a `message`, los _linters_ y compiladores no serían capaces de lanzar un error, indicando que debería ser una cadena de texto.

- Debido a la limitación anterior, __annotando tipos de esta manera puede tornarse tedioso y largo__. La única razón por la que tenemos que declarar manualmente `message` como una cadena de texto es porque TypeScript no puede inferir el tipo en este caso.

Afortunadamente, [vue-class-component](https://github.com/vuejs/vue-class-component) puede resolver ambos problemas. Es una biblioteca oficial que te permite declarar componentes como clases nativas de JavaScript, con el decorador `@Component`. Como ejemplo, reescribamos el componente anterior:

``` ts
import Vue = require('vue')
import Component from 'vue-class-component'

// El decorador @Component indica que la clase es un componente de Vue
@Component({
  // Todas las opciones del componente están permitidas aquí
  template: '<button @click="onClick">Click!</button>'
})
export default class MyComponent extends Vue {
  // Los datos iniciales pueden ser declarados como propiedades de instancia
  message: string = 'Hello!'

  // Los métodos del componente pueden ser declarados por métodos de la instancia
  onClick (): void {
    window.alert(this.message)
  }
}
```

Con esta sintaxis alternativa, nuestra definición de componente no solo es más corta, sino que TypeScript puede también inferir los tipos de `message` y `onClick` sin declaraciones de interface explícitas. Esta estrategia incluso te permite manejar los tipos de las propiedades computadas, _hooks_ de ciclo de vida y funciones de renderizado. Para los detalles completos de uso, lee la [documentación de vue-class-component](https://github.com/vuejs/vue-class-component#vue-class-component).
