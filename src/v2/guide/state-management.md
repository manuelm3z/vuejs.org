---
title: Manejo de estado
type: guia
order: 22
---

## Implementación oficial estilo Flux

Las grandes aplicaciones pueden tornarse complejas, debido a múltiples porciones de estado dispersadas a través de muchos componentes y las interacciones entre ellos. Para resolver este problema, Vue ofrece [vuex](https://github.com/vuejs/vuex): nuestra biblioteca de manejo de estado propia inspirada en Elm. Incluso se integra con [vue-devtools](https://github.com/vuejs/vue-devtools), dando un acceso con configuración cero a los viajes en el tiempo.

### Información para desarrolladores de React

Si vienes de React, puede estar preguntandote como vuex se compara con [redux](https://github.com/reactjs/redux), la implementación Flux más popular en ese ecosistema. En realidad, Redux es independiente de la capa "Vista", por lo que puede ser utilizada con Vue a través de [algunos enlaces sencillos](https://github.com/egoist/revue). Vuex es diferente en el sentido que _sabe_ que se encuentra en una aplicación Vue. Esto permite una mejor integración, ofreciendo una API más intuitiva y una experiencia de desarrollo mejorada.

## Manejo de estado sencillo desde cero

Es normal pasar por alto que la fuente de verdad en las aplicaciones Vue es el objeto `data` - una instancia de Vue simplemente delega el acceso a él. Por lo tanto, si tienes una porción de estado que debería ser compartido por varias instancias, puedes compartirla por identidad:

``` js
const sourceOfTruth = {}

const vmA = new Vue({
  data: sourceOfTruth
})

const vmB = new Vue({
  data: sourceOfTruth
})
```

Ahora, cuando `sourceOfTruth` sea modificada, tanto `vmA` como `vmB` actualizarán sus vistas automáticamente. Los subcomponentes dentro de cada una de estas instancias también tendrán acceso a través de `this.$root.$data`. Ahora tenemos una sola fuente de verdad, pero depurar sería una pesadilla. Cualquier porción de los datos puede ser cambiada en cualquier lugar de nuestra aplicación en cualquier momento, sin dejar rastro.

Para ayudar a resolver este problema, podemos adoptar un sencillo **patrón store**:

``` js
var store = {
  debug: true,
  state: {
    message: 'Hello!'
  },
  setMessageAction (newValue) {
    this.debug && console.log('setMessageAction triggered with', newValue)
    this.state.message = newValue
  },
  clearMessageAction () {
    this.debug && console.log('clearMessageAction triggered')
    this.state.message = ''
  }
}
```

Nota que todas las acciones que modifican el estado del _store_ se encuentran dentro del mismo. Este tipo de manejo de estado centralizado hace más simple entender que tipo de mutaciones pueden ocurrir y como son disparadas. Ahora, cuando algo salga mal, también tendremos un registro de lo sucedido indicándonos donde está el error.

Además, cada instancia/componente puede manejar todavía su propio estado privado:

``` js
var vmA = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
})

var vmB = new Vue({
  data: {
    privateState: {},
    sharedState: store.state
  }
})
```

![State Management](/images/state.png)

<p class="tip">Es importante notar que nunca deberías reemplazar el objeto de estado original in tus acciones - los componentes y el _store_ necesitan compartir referencias al mismo objeto para que las mutaciones sean observadas.</p>

A medid que continuamos desarrollando la convención donde los componentes no tienen permitido modificar directamente el estado que pertenece a un _store_, sino que deberían emitir eventos que notifiquen al _store_ para que realice acciones, eventualmente llegamos a la arquitectura [Flux](https://facebook.github.io/flux/). El beneficio de esta convención es que podemos registrar todas las mutaciones de estado que ocurren en el _store_ e implementar características de depuración avanzada como registros de modificaciones, instantáneas, y recorrer hacia atrás/adelante el historial.

Esto nos pone nuevamente frente a [vuex](https://github.com/vuejs/vuex), por lo que si ya has leído hasta aquí, ¡probablemente es tiempo de probarlo!
