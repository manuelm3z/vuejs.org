---
title: Consejos para el despliegue a producción
type: guia
order: 20
---

## Habilita el modo producción

Durante el desarrollo, Vue provee gran cantidad de advertencias para ayudarte con errores comunes y dificultades. Sin embargo, estas advertencias se vuelven inútiles en producción e inflan el tamaño de tu aplicación final. Además, algunas de estas advertencias tienen un pequeño costo en terminos de rendimiento que pueden ser evitados en el modo producción.

### Sin herramientas de empaquetado

Si estás utilizando la versión independiente, por ejemplo, incluyendo Vue directamente a través de una etiqueta `script` sin herramientas de empaquetado, asegúrate de utilizar la versión minificada (`vue.min.js`) en producción.

### Con herramientas de empaquetado

Cuando utilices herramientas de empaquetado como Webpack o Browserify, el modo producción será determinado por `process.env.NODE_ENV` dentro del código fuente de Vue, y estará en modo desarrollo por defecto. Ambas herramientas proveen formas de sobrescribir esta variable para habilitar el modo producción de Vue, y las advertencias serán eliminadas por los minificadores durante el empaquetado. Todas las plantillas de proyectos de `vue-cli` tienen estas características ya configuradas, pero sería bueno que sepas como se hace:

#### Webpack

Usa [DefinePlugin](https://webpack.js.org/plugins/define-plugin/) de Webpack para indicar un entorno de producción, para que los bloques de advertencias puedan ser automáticamente descartados por UglifyJS durante la minificación. Configuración de ejemplo:

``` js
var webpack = require('webpack')

module.exports = {
  // ...
  plugins: [
    // ...
    new webpack.DefinePlugin({
      'process.env': {
        NODE_ENV: '"production"'
      }
    })
  ]
}
```

#### Browserify

- Ejecuta tu comando de empaquetamiento con la variable de entorno `NODE_ENV` con el valor `"production"`. Esto indica a `vueify` que evite incluir la recarga en caliente y el código relativo al desarrollo.

- Aplica una transformación global [envify](https://github.com/hughsk/envify) a tu paquete. Esto permite al minificador descartar todas las advertencias envueltas en bloques condicionales de la variable env dentro del código fuente de Vue. Por ejemplo:

  ``` bash
  NODE_ENV=production browserify -g envify -e main.js | uglifyjs -c -m > build.js
  ```

#### Rollup

Utiliza [rollup-plugin-replace](https://github.com/rollup/rollup-plugin-replace):

``` js
const replace = require('rollup-plugin-replace')

rollup({
  // ...
  plugins: [
    replace({
      'process.env.NODE_ENV': JSON.stringify( 'production' )
    })
  ]
}).then(...)
```

## Plantillas precompiladas

Cuando utilices plantillas dentro del DOM o plantillas como cadenas de texto en JavaScript, la compilación de _funciones de renderizado a plantillas_ es ejecutada sobre la marcha. Esto normalmente es lo suficientemente rapido en la mayoría de los casos, pero es mejor evitarlo si tu aplicación es sensible al rendimiento.

La manera mas fácil de precompilar plantillas es utilizando [componentes de un solo archivo](./single-file-components.html) - los procesos de empaquetamiento asociados automáticamente realizan la precompilacion por ti, por lo que el código resultante ya contiene las funciones de renderizado en lugar de las plantillas como cadenas de texto.

Si estas utilizando Webpack, y prefieres separar JavaScript y archivos de plantillas, puedes utilizar [vue-template-loader](https://github.com/ktsn/vue-template-loader), el cual también transforma los archivos de plantilla en funciones JavaScript de renderizado durante el proceso de empaquetado.

## Extrayendo CSS de los componentes

Cuando utilices componentes de un solo archivo, el CSS dentro de los mismos es inyectado dinámicamente como etiquetas `<style>` a través de JavaScript. Esto tiene un pequeño costo en términos de rendimiento, y si estás utilizando renderizado del lado servidor causará un "flash de contenido sin estilizar". Extrae el CSS de todos los componentes en un mismo archivo y evita estos problemas, además resulta en una mejor minificación y cacheo del CSS.

Verifica la documentación de las herramientas correspondientes para ver como se realiza:

- [Webpack + vue-loader](http://vue-loader.vuejs.org/en/configurations/extract-css.html) (la plantilla de proyecto webpack de `vue-cli` lo incluye preconfigurado)
- [Browserify + vueify](https://github.com/vuejs/vueify#css-extraction)
- [Rollup + rollup-plugin-vue](https://github.com/znck/rollup-plugin-vue#options)

## Registrando errores en tiempo de ejecución

Si un error en tiempo de ejecución ocurre durante el renderizado de un componente, será pasado a la función de configuración global `Vue.config.errorHandler` si ha sido agregada. Puede ser una buena idea apoyarse en esta herramienta en conjunto con algún servicio de registro de errores como [Sentry](https://sentry.io), el cual provee [integración oficial](https://sentry.io/for/vue/) para Vue.
