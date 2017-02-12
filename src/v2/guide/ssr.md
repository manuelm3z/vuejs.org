---
title: Renderizado del lado servidor
type: guia
order: 24
---

## ¿Necesitas SSR?

Antes de introducirnos en el SSR, veamos que es lo que realmente hace por ti y cuando puede ser que lo necesites.

### SEO

Google y Bing pueden indexar correctamente aplicaciones JavaScript síncronas. _Síncronas_ siendo la palabra clave. Si tu aplicación comienza con un indicador de carga y luego obtiene información utilizando AJAX, el _crawler_ no esperará a que termines.

Esto significa que si tienes contenido recuperado asíncronicamente en páginas donde el SEO es importante, SSR puede ser necesario.

### Clientes con una conexión lenta a Internet

Puede que los usuarios naveguen a tu sitio desde un área remota con conexiones lentas a Internet - o simplemente con una mala conexión desde el celular. En estos casos, querrás minimizar el número y tamaño de las peticiones necesarias para que los usuarios puedan ver el contenido básico.

Puedes utilizar [el divisor de código de Webpack](https://webpack.js.org/guides/code-splitting-require/) para evitar forzar a los usuarios a descargar tu aplicación entera para ver una única página, pero aún no será igual de rápido que bajar un solo archivo HTML pre-renderizado.

### Clientes con un motor JavaScript antiguo (o sin soporte JavaScript)

En algunos lugares del mundo, utilizar una computadora del año 1998 para acceder a Internet puede ser la única opción. Mientras que Vue solo funciona con IE9 o superiors, tal vez quieras devolver algo de contenido básico a estos usuarios con navegadores antiguos - o a los hackers _hipsters_ que utilizan [Lynx](http://lynx.browser.org/) en una consola.

### SSR vs Pre-renderizado

Si solo estás invetigando SSR para mejorar el SEO de un puñado de páginas de marketing/presentación (por ejemplo. `/`, `/about`, `/contact`, etc), entonces lo que probablemente desees sea __pre-renderizado__. En lugar de utilizar un servidor web para compilar HTML en tiempo real, el pre-renderizado simplemente genera archivos HTML estáticos para rutas específicas en el momento de compilación. La ventaja es que configurar el pre-renderizado es mucho más sencillo y te permite mantener tu _frontend_ como un sitio completamente estático.

Si estás utilizando Webpack, puedes añadir pre-renderizado muy fácilmente con [prerender-spa-plugin](https://github.com/chrisvfritz/prerender-spa-plugin). Ha sido probado extensamente con aplicaciones Vue y, de hecho, el creador es un miembro del equipo de trabajo de Vue.

## Hola mundo

Si llegaste hasta aquí, estás listo para ver SSR en acción. Suena complejo, pero un _script_ node con esta característica requiere solo de 3 pasos:

``` js
// Paso 1: Crear una instancia de Vue
var Vue = require('vue')
var app = new Vue({
  render: function (h) {
    return h('p', 'hello world')
  }
})

// Paso 2: Crear un renderizador
var renderer = require('vue-server-renderer').createRenderer()

// Paso 3: Renderizar la instancia de Vue en el HTML
renderer.renderToString(app, function (error, html) {
  if (error) throw error
  console.log(html)
  // => <p server-rendered="true">hello world</p>
})
```

No es tan complicado, ¿verdad?. Por supuesto, este ejemplo es mucho más simple que la mayoría de las aplicaciones. No tenemos que preocuparnos todavía por:

- Un servidor web
- Respuestas como Streaming
- Cacheo de componentes
- Un proceso de compilación/empaquetamiento
- Enrutamiento
- Regeneración del estado de Vuex

En el resto de esta guia, repasaremos como trabajar con alguna de estas características. Una vez que entiendas los conceptos básicos, te indicaremos donde leer más documentación y ejemplos avanzados para ayudarte a manejar casos extremos.

## SSR sencillo con el servidor web Express

Es un poco difícil llarlo "renderizado del lado servidor" cuando en realidad no tenemos un servidor web, así que arreglemos eso. Vamos a construir una aplicación SSR muy sencilla utilizando solo ES5 sin ningún proceso de compilación ni complementos de Vue.

Empezaremos con una aplicación que le indica al usuario cuantos segundos ha estado en la página:

``` js
new Vue({
  template: '<div>You have been here for {{ counter }} seconds.</div>',
  data: {
    counter: 0
  },
  created: function () {
    var vm = this
    setInterval(function () {
      vm.counter += 1
    }, 1000)
  }
})
```

Para adaptar esto para SSR, hay algunas pocas modificaciones que tendremos que hacer, para que funcione tanto en los navegadores como con node:

- Cuando estemos en un navegador, añadir una instancia de nuestra aplicación al contexto global (es decir, `window`), para que podamos montarla.
- Cuando estemos en node, exportar una función factoría para que podamos crear una instancia nueva de la aplicación para cada petición.

Lograr esto requiere un poco más de código:

``` js
// assets/app.js
(function () { 'use strict'
  var createApp = function () {
    // ----------------------------
    // COMIENZA EL CÓDIGO DE LA APP
    // ----------------------------

    // La instancia principal debe ser devuelta y tener un 
    // nodo raíz con el id "app", para que la versión del lado 
    // cliente pueda tomar el control una vez que cargue.
    return new Vue({
      template: '<div id="app">You have been here for {{ counter }} seconds.</div>',
      data: {
        counter: 0
      },
      created: function () {
        var vm = this
        setInterval(function () {
          vm.counter += 1
        }, 1000)
      }
    })

    // ---------------------------
    // TERMINA EL CÓDIGO DE LA APP
    // ---------------------------
  }
  if (typeof module !== 'undefined' && module.exports) {
    module.exports = createApp
  } else {
    this.app = createApp()
  }
}).call(this)
```

Ahora que tenemos el código de nuestra aplicación, creemos un archivo `index.html`:

``` html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
  <title>My Vue App</title>
  <script src="/assets/vue.js"></script>
</head>
<body>
  <div id="app"></div>
  <script src="/assets/app.js"></script>
  <script>app.$mount('#app')</script>
</body>
</html>
```

Mientras que el directorio `assets` referenciado contenga el archivo `app.js` que creamos anteriormente junto con el archivo`vue.js` con Vue, ¡deberíamos tener una aplicación de una sola página funcionando!

Luego, para hacerla funcionar con el renderizado del lado servidor, hay un solo paso extra - el servidor web:

``` js
// server.js
'use strict'

var fs = require('fs')
var path = require('path')

// Define global.Vue para app.js funcionando del lado servidor
global.Vue = require('vue')

// Obtén la estructura HTML
var layout = fs.readFileSync('./index.html', 'utf8')

// Crea un renderizador
var renderer = require('vue-server-renderer').createRenderer()

// Crea un servidor express
var express = require('express')
var server = express()

// Sirve los archivos desde el directorio de recursos
server.use('/assets', express.static(
  path.resolve(__dirname, 'assets')
))

// Maneja las peticiones GET
server.get('*', function (request, response) {
  // Renderiza nuestra app Vue a una cadena de texto
  renderer.renderToString(
    // Crea una instancia de app
    require('./assets/app')(),
    // Maneja el resultado del renderizado
    function (error, html) {
      // Si ocurre un error durante el renderizado
      if (error) {
        // Registra el error en la consola
        console.error(error)
        // Avisa al cliente que algo sucedio
        return response
          .status(500)
          .send('Server Error')
      }
      // Envia la estructura con el HTML renderizado de la aplicación
      response.send(layout.replace('<div id="app"></div>', html))
    }
  )
})

// Escucha en el puerto 5000
server.listen(5000, function (error) {
  if (error) throw error
  console.log('Server is running at localhost:5000')
})
```

¡Y eso es todo! Aquí está [la aplicación completa](https://github.com/chrisvfritz/vue-ssr-demo-simple), en caso que quieras clonarla y experimentar un poco más. Una vez que la ejecutes localmente, puedes confirmar que el renderizado del lado servidor está realmente funcionando haciendo clic derecho en la página y seleccionando `View Page Source` (o similar). Deberías ver esto en el `body`:

``` html
<div id="app" server-rendered="true">You have been here for 0 seconds&period;</div>
```

en lugar de:

``` html
<div id="app"></div>
```

## Respuesta como Streaming

Vue tambén soporta renderizar a un __stream__, lo cual se prefiere para servidores web que soporten _streaming_. Esto permite que el HTML sea escrito a la respuesta _a medida que es generado_, en lugar de escribirlo todo junto al final. El resultado es que las respuestas son devueltas más rápido, ¡y no hay contras!

Para adaptar tu aplicación de la sección anterior y utilizar _streaming_, podemos simplemente reemplazar los bloques `server.get('*', ...)` con lo siguiente:

``` js
// Divide la estructura en dos secciones de HTML
var layoutSections = layout.split('<div id="app"></div>')
var preAppHTML = layoutSections[0]
var postAppHTML = layoutSections[1]

// Maneja todas las peticiones GET
server.get('*', function (request, response) {
  // Renderiza nuestra aplicación Vue a un _stream_
  var stream = renderer.renderToStream(require('./assets/app')())

  // Escribe el HTML preAppHtml en la respuesta
  response.write(preAppHTML)

  // Cada vez que nuevas porciones son renderizadas...
  stream.on('data', function (chunk) {
    // Escribe la porción en la respuesta
    response.write(chunk)
  })

  // Cuando todas las porciones son renderizadas...
  stream.on('end', function () {
    // Escribe el HTML postAppHTML en la respuesta
    response.end(postAppHTML)
  })

  // Si ocurre un error durante el renderizado...
  stream.on('error', function (error) {
    // Registra el error en la consola
    console.error(error)
    // Informa al cliente que algo sucedió
    return response
      .status(500)
      .send('Server Error')
  })
})
```
Como puedes ver, no es mucho más complicada que la versión anterior, incluso si los streams son conceptualmente nuevos para ti. Simplemente:

1. Incializamos el stream
2. Escribimos en la respuesta el HTML necesario previo a la aplicacion
3. Escribimos en la respuesta el HTML de la aplicación a medida que está disponible
4. Escribimos en la respuesta el HTML necesario posterior a la aplicacion y luego la finalizamos
5. Manejamos cualquier error que haya surgido

## Cacheo de componentes

El SSR de Vue es muy rápido por defecto, pero puedes mejorar aún más el rendimiento cacheando componentes renderizados. Sin embargo, esto debe considerarse una característica avanzada, dado que cachear los componentes equivocados (o los componentes correctos con la llave equivocada) puede conducir a errores en el renderizado de tu aplicación. Específicamente:

<p class="tip">No debes cachear un coponente que contiene componentes hijo que dependen de estado global (por ejemplo, desde un _store_ de vuex). Si lo haces, esos componentes hijo (y, de hecho, todo el sub-árbol) será también cacheado. Se especialmente cuidadoso con componentes que aceptan slots/hijos.</p>

### Configuración

Con esa advertencia fuera del camino, así es como cacheas tus componentes.

Primero, necesitarás asignar a tu renderizador un [objeto cache](https://www.npmjs.com/package/vue-server-renderer#cache). Aquí hay un ejemplo sencillo utilizando [lru-cache](https://github.com/isaacs/node-lru-cache):

``` js
var createRenderer = require('vue-server-renderer').createRenderer
var lru = require('lru-cache')

var renderer = createRenderer({
  cache: lru(1000)
})
```

Eso cacheará hasta 1000 renderizados únicos. Para otras configuraciones que se ajustan mejor al uso de memoria, verifica las [opciones de lru-cache](https://github.com/isaacs/node-lru-cache#options).

Luego, para componentes que desees cachear, debes asignarles:

- un `name` único
- una función `serverCacheKey`, devolviendo una llave única en el ámbito del componente

Por ejemplo:

``` js
Vue.component({
  name: 'list-item',
  template: '<li>{{ item.name }}</li>',
  props: ['item'],
  serverCacheKey: function (props) {
    return props.item.type + '::' + props.item.id
  }
})
```

### Componentes ideales para cacheo

Cualquier componente "puro" puede ser cacheado sin problemas - esto es, cualquier componente que puedas garantizar que renderizará el mismo HTML dadas las mismas propiedades. Ejemplos comunes de esto son:

- Componentes estáticos (i.e. siempre generan el mismo HTML, por lo que la función `serverCacheKey` puede directamente devolver `true`)
- Componentes elemento de una lista (cuando se trabaja con listas grandes, cachear estos puede mejorar el rendimiento significativamente)
- Componentes de UI genéricos (e.g. botones, alertas, etc - al menos aquellos que aceptan contenido a travéd e propiedades en lugar de slots/hijos)

## Proceso de compilación, ruteo y regeneración de estado de Vuex

En este punto, deberías entender los conceptos fundamentales detrás del renderizado del lado servidor. Sin embargo, a medida que agregas un proceso de compilación/empaquetamiento, ruteo y vuex, cada uno introduce sus propias consideraciones.

Para dominar realmente el renderizado del lado servidor en aplicaciones complejas, te recomendamos leer detenidamente los siguientes enlaces:

- [Documentación de vue-server-renderer](https://www.npmjs.com/package/vue-server-renderer#api): mayor detalle en los temas tratados aquí, así como documentación de temas más avanzados, como [prevenir contaminación de peticiones cruzadas](https://www.npmjs.com/package/vue-server-renderer#why-use-bundlerenderer) y [añadir un build de servidor separado](https://www.npmjs.com/package/vue-server-renderer#creating-the-server-bundle)
- [vue-hackernews-2.0](https://github.com/vuejs/vue-hackernews-2.0): el ejemplo definitivo de integración de todas las bibliotecas principales y conceptos de Vue en una sola aplicación.

## Nuxt.js

Configurar apropiadamente todos los aspectos discutidos de una aplicación renderizada del lado servidor preparada para producción puede ser una tarea intimidante. Por suerte, existe un excelente proyecto de la comunidad que apunta a hacer todo esto mucho más sencillo: [Nuxt.js](https://nuxtjs.org/). Nuxt.js es un framework de nivel superior construído sobre el ecosistema de Vue, el cual provee una experiencia de desarrollo extremadamente eficiente para escribir aplicaciones Vue universales. Aún mejor, puedes usarlo como un generador de sitios estático (con las páginas creadas como componentes de un solo archivo de Vue). Recomendamos fervientemente que lo pruebes.
