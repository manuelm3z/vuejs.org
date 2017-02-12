---
title: Funciones de renderizado
type: guia
order: 15
---

## Fundamentos

Vue recomienda utilizar plantillas para generar tu HTML in la gran mayoría de los casos. Sin embargo, hay situaciones donde puedes necesitar todo el poder programático de JavaScript. Ahí es donde puedes utilizar **la función render**, una alternativa a las plantillas más parecida a un compilador.

Veamos un ejemplo sencillo donde la función `render` sería práctica. Digamos que quieres generar encabezados con anclas:

``` html
<h1>
  <a name="hello-world" href="#hello-world">
    Hello world!
  </a>
</h1>
```

Para el HTML anterior, decides que deseas la siguiente interface:

``` html
<anchored-heading :level="1">Hello world!</anchored-heading>
```

Cuando comienzas con un componente que simplemente genera un encabezado basado en la propiedad `level`, rápidamente llegas a lo siguiente:

``` html
<script type="text/x-template" id="anchored-heading-template">
  <div>
    <h1 v-if="level === 1">
      <slot></slot>
    </h1>
    <h2 v-if="level === 2">
      <slot></slot>
    </h2>
    <h3 v-if="level === 3">
      <slot></slot>
    </h3>
    <h4 v-if="level === 4">
      <slot></slot>
    </h4>
    <h5 v-if="level === 5">
      <slot></slot>
    </h5>
    <h6 v-if="level === 6">
      <slot></slot>
    </h6>
  </div>
</script>
```

``` js
Vue.component('anchored-heading', {
  template: '#anchored-heading-template',
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

Esa plantilla no parece estar bien. No solo es repetitiva, sino que estamos duplicando `<slot></slot>` en cada nivel de encabezado y tendremos que hacer lo mismo cuando agreguemos el elemento ancla. Además, hemos envuelto todo en un `div` inútil porque los componentes deben contener solo un nodo raíz.

A pesar que las plantillas funcionan muy bien para la mayoría de los componentes, es claro que este no es el caso. Intentemos reescribirlo con una función `render`:

``` js
Vue.component('anchored-heading', {
  render: function (createElement) {
    return createElement(
      'h' + this.level,   // nombre de la etiqueta
      this.$slots.default // arreglo de hijos
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

¡Mucho más simple! Casi. El código es más simple, pero require estar familiarizado con las propiedades de instancia de Vue. En este caso, debes saber que cuando pasas hijos sin un atributo `slot` a un componente, tales como el `Hello world!` dentro de `anchored-heading`, esos hijos son almacenados en la instancia del componente como `$slots.default`. Si todavía no lo hiciste, **es recomendable que leas la [API de las propiedades de instancia](../api/#vm-slots) antes que utilices las funciones de renderizado.**

## Parámetros de `createElement`

Lo segundo con lo que tendrás que familiarizarte es con como usar las características de las plantillas en la función `createElement`. Aquí están los parámetros que acepta `createElement`:

``` js
// @returns {VNode}
createElement(
  // {String | Object | Function}
  // Un nombre de etiqueta HTML, opciones de componente, o función
  // que devuelva uno de los dos anteriores. Requerido.
  'div',

  // {Object}
  // Un objeto de datos correspondiente a los atributos
  // que usarías en una plantilla. Opcional.
  {
    // (los detalles en la siguiente sección)
  },

  // {String | Array}
  // VNodes hijos. Opcional.
  [
    createElement('h1', 'hello world'),
    createElement(MyComponent, {
      props: {
        someProp: 'foo'
      }
    }),
    'bar'
  ]
)
```

### El objeto de datos en profundidad

Una cosa a tener en cuenta: similar a la manera en que `v-bind:class` y`v-bind:style` tienen un tratamiento especial en las plantillas, tienen sus propios campos de nivel superior en los objetos de datos VNode.

``` js
{
  // Misma API que `v-bind:class`
  'class': {
    foo: true,
    bar: false
  },
  // Misma API que `v-bind:style`
  style: {
    color: 'red',
    fontSize: '14px'
  },
  // Atributos HTML normales
  attrs: {
    id: 'foo'
  },
  // Propiedades del componente
  props: {
    myProp: 'bar'
  },
  // Propiedades del DOM
  domProps: {
    innerHTML: 'baz'
  },
  // Los controladores de eventos están anidadas bajo "on", sin 
  // embargo los modificadores como in v-on:keyup.enter no
  // están soportados. Tendrás que verificar manualmente
  // el código de la tecla en la función.
  on: {
    click: this.clickHandler
  },
  // Solo para componentes. Permite escuchar
  // eventos nativos en lugar de eventos emitidos desde
  // el componente utilizando vm.$emit.
  nativeOn: {
    click: this.nativeClickHandler
  },
  // Custom directives. Note that the binding's
  // oldValue cannot be set, as Vue keeps track
  // of it for you.
  directives: [
    {
      name: 'my-custom-directive',
      value: '2'
      expression: '1 + 1',
      arg: 'foo',
      modifiers: {
        bar: true
      }
    }
  ],
  // _Slots_ dentro del ámbito de la forma
  // { name: props => VNode | Array<VNode> }
  scopedSlots: {
    default: props => createElement('span', props.text)
  },
  // El nombre del _slot_, si este componente
  // es hijo de otro componente.
  slot: 'name-of-slot'
  // Otras propiedades especiales de alto nivel
  key: 'myKey',
  ref: 'myRef'
}
```

### Ejemplo completo

Con este conocimiento, podemos ahora terminar el componente que iniciamos:

``` js
var getChildrenTextContent = function (children) {
  return children.map(function (node) {
    return node.children
      ? getChildrenTextContent(node.children)
      : node.text
  }).join('')
}

Vue.component('anchored-heading', {
  render: function (createElement) {
    // create kebabCase id
    var headingId = getChildrenTextContent(this.$slots.default)
      .toLowerCase()
      .replace(/\W+/g, '-')
      .replace(/(^\-|\-$)/g, '')

    return createElement(
      'h' + this.level,
      [
        createElement('a', {
          attrs: {
            name: headingId,
            href: '#' + headingId
          }
        }, this.$slots.default)
      ]
    )
  },
  props: {
    level: {
      type: Number,
      required: true
    }
  }
})
```

### Restricciones

#### Los VNodes deben ser únicos

Todos los VNodes en un árbol de componente deben ser únicos. Eso significa que la siguiente función de renderizado es inválida:

``` js
render: function (createElement) {
  var myParagraphVNode = createElement('p', 'hi')
  return createElement('div', [
    // Aaagh - ¡VNodes duplicados!
    myParagraphVNode, myParagraphVNode
  ])
}
```

Si realmente deseas duplicar el mismo elemento/componente varias veces, puedes hacerlo con una función factoría. Por ejemplo, la siguiente función de renderizado es una forma perfectamente válida de renderizar 20 párrafos idénticos:

``` js
render: function (createElement) {
  return createElement('div',
    Array.apply(null, { length: 20 }).map(function () {
      return createElement('p', 'hi')
    })
  )
}
```

## Reemplazando características de plantillas con JavaScript puro

### `v-if` y `v-for`

Donde algo pueda ser solucionado con JavaScript puro, las funciones de renderizado de Vue no proveen una alternativa propietaria. Por ejemplo, en una plantilla utilizando `v-if` y `v-for`:

``` html
<ul v-if="items.length">
  <li v-for="item in items">{{ item.name }}</li>
</ul>
<p v-else>No items found.</p>
```

Esto puede ser reescrito con `if`/`else` y `map` de JavaScript en una función de renderizado:

``` js
render: function (createElement) {
  if (this.items.length) {
    return createElement('ul', this.items.map(function (item) {
      return createElement('li', item.name)
    }))
  } else {
    return createElement('p', 'No items found.')
  }
}
```

### `v-model`

No existe una contraparte directa de `v-model` en las funciones de renderizado - tendrás que implementar la lógica tú mismo:

``` js
render: function (createElement) {
  var self = this
  return createElement('input', {
    domProps: {
      value: self.value
    },
    on: {
      input: function (event) {
        self.value = event.target.value
      }
    }
  })
}
```

Este es el costo de trabajar a bajo nivel, pero también obtienes un mayor control sobre la interacción a comparación de `v-model`.

### Modificadores de eventos y teclas

Para los modificadores de eventos `.capture` y `.once`, Vue ofrece prefijos que pueden ser utilizados con `on`:

| Modificador(es) | Prefijo |
| ------ | ------ |
| `.capture` | `!` |
| `.once` | `~` |
| `.capture.once` o<br>`.once.capture` | `~!` |

Por ejemplo:

```javascript
on: {
  '!click': this.doThisInCapturingMode,
  '~keyup': this.doThisOnce,
  `~!mouseover`: this.doThisOnceInCapturingMode
}
```

Para todos los otros modificadores de eventos y teclas, no es necesario un prefijo propietario porque puedes simplemente utilizar los métodos de eventos en la función controladora:

| Modificador(es) | Equivalencia |
| ------ | ------ |
| `.stop` | `event.stopPropagation()` |
| `.prevent` | `event.preventDefault()` |
| `.self` | `if (event.target !== event.currentTarget) return` |
| Teclas:<br>`.enter`, `.13` | `if (event.keyCode !== 13) return` (cambia `13` por [otro código de tecla](http://keycode.info/) para otros modificadores) |
| Teclas modificadoras:<br>`.ctrl`, `.alt`, `.shift`, `.meta` | `if (!event.ctrlKey) return` (cambia `ctrlKey` por `altKey`, `shiftKey`, o `metaKey`, respectivamente) |

Aquí hay un ejemplo con todos estos modificadores utilizados en conjunto:

```javascript
on: {
  keyup: function (event) {
    // Retorna si el elemento emitiendo el evento no es
    // el elemento al cual está enlazado
    if (event.target !== event.currentTarget) return
    // Retorna si la tecla presionada no es _enter_
    // (13) y la tecla _shift_ no fue presionada 
    // al mismo tiempo
    if (!event.shiftKey || event.keyCode !== 13) return
    // Detiene la propagación del evento
    event.stopPropagation()
    // Previene la ejecución de la controladora por defecto
    // para este evento
    event.preventDefault()
    // ...
  }
}
```

### Slots

Puedes acceder al contenido de los _slots_ estáticos como un arreglo de VNodes en [`this.$slots`](http://vuejs.org/v2/api/#vm-slots):

``` js
render: function (createElement) {
  // <div><slot></slot></div>
  return createElement('div', this.$slots.default)
}
```

Y acceder a slots del ámbito como funciones que devuelven VNodes en [`this.$scopedSlots`](http://vuejs.org/v2/api/#vm-scopedSlots):

``` js
render: function (createElement) {
  // <div><slot :text="msg"></slot></div>
  return createElement('div', [
    this.$scopedSlots.default({
      text: this.msg
    })
  ])
}
```

Para pasar slots del ámbito a un componente hijo utilizando funciones de renderizado, agrega el campo `scopedSlots` en los datos del VNode:

``` js
render (createElement) {
  return createElement('div', [
    createElement('child', {
      // pasa scopedSlots en el objeto de datos
      // con la forma { name: props => VNode | Array<VNode> }
      scopedSlots: {
        default: function (props) {
          return createElement('span', props.text)
        }
      }
    })
  ])
}
```

## JSX

Si estás escribiendo muchas funciones `render`, puede parecer desagradable escribir algo como:

``` js
createElement(
  'anchored-heading', {
    props: {
      level: 1
    }
  }, [
    createElement('span', 'Hello'),
    ' world!'
  ]
)
```

Especialmente cuando la versión de plantillas es mucho más simple en comparación:

``` html
<anchored-heading :level="1">
  <span>Hello</span> world!
</anchored-heading>
```

Por eso es que existe un [complemento de Babel](https://github.com/vuejs/babel-plugin-transform-vue-jsx) para utilizar JSX con Vue, devolviéndonos a una sintaxis más parecida a las plantillas:

``` js
import AnchoredHeading from './AnchoredHeading.vue'

new Vue({
  el: '#demo',
  render (h) {
    return (
      <AnchoredHeading level={1}>
        <span>Hello</span> world!
      </AnchoredHeading>
    )
  }
})
```

<p class="tip">Darle el alias `h` a `createElement` es una convención común que verás en el ecosistema de Vue y que es obligatoria para JSX. Si `h` no está disponible en el ámbito, tu aplicación lanzará un error.</p>

Para más información acerca de como JSX se mapea a JavaScript, revisa [su documentación](https://github.com/vuejs/babel-plugin-transform-vue-jsx#usage).

## Componentes funcionales

El componente de encabezado con anclas que creamos anteriormente es relativamente sencillo. No maneja estado, no observa ninguna porción de estado que haya recibido y no tiene métodos de ciclo de vida. Es simplemente una función con algunas propiedades.

En casos como este, podemos marcar el componente como `funcional`, lo cual significa que carecen de estado (no tienen `data`) y de instancia (no tienen un contexto `this`). Un **componente funcional** luce así:

``` js
Vue.component('my-component', {
  functional: true,
  // Para compensar la falta de instancia,
  // se nos provee un segundo argumento de contexto
  render: function (createElement, context) {
    // ...
  },
  // Las propiedades son opcionales
  props: {
    // ...
  }
})
```

Todo lo que necesita el componente se pasa a través de `context`, el cual es un objeto que contiene:

- `props`: Un objeto con las propiedades provistas
- `children`: Un arreglo de VNode hijos
- `slots`: Una función que devuelve objetos _slot_
- `data`: El objeto _data_ entero pasado al componente
- `parent`: Una referencia al componente padre

Luego de agregar `functional: true`, actualizar la función de renderizado de nuestro componente de encabezado con anclas requeriría añadir el parámetro `context`, actualizar `this.$slots.default` a `context.children`, y luego `this.level` a `context.props.level`.

Dado que los componentes funcionales son funciones, son mucho menos costosas de renderizar. También son muy útiles como componentes envolventes. Por ejemplo, cuando necesitas:

- Escoger programáticamente uno de varios componentes para delegar
- Manipular hijos, propiedades o datos antes de pasarlos a un componente hijo

Aquí hay un ejemplo de un componente `smart-list` que delega a componentes más específicos, dependiendo de las propiedades que reciba:

``` js
var EmptyList = { /* ... */ }
var TableList = { /* ... */ }
var OrderedList = { /* ... */ }
var UnorderedList = { /* ... */ }

Vue.component('smart-list', {
  functional: true,
  render: function (createElement, context) {
    function appropriateListComponent () {
      var items = context.props.items

      if (items.length === 0)           return EmptyList
      if (typeof items[0] === 'object') return TableList
      if (context.props.isOrdered)      return OrderedList

      return UnorderedList
    }

    return createElement(
      appropriateListComponent(),
      context.data,
      context.children
    )
  },
  props: {
    items: {
      type: Array,
      required: true
    },
    isOrdered: Boolean
  }
})
```

### `slots()` vs `children`

Te debes estar preguntando por qué necesitamos tanto `slots()` como `children`. ¿No sería `slots().default` lo mismo que `children`? En algunos casos, si. Pero, ¿qué sucede si tienes un componente funcional con los siguientes hijos?

``` html
<my-functional-component>
  <p slot="foo">
    first
  </p>
  <p>second</p>
</my-functional-component>
```

Para este componente, `children` te dará ambos párrafos, `slots().default` solo te dará el segundo, y `slots().foo` solo el primero. Tener tanto `children` como `slots()` te permite elegir si este componente conoce el sistema de _slot_ o tal vez delegar esa responsabilidad en otro componente simplemente pasando `children`.

## Compilación de plantillas

Puede que estés interesado en saber que las plantillas de Vue son compiladas a funciones de renderizado. Este es un detalle de implementación que normalmente no necesitas conocer, pero si desearas ver como las características específicas de las plantillas son compiladas, puede resultarte interesante. Debajo hay una pequeña demostración utilizando `Vue.compile` para compilar en vivo una plantilla en forma de cadena de texto:

{% raw %}
<div id="vue-compile-demo" class="demo">
  <textarea v-model="templateText" rows="10"></textarea>
  <div v-if="typeof result === 'object'">
    <label>render:</label>
    <pre><code>{{ result.render }}</code></pre>
    <label>staticRenderFns:</label>
    <pre v-for="(fn, index) in result.staticRenderFns"><code>_m({{ index }}): {{ fn }}</code></pre>
  </div>
  <div v-else>
    <label>Compilation Error:</label>
    <pre><code>{{ result }}</code></pre>
  </div>
</div>
<script>
new Vue({
  el: '#vue-compile-demo',
  data: {
    templateText: '\
<div>\n\
  <h1>I\'m a template!</h1>\n\
  <p v-if="message">\n\
    {{ message }}\n\
  </p>\n\
  <p v-else>\n\
    No message.\n\
  </p>\n\
</div>\
    ',
  },
  computed: {
    result: function () {
      if (!this.templateText) {
        return 'Enter a valid template above'
      }
      try {
        var result = Vue.compile(this.templateText.replace(/\s{2,}/g, ''))
        return {
          render: this.formatFunction(result.render),
          staticRenderFns: result.staticRenderFns.map(this.formatFunction)
        }
      } catch (error) {
        return error.message
      }
    }
  },
  methods: {
    formatFunction: function (fn) {
      return fn.toString().replace(/(\{\n)(\S)/, '$1  $2')
    }
  }
})
console.error = function (error) {
  throw new Error(error)
}
</script>
<style>
#vue-compile-demo pre {
  padding: 10px;
  overflow-x: auto;
}
#vue-compile-demo code {
  white-space: pre;
  padding: 0
}
#vue-compile-demo textarea {
  width: 100%;

}
</style>
{% endraw %}
