---
title: Pruebas unitarias
type: guia
order: 23
---

## Configuración y herramientas

Cualquier cosa compatible con un sistema de empaquetamiento basado en módulos funcionará, pero si buscas recomendaciones específicas, prueba el ejecutor de pruebas [Karma](http://karma-runner.github.io). Tiene muchos complementos desarrollados por la comunidad, incluyendo soporte para [Webpack](https://github.com/webpack/karma-webpack) y [Browserify](https://github.com/Nikku/karma-browserify). Para una configuración detallada, por favor lee la documentación respectiva de cada proyecto, auqneu estas configuraciones de ejemplo de Karma para [Webpack](https://github.com/vuejs-templates/webpack/blob/master/template/test/unit/karma.conf.js) y [Browserify](https://github.com/vuejs-templates/browserify/blob/master/template/karma.conf.js) deberían ayudarte a empezar.

## Verificaciones simples

En términos de estructura de código para las pruebas, no tienes que hacer nada especial en tus componentes para poder probarlos. Simplemente exporta las opciones:

``` html
<template>
  <span>{{ message }}</span>
</template>

<script>
  export default {
    data () {
      return {
        message: 'hello!'
      }
    },
    created () {
      this.message = 'bye!'
    }
  }
</script>
```

Cuando pruebes ese componente, todo lo que tendrás que hacer es importar el objeto junto con Vue para intentar algunas verificaciones comunes:

``` js
// Importa Vue y el componente bajo pruebas
import Vue from 'vue'
import MyComponent from 'path/to/MyComponent.vue'

// Aquí hay alguas pruebas para Jasmine 2.0, aunque puedes
// utilizar cualquier biblioteca que prefieras
describe('MyComponent', () => {
  // Inspecciona las opciones base del componente
  it('has a created hook', () => {
    expect(typeof MyComponent.created).toBe('function')
  })

  // Evalúa los resultados de las funciones
  // en las opciones base del componente
  it('sets the correct default data', () => {
    expect(typeof MyComponent.data).toBe('function')
    const defaultData = MyComponent.data()
    expect(defaultData.message).toBe('hello!')
  })

  // Inspecciona la instancia del componente al montarla
  it('correctly sets the message when created', () => {
    const vm = new Vue(MyComponent).$mount()
    expect(vm.message).toBe('bye!')
  })

  // Monta la instancia e inspecciona el resultado del renderizado
  it('renders the correct message', () => {
    const Ctor = Vue.extend(MyComponent)
    const vm = new Ctor().$mount()
    expect(vm.$el.textContent).toBe('bye!')
  })
})
```

## Escribiendo componentes testeables

Muchas de las salidas del renderizado de componentes están determinadas por las propiedades que recibe. De hecho, si el resultado del renderizado de un componente depende únicamente de sus propiedades, es bastante sencillo testearlo, parecido a verificar el valor de retorno de una función pura con diferentes parámetros. Un ejemplo inventado:

``` html
<template>
  <p>{{ msg }}</p>
</template>

<script>
  export default {
    props: ['msg']
  }
</script>
```

Puedes verificar el resultado del renderizado con diferentes propiedades utilizando la opción `propsData`:

``` js
import Vue from 'vue'
import MyComponent from './MyComponent.vue'

// función auxiliar que monta y retorna el texto renderizado
function getRenderedText (Component, propsData) {
  const Ctor = Vue.extend(Component)
  const vm = new Ctor({ propsData }).$mount()
  return vm.$el.textContent
}

describe('MyComponent', () => {
  it('renders correctly with different props', () => {
    expect(getRenderedText(MyComponent, {
      msg: 'Hello'
    })).toBe('Hello')

    expect(getRenderedText(MyComponent, {
      msg: 'Bye'
    })).toBe('Bye')
  })
})
```

## Verificando actualizaciones asíncronas

Dado que Vue realiza [actualizaciones del DOM asíncronicamente](reactivity.html#Async-Update-Queue), las verificaciones sobre actualizaciones del DOM resultado de un cambio de estado tendrán que ser hechas en la _callback_ `Vue.nextTick`:

``` js
// Inspecciona el HTML generado luego de una actualización de estado
it('updates the rendered message when vm.message updates', done => {
  const vm = new Vue(MyComponent).$mount()
  vm.message = 'foo'

  // espera un "tick" luego del cambio del estado y antes de verificar actualizaciones del DOM
  Vue.nextTick(() => {
    expect(vm.$el.textContent).toBe('foo')
    done()
  })
})
```

Tenemos en vista trabajar en una colección de funciones auxiliares de pruebas comunes que hacen más simple renderizar componentes con diferentes restricciones (por ejemplo, renderizado superficial ignorando los componentes hijo) y verificar sus resultados.
