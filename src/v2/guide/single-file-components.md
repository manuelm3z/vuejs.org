---
title: Componentes de un solo archivo
type: guia
order: 19
---

## Introducción

En muchos proyectos Vue, los componentes globales serán definidos utilizando `Vue.component`, seguido de `new Vue({ el: '#container' })` para especificar un elemento contenedor en el cuerpo de cada página.

Esto puede funcionar muy bien en proyectos pequeños a medianos, donde JavaScript se utiliza solo para mejorar ciertas vistas. Sin embargo, en proyectos más complejos o donde el lado cliente sea manejado totalmente con JavaScript, las siguientes desventajas se vuelven obvias:

- **Definiciones globales** fuerza a definir nombres únicos para cada componente
- **Plantillas como cadenas de texto** no se obtiene sintaxis resaltada y requiere utilizar horribles barras para HTML multilínea
- **Sin soporte CSS** significa que mientras el HTML y JavaScript son modularizados en componentes, el CSS es claramente dejado de lado
- **Sin proceso de construcción** nos restring a utilizar HTML y ES5 JavaScript, en lugar de preprocesadores como Pug (anteriormente Jade) y Babel

Todo lo anterior es resuelto mediante **componentes de un solo archivo** con extensión `.vue`, gracias a herramientas de construcción como Webpack o Browserify.

Aquí hay un ejemplo sencillo de un archivo que llamaremos `Hello.vue`:

<img src="/images/vue-component.png" style="display: block; margin: 30px auto">

Ahora obtenemos:

- [Sintaxis resaltada](https://github.com/vuejs/awesome-vue#syntax-highlighting)
- [Módulos CommonJS](https://webpack.js.org/concepts/modules/#what-is-a-webpack-module)
- [CSS en el ámbito del componente](https://github.com/vuejs/vue-loader/blob/master/docs/en/features/scoped-css.md)

Como prometimos, también podemos usar preprocesadores como Pug, Babel (con módulos ES2015) y Stylus para obtener componentes más limpios y con mejores características.

<img src="/images/vue-component-with-preprocessors.png" style="display: block; margin: 30px auto">

Estos lenguajes específicos son solo ejemplos. Podrías utilizar igual de fácil Bublé, TypeScript, SCSS, PostCSS - o cualquier otro preprocesador que te ayude a ser productivo. Si utilizas Webpack con `vue-loader`, también tienes soporte _first-class_ para módulos CSS.

### ¿Qué sucede con la separación de intereses?

Una cosa importante a tener en cuenta es que **separación de intereses no es equivalente a separación de tipos de archivo.** El el desarrollo UI moderno, hemos descubierto que en lugar de dividir el código base en tres capas enormes que se entrelazan entre ellas, tiene mucho más sentido dividirlas en componentes poco acoplados y componerlas. Dentro de un componente, su plantilla, lógica y estilos están inherentemente acoplados, y agruparlos hace que el componente sea más cohesivo y fácil de mantener.

Incluso si no te gusta la idea de los componentes de un solo archivo, puedes tomar ventaja de sus características de recarga en caliente y precompilación separando tu JavaScript y CSS en distintos archivos:

``` html
<!-- my-component.vue -->
<template>
  <div>This will be pre-compiled</div>
</template>
<script src="./my-component.js"></script>
<style src="./my-component.css"></style>
```

## Primeros pasos

### Para usuarios sin experiencia con sistemas de empaquetamiento de módulos en JavaScript

Con los componentes `.vue`, entramos en el ámbito de las aplicaciones avanzadas JavaScript. Esto significa aprender a usar algunas herramientas adicionales si todavía no lo has hecho:

- **Gestor de paquetes de Node (NPM por sus siglas en inglés)**: Lee la [guía de primeros pasos](https://docs.npmjs.com/getting-started/what-is-npm) hasta la sección _10: Desinstalando paquetes globales_.

- **JavaScript moderno con ES2015/16**: Lee la guia de Babel [para aprender ES2015](https://babeljs.io/docs/learn-es2015/). No tienes que memorizar cada nueva característica ahora mismo, pero mantén esa página como una refecencia a la que puedes consultar.

Luego de que te hayas tomado un día para profundizar esos recursos, te recomendamos dar un vistazo a la plantilla de proyecto [webpack-simple](https://github.com/vuejs-templates/webpack-simple). Sigue las instrucciones y ¡deberías tener un proyecto Vue con componentes `.vue`, ES2015 y recarga en caliente casi inmediatamente!

Las plantillas de proyecto utilizan [Webpack](https://webpack.js.org/), un sistema de empaquetamiento de módulos que recibe una cantidad de módulos X y los empaqueta en tu aplicación final. Para aprender más acerca de Webpack, [este video](https://www.youtube.com/watch?v=WQue1AN93YU) ofrece una buena introducción. Una vez que hayas entendido los conceptos básicos, puede que quieras mirar [este curso avanzado de Webpack en Egghead.io](https://egghead.io/courses/using-webpack-for-production-javascript-applications).

En Webpack, cada módulo puede ser transformado por un "cargador" antes de ser incluído en el paquete final, y Vue ofrece el complemento [vue-loader](https://github.com/vuejs/vue-loader) que se encarga de traducir los componentes de un solo archivo `.vue`. La plantilla de proyecto [webpack-simple](https://github.com/vuejs-templates/webpack-simple) ya tiene todo esto configurado para ti, pero si quieres aprender más acerca de como funcionan los componentes `.vue` en conjunto con Webpack, puedes leer [la documentación de vue-loader](https://vue-loader.vuejs.org).

### Para usuarios avanzados

Ya sea que prefieras Webpack o Browserify, hemos documentado las plantillas tanto para proyectos simples como complejos. Te recomendamos visitar [github.com/vuejs-templates](https://github.com/vuejs-templates), elegir la plantilla de proyecto que se ajuste a tu trabajo, y luego seguir las instrucciones en el README para generar un nuevo proyecto con [vue-cli](https://github.com/vuejs/vue-cli).
