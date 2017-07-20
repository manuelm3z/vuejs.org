---
title: Instalación
type: guia
order: 1
vue_version: 2.1.3
dev_size: "206.14"
min_size: "67.60"
gz_size: "24.72"
ro_gz_size: "17.14"
---

### Notas de compatibilidad

Vue **no** está soportado en IE8 ni versiones anteriores, porque utiliza características de ECMAScript 5 que son irreproducibles en ellos. Sin embargo soporta todos los [navegadores compatibles con ECMAScript 5](http://caniuse.com/#feat=es5).

### Notas de lanzamiento

Se pueden encontrar notas de lanzamiento detalladas para cada versión en [GitHub](https://github.com/vuejs/vue/releases).

## Versión independiente `<script>`

Simplemente descargala e incluyela con una etiqueta script. `Vue` se registrará como una variable global.

<p class="tip">No utilices la versión minificada durante el desarrollo. ¡Perderás todas las advertencias para los errores comunes!</p>

<div id="downloads">
<a class="button" href="/js/vue.js" download>Versión de desarrollo</a><span class="light info">Con todas las advertencias y el modo depuración.</span>

<a class="button" href="/js/vue.min.js" download>Versión de producción</a><span class="light info">Advertencias eliminadas, {{gz_size}}kb min+gzip</span>
</div>

### CDN

Recomendación: [https://unpkg.com/vue](https://unpkg.com/vue/dist/vue.js), el cual contendrá la última versión apenas haya sido publicada en npm. También puedes explorar el código fuente del paquete npm en [https://unpkg.com/vue/](https://unpkg.com/vue/).

También se encuentra disponible en [jsDelivr](//cdn.jsdelivr.net/vue/{{vue_version}}/vue.js) o [cdnjs](//cdnjs.cloudflare.com/ajax/libs/vue/{{vue_version}}/vue.js), pero estos dos servicios pueden tardar algún tiempo en sincronizar, por lo que la última versión puede no estar disponible todavía.

## NPM

NPM es el método de instalación recomendado cuando se construyen aplicaciones a gran escala con Vue. Se integra bien con empaquetadores de módulos como [Webpack](http://webpack.github.io/) o [Browserify](http://browserify.org/). Vue también provee herramientas complementarias para la creación de [componentes de un solo archivo](single-file-components.html).

``` bash
# última versión estable
$ npm install vue
```
### Versión independiente vs. versión _Runtime-only_

Hay dos versiones disponibles, la independiente y la _runtime-only_. La diferencia es que la primera incluye un **compilador de plantillas** y la última no.

El compilador de plantillas es responsable de compilar plantillas en formato string de Vue a funciones de renderizacion en JavaScript puro. Si deseas utilizar la opción `template`, entonces necesitas el compilador.

- La versión independiente incluye el compilador y soporta la opción `template`. **También depende de la presencia de APIs del navegador, por lo que no puedes usarlo para renderizado del lado servidor.**

- La versión _runtime-only_ no incluye el compilador de plantillas y no soporta la opción `template`. Solo puedes utilizar la opción `render` cuando estás utilizando esta versión, pero funciona con componentes de un solo archivo, porque las plantillas de los componentes de un solo archivo son pre-compiladas en funciones `render` durante la etapa de construcción. La versión _runtime-only_ es aproximadamente 30% más liviana que la versión independiente, ocupando solo {{ro_gz_size}}kb min+gzip.

Por defecto, el paquete NPM exporta la versión **runtime-only**. Para utilizar la versión independiente, añade el siguiente alias en tu archivo de configuración de Webpack:

``` js
resolve: {
  alias: {
    'vue$': 'vue/dist/vue.common.js'
  }
}
```

Para Browserify, puedes añadir un alias a tu archivo package.json:

``` js
"browser": {
  "vue": "vue/dist/vue.common"
},
```

<p class="tip">No realices un `import Vue from 'vue/dist/vue.js'` - dado que algunas herramientas en bibliotecas de terceros pueden también importar vue y podría causar que la aplicación intente cargar ambas versiones al mismo tiempo, conduciendo a errores.</p>

### Ambientes CSP

Algunos ambientes, como las aplicaciones de Google Chrome, imponen las Políticas de Seguridad de Contenido (CSP por sus siglas en inglés), las cuales prohiben el uso de `new Function()` para la evaluación de expresiones. La versión independiente depende de esta característica para compilar plantillas, por lo que no es posible utilizarla en estos ambientes.

Por otro lado, la versión _runtime-only_ es completamente compatible con CSP. Cuando utilices la versión _runtime-only_ con [Webpack + vue-loader](https://github.com/vuejs-templates/webpack-simple) o [Browserify + vueify](https://github.com/vuejs-templates/browserify-simple), tus plantillas serán pre-compiladas en funciones `render` las cuales funcionan perfectamente en ambientes CSP.

## CLI

Vue.js provee una [CLI oficial](https://github.com/vuejs/vue-cli) para estructurar rápidamente Aplicaciones de una Sola Página (SPA por sus siglas en inglés). Provee configuraciones _todo-en-uno_ para un flujo de trabajo frontend moderno. Solo toma unos minutos estar preparado para el desarrollo con: recarga en caliente, _lint-on-save_ y versiones listas para producción:

``` bash
# Instala vue-cli
$ npm install --global vue-cli
# Crea un nuevo proyecto usando la plantilla "webpack"
$ vue init webpack my-project
# Instala las dependencias y ¡listo!
$ cd my-project
$ npm install
$ npm run dev
```

<p class="tip">La _CLI_ asume un conocimiento previo de Node.js y las herramientas de trabajo asociadas. Si eres principiante con Vue o las herramientas de desarrollo front-end, te recomendamos fervientemente leer <a href="./">la guía</a> sin ninguna herramienta de desarrollo previo a usar la _CLI_.</p>

## Versión desarrollo

**Importante**: los archivos construidos dentro de la carpeta `/dist` de GitHub son compiladas solo durante lanzamientos. Para utilizar el código fuente más reciente de Vue en GitHub, ¡tendrás que construirlo tú mismo!

``` bash
git clone https://github.com/vuejs/vue.git node_modules/vue
cd node_modules/vue
npm install
npm run build
```

## Bower

``` bash
# Última versión estable
$ bower install vue
```

## Gestores de módulos AMD

La versión independiente o las instaladas a través de Bower están encapsuladas con UMD, por lo que pueden ser usadas directamente como módulos AMD.
