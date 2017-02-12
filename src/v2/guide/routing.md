---
title: Enrutamiento
type: guia
order: 21
---

## Enrutador oficial

Para la mayoría de las aplicaciones de una sola página, es recomendable utilizar la biblioteca con soporte oficial [vue-router library](https://github.com/vuejs/vue-router). Para más detalles, lee la [documentación](http://vuejs.github.io/vue-router/) de vue-router.

## Ruteo sencillo desde cero

Si solo necesitas enrutamient simple y no deseas utilizar una biblioteca de enrutamiento completa, puedes hacerlo renderizando dinámicamente un componente de página de la siguiente manera:

``` js
const NotFound = { template: '<p>Page not found</p>' }
const Home = { template: '<p>home page</p>' }
const About = { template: '<p>about page</p>' }

const routes = {
  '/': Home,
  '/about': About
}

new Vue({
  el: '#app',
  data: {
    currentRoute: window.location.pathname
  },
  computed: {
    ViewComponent () {
      return routes[this.currentRoute] || NotFound
    }
  },
  render (h) { return h(this.ViewComponent) }
})
```

En combinación con la API de historial de HTML5, puedes construir un enrutador de lado cliente muy básico completamente funcional. Para ver esto en práctica, revisa esta [aplicación de ejemplo](https://github.com/chrisvfritz/vue-2.0-simple-routing-example).

## Integrando enrutadores de terceros

Si hay algún enrutador de terceros que prefieras utilizar, como [Page.js](https://github.com/visionmedia/page.js) o [Director](https://github.com/flatiron/director), la integración es [igual de sencilla](https://github.com/chrisvfritz/vue-2.0-simple-routing-example/compare/master...pagejs). Aquí hay un [ejemplo completo](https://github.com/chrisvfritz/vue-2.0-simple-routing-example/tree/pagejs) utilizando Page.js.
