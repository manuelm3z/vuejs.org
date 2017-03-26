---
type: api
---

## Configuración global

`Vue.config` es un objeto que contiene las configuraciones globales de Vue. Puedes modificar las propiedades listadas debajo antes de iniciar tu aplicación:

### silent

- **Tipo:** `boolean`

- **Valor por defecto:** `false`

- **Uso:**

  ``` js
  Vue.config.silent = true
  ```

  Elimina todos los registros y advertencias de Vue.

### optionMergeStrategies

- **Tipo:** `{ [key: string]: Function }`

- **Valor por defecto:** `{}`

- **Uso:**

  ``` js
  Vue.config.optionMergeStrategies._my_option = function (parent, child, vm) {
    return child + 1
  }

  const Profile = Vue.extend({
    _my_option: 1
  })

  // Profile.options._my_option = 2
  ```

  Define estrategias de fusionado personalizadas para las opciones.

  La estrategia de fusión recibe el valor de la opción definida en las instancias padre e hijo como primer y segundo argumento respectivamente. La instancia Vue de contexto es pasada como tercer argumento.

- **Lee también:** [Estrategias personalizadas de fusionado de opciones](../guide/mixins.html#Custom-Option-Merge-Strategies)

### devtools

- **Tipo:** `boolean`

- **Valor por defecto:** `true` (`false` en versiones de producción)

- **Uso:**

  ``` js
  // asegúrate de establecer síncronicamente lo siguiente inmediatamente después de cargar Vue
  Vue.config.devtools = true
  ```

  Permite o no la inspección de [vue-devtools](https://github.com/vuejs/vue-devtools). EL valor por defecto de esta opción es `true` en las versiones de desarrollo y `false` en versiones de producción. Puedes establecer este valor en `true` para habilitar la inspección en versiones de producción.

### errorHandler

- **Tipo:** `Function`

- **Valor por defecto:** El error es lanzado en el lugar

- **Uso:**

  ``` js
  Vue.config.errorHandler = function (err, vm) {
    // maneja el error
  }
  ```

  Asigna una función controladora para los errores no capturados durante renderización de componentes y observadores. La función controladora es llamada con el error y la instancia de Vue.

  > [Sentry](https://sentry.io), un servicio de seguimiento de errores, provee [integración oficial](https://sentry.io/for/vue/) utilizando esta opción.

### ignoredElements

- **Tipo:** `Array<string>`

- **Valor por defecto:** `[]`

- **Uso:**

  ``` js
  Vue.config.ignoredElements = [
    'my-custom-web-component', 'another-web-component'
  ]
  ```

  Hace que Vue ignore los elementos personalizados definidos fuera de si mismo (por ejemplo, utilizando la API de componentes Web). De otra forma, lanzará una advertencia `Unknown custom element`, asumiendo que has olvidado registrar un componente global o escrito mal un nombre de componente.

### keyCodes

- **Tipo:** `{ [key: string]: number | Array<number> }`

- **Valor por defecto:** `{}`

- **Uso:**

  ``` js
  Vue.config.keyCodes = {
    v: 86,
    f1: 112,
    mediaPlayPause: 179,
    up: [38, 87]
  }
  ```

  Define alias personalizados de teclas para v-on.

## API Global

<h3 id="Vue-extend">Vue.extend( options )</h3>

- **Argumentos:**
  - `{Object} options`

- **Uso:**

  Crea una "subclase" del constructor base de Vue. El argumento debe ser un objeto que contenga las opciones de componente.

  El caso especial a tener en cuenta aquí es la opción `data` - debe ser una función cuando se utiliza con `Vue.extend()`.

  ``` html
  <div id="mount-point"></div>
  ```

  ``` js
  // crea el constructor
  var Profile = Vue.extend({
    template: '<p>{{firstName}} {{lastName}} aka {{alias}}</p>',
    data: function () {
      return {
        firstName: 'Walter',
        lastName: 'White',
        alias: 'Heisenberg'
      }
    }
  })
  // crea una instancia de 'Profile' y la monta en un elemento
  new Profile().$mount('#mount-point')
  ```

  Se obtiene como resultado:

  ``` html
  <p>Walter White aka Heisenberg</p>
  ```

- **Lee también:** [Componentes](../guide/components.html)

<h3 id="Vue-nextTick">Vue.nextTick( [callback, context] )</h3>

- **Argumentos:**
  - `{Function} [callback]`
  - `{Object} [context]`

- **Uso:**
  
  Posterga la llamada a la función _callback_ hasta el próximo ciclo de actualización del DOM. Utilízala inmediatamente luego de que hayas cambiado datos para esperar a la siguiente actualización del DOM.

  ``` js
  // modifica los datos
  vm.msg = 'Hello'
  // el DOM no fue actualizado todavía
  Vue.nextTick(function () {
    // DOM actualizado
  })
  ```

  > Nuevo en 2.1.0: devuelve una _Promise_ si no se provee una función _callback_ y el ambiente de ejecución las soporta.

- **Lee también:** [Coloa de actualización asíncrona](../guide/reactivity.html#Async-Update-Queue)

<h3 id="Vue-set">Vue.set( object, key, value )</h3>

- **Argumentos:**
  - `{Object} object`
  - `{string} key`
  - `{any} value`

- **Devuelve:** el valor establecido.

- **Uso:**

  Establece una propiedad en un objeto. Si el objeto es reactivo, asegura que la propiedad es creada como una propiedad reactiva y dispara actualizaciones de la vista. Es utilizado principalmente para solucionar la limitacion de Vue de no poder detectar el agregado de propiedades.

  **Nota que el objeto no puede ser una instancia de Vue, ni el objeto de datos raíz de una instancia de Vue.**

- **Lee también:** [Reactividad en profundidad](../guide/reactivity.html)

<h3 id="Vue-delete">Vue.delete( object, key )</h3>

- **Argumentos:**
  - `{Object} object`
  - `{string} key`

- **Uso:**

  Borra una propiedad de un objeto. Si el objeto es reactivo, asegura el borrado de los disparadores de actualizaciones de vistas. Es utilizado principalmente para solucionar la limitacion de Vue de no poder detectar el quitado de propiedades, pero no debería ser normal que necesites utilizarlo.

  **Nota que el objeto no puede ser una instancia de Vue, ni el objeto de datos raíz de una instancia de Vue.**

- **Lee también:** [Reactividad en profundidad](../guide/reactivity.html)

<h3 id="Vue-directive">Vue.directive( id, [definition] )</h3>

- **Argumentos:**
  - `{string} id`
  - `{Function | Object} [definition]`

- **Uso:**

  Registra o recupera una directiva global.

  ``` js
  // registra
  Vue.directive('my-directive', {
    bind: function () {},
    inserted: function () {},
    update: function () {},
    componentUpdated: function () {},
    unbind: function () {}
  })

  // registra (función directiva sencilla)
  Vue.directive('my-directive', function () {
    // esto será ejecutado como `bind` y `update`
  })

  // 'getter', devuelve la definición de la directiva si ha sido registrada
  var myDirective = Vue.directive('my-directive')
  ```

- **Lee también:** [Directivas personalizadas](../guide/custom-directive.html)

<h3 id="Vue-filter">Vue.filter( id, [definition] )</h3>

- **Argumentos:**
  - `{string} id`
  - `{Function} [definition]`

- **Uso:**

  Registra o recupera un filtro global.

  ``` js
  // registra
  Vue.filter('my-filter', function (value) {
    // devuelve el valor procesado
  })

  // 'getter', devuelve el filtro si ha sido registrado
  var myFilter = Vue.filter('my-filter')
  ```

<h3 id="Vue-component">Vue.component( id, [definition] )</h3>

- **Argumentos:**
  - `{string} id`
  - `{Function | Object} [definition]`

- **Uso:**

  Registra o recupera un componente global. El registro establece automáticamente la propiedad `name` del componente como el `id` dado.

  ``` js
  // registra un constructor extendido
  Vue.component('my-component', Vue.extend({ /* ... */ }))

  // registra un objeto de opciones (automáticamente llama a Vue.extend)
  Vue.component('my-component', { /* ... */ })

  // recupera un componente registrado (siempre devuelve un constructor)
  var MyComponent = Vue.component('my-component')
  ```

- **Lee también:** [Componentes](../guide/components.html)

<h3 id="Vue-use">Vue.use( plugin )</h3>

- **Argumentos:**
  - `{Object | Function} plugin`

- **Uso:**

  Instala un complemento de Vue.js. Si el complemento es un objeto, debe exponer un método `install`. Si es una función, será tratada como el método de instalación. Este método será llamado con Vue como argumento.

  Cuando este método es ejecutado con el mismo complemento varias veces, el complemento será instalado solo una vez.

- **Lee también:** [Complementos](../guide/plugins.html)

<h3 id="Vue-mixin">Vue.mixin( mixin )</h3>

- **Argumentos:**
  - `{Object} mixin`

- **Uso:**

  Aplica un _mixin_ globalmente, el cual afecta a cada instancia de Vue creada luego de la llamada. Puede ser utilizado por creadores de complementos para inyectar comportamiento personalizado dentro de los componentes.
  **No se recomienda en el código de la aplicación**.

- **Lee también:** [Mixins globales](../guide/mixins.html#Global-Mixin)

<h3 id="Vue-compile">Vue.compile( template )</h3>

- **Argumentos:**
  - `{string} template`

- **Uso:**

  Compila una plantilla de texto en una función de renderizado. **Solo disponible en la versión independiente.**

  ``` js
  var res = Vue.compile('<div><span>{{ msg }}</span></div>')

  new Vue({
    data: {
      msg: 'hello'
    },
    render: res.render,
    staticRenderFns: res.staticRenderFns
  })
  ```

- **Lee también:** [Funciones de renderizado](../guide/render-function.html)

## Opciones / Datos

### data

- **Tipo:** `Object | Function`

- **Restricciones:** Solo acepta `Function` cuando se utiliza en la definición de componentes.

- **Detalles:**

  El objeto de datos para la instancia de Vue. Vue convertirá recursivamente sus propiedades en _getters/setters_ para hacerlos "reactivos". **Debe ser un objeto plano**: los objetos nativos tales como los objetos de API de los navegadores y propiedades de prototipo son ignorados. Una regla sencilla es que los datos deben ser solo datos - no es recomendable observar objetos con comportamiento propio.

  Una vez observado, no puedes añadir más propiedades reactivas al objeto de datos raíz. Por lo tanto, es recomendable declarar todas las propiedades reactivas en el nivel raíz por adelantado, antes de crear la instancia.

  Luego que la instancia ha sido creada, el objeto original de dato puede ser accedido a través de `vm.$data`. La instancia de Vue también espeja todas las propiedades encontradas en el objeto de datos, por lo que `vm.a` será equivalente a `vm.$data.a`.

  Las propiedades que comienzan con `_` o `$` **no** serán espejadas en la instancia de Vue porque pueden entrar en conflicto con las propiedades internas y métodos de la API de Vue. Tendrás que acceder a ellos como `vm.$data._property`.

  Cuando se define un **componente**, `data` debe ser declarado como una función que devuelve el objeto de datos inicial, porque habrá varias instancias creadas utilizando la misma definición. Si utilizamos un objeto plano para `data`, ¡ese mismo objeto será **compartido por referencia** en todas las instancias creadas! Creando una función `data`, cada vez que una nueva instancia es creada, podemos simplemente ejecutarla para devolver una nueva copia de los datos originales.

  Si se requiere, se puede obtener una copia profunda del objeto original pasando `vm.$data` a `JSON.parse(JSON.stringify(...))`.

- **Ejemplo:**

  ``` js
  var data = { a: 1 }

  // creación directa de instancia
  var vm = new Vue({
    data: data
  })
  vm.a // -> 1
  vm.$data === data // -> true

  // debe utilizarse una función cuando se trabaja con Vue.extend()
  var Component = Vue.extend({
    data: function () {
      return { a: 1 }
    }
  })
  ```

  <p class="tip">Nota que __no debes utilizar una función flecha con la propiedad `data`__ (por ejemplo: `data: () => { return { a: this.myProp }}`). La razón es que las funciones flecha enlazan el contexto padre, por lo que `this` no será la instancia de Vue como esperarías y `this.myProp` tendrá un valor `undefined`.</p>

- **Lee también:** [Reactividad en profundidad](../guide/reactivity.html)

### props

- **Tipo:** `Array<string> | Object`

- **Detalles:**

  Una lista/hash de atributos expuestos para aceptar datos desde el componente padre. Tiene una sintaxis sencilla basada en vectores y otra alternativa basada en objetos que permite configuraciones avanzadas como verificación de tipos, validación personalizada y valores por defecto.

- **Ejemplo:**

  ``` js
  // sintaxis simple
  Vue.component('props-demo-simple', {
    props: ['size', 'myMessage']
  })

  // sintaxis de objeto con validación
  Vue.component('props-demo-advanced', {
    props: {
      // solo verificación de tipo
      height: Number,
      // verificación de tipo junto con otras validaciones
      age: {
        type: Number,
        default: 0,
        required: true,
        validator: function (value) {
          return value >= 0
        }
      }
    }
  })
  ```

- **Lee también:** [Propiedades](../guide/components.html#Props)

### propsData

- **Tipo:** `{ [key: string]: any }`

- **Restricciones:** solo respetadaa en la creación de una instancia utilizando `new`.

- **Detalles:**

  Pasa propiedades a una instancia durante su creación. Está pensado principalmente para facilitar las pruebas unitarias.

- **Ejemplo:**

  ``` js
  var Comp = Vue.extend({
    props: ['msg'],
    template: '<div>{{ msg }}</div>'
  })

  var vm = new Comp({
    propsData: {
      msg: 'hello'
    }
  })
  ```

### computed

- **Tipo:** `{ [key: string]: Function | { get: Function, set: Function } }`

- **Detalles:**

  Las propiedades computadas a ser fusionadas en la instancia de Vue. Todos los _getters_ y _setters_ tienen su contexto `this` automáticamente enlazado con la instancia de Vue.

  <p class="tip">Nota que __no deberías utilizar una función flecha para definir una propiedad computada__ (por ejemplo: `aDouble: () => this.a * 2`). La razón es que las funciones flecha enlazan el contexto padre, por lo que `this` no será la instancia de Vue como esperarías y `this.a` tendrá un valor `undefined`.</p>

  Las propiedades computadas son guardadas en memoria caché, y solo son re-evaluadas cuando las dependencias cambian. Nota que si cierta dependencia está fuera del ámbito de la instancia (i.e. no es reactiva), la propiedad computada __no__ será actualizada.

- **Ejemplo:**

  ```js
  var vm = new Vue({
    data: { a: 1 },
    computed: {
      // si solo necesitas obtener datos, se pasa una única función
      aDouble: function () {
        return this.a * 2
      },
      // tanto obtener como establecer datos
      aPlus: {
        get: function () {
          return this.a + 1
        },
        set: function (v) {
          this.a = v - 1
        }
      }
    }
  })
  vm.aPlus   // -> 2
  vm.aPlus = 3
  vm.a       // -> 2
  vm.aDouble // -> 4
  ```

- **Lee también:**
  - [Propiedades computadas](../guide/computed.html)

### methods

- **Tipo:** `{ [key: string]: Function }`

- **Detalles:**

  Los métodos a ser fusionados en la instancia de Vue. Puedes acceder a estos métodos directamente en la instancia de la VM, o usarlos en expresiones de directivas. Todos los métodos tendrán su contexto `this` enlazado con la instancia de Vue.

  <p class="tip">Nota que __no deberías utilizar una función flecha para definir un método__ (por ejemplo: `plus: () => this.a++`). La razón es que las funciones flecha enlazan el contexto padre, por lo que `this` no será la instancia de Vue como esperarías y `this.a` tendrá un valor `undefined`.</p>

- **Ejemplo:**

  ```js
  var vm = new Vue({
    data: { a: 1 },
    methods: {
      plus: function () {
        this.a++
      }
    }
  })
  vm.plus()
  vm.a // 2
  ```

- **Lee también:** [Métodos y Manejo de eventos](../guide/events.html)

### watch

- **Tipo:** `{ [key: string]: string | Function | Object }`

- **Detalles:**

  Un objeto donde las llaves son expresiones a observar y los valores son sus correspondientes funciones _callback_. El valor puede ser también una cadena de texto equivalente al nombre de un método, o un objeto que contenga opciones adicionales. La instancia de Vue ejecutará `$watch()` por cada entrada del objeto al momento de la creación de la instancia.

- **Ejemplo:**

  ``` js
  var vm = new Vue({
    data: {
      a: 1,
      b: 2,
      c: 3
    },
    watch: {
      a: function (val, oldVal) {
        console.log('new: %s, old: %s', val, oldVal)
      },
      // cadena de texto con el nombre del método
      b: 'someMethod',
      // deep watcher
      c: {
        handler: function (val, oldVal) { /* ... */ },
        deep: true
      }
    }
  })
  vm.a = 2 // -> nuevo: 2, viejo: 1
  ```

  <p class="tip">Nota que __no deberías utilizar una función flecha para definir un observador__ (por ejemplo: `searchQuery: newValue => this.updateAutocomplete(newValue)`). La razón es que las funciones flecha enlazan el contexto padre, por lo que `this` no será la instancia de Vue como esperarías y `this.updateAutocomplete` tendrá un valor `undefined`.</p>


- **Lee también:** [Métodos de instancia - vm.$watch](#vm-watch)

## Opciones / DOM

### el

- **Tipo:** `string | HTMLElement`

- **Restricciones:** solo respetado en la creación de instancias a través de `new`.

- **Detalles:**

  Provee a la instancia de Vue un elemento del DOM existente sobre el que montarse. Puede ser un selector CSS o un elemento HTMLElement.

  Luego que la instancia haya sido montada, el elemento resuelto será accesible como `vm.$el`.

  Si esta opción está disponible al momento de la instanciación, la instancia entrará inmediatamente en la compilación; en otro caso, el usuario tendrá que llamar explícitamente a `vm.$mount()` para iniciar manualmente la compilación.

  <p class="tip">El elemento proviste sirve únicamente como punto de montaje. A diferencia de Vue 1.x, el elemento montado será reemplazado con DOM generado por Vue en todos los casos. Por lo tanto no es recomendable montar la instancia principal en `<html>` o `<body>`.</p>

- **Lee también:** [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)

### template

- **Tipo:** `string`

- **Detalles:**
  
  Plantilla de texto a ser usada como estructura para la instancia de Vue. La plantilla **reemplazará** al elemento montado. Cualquier estructura existente dentro del elemento montado será ignorada, a menos que se encuentren _slots_ de distribución de contenido en la plantilla.

  Si la cadena de texto comienza con `#` será utilizada como _querySelector_ y se usará el contenido interno HTML del elemento seleccionado como plantilla de texto. Esto permite la utilización del conocido truco `<script type="x-template">` para incluir plantillas.

  <p class="tip">Desde una perspectiva de seguridad, solo deberías utilizar plantillas de Vue en las que confíes. Nunca utilices contenido generado por el usuario como tu plantilla.</p>

- **Lee también:**
  - [Diagrama de ciclo de vida](../guide/instance.html#Lifecycle-Diagram)
  - [Distribución de contenido](../guide/components.html#Content-Distribution-with-Slots)

### render

  - **Tipo:** `Function`

  - **Detalles:**

    Una alternativa a las plantillas de texto que te permite apoyarte en el poder de JavaScript. La función de renderizado recibe un método `createElement` como su primer argumento el cual se utiliza para crear un `VNode`.

    Si el componente es uno funcional, la función de renderizado también recibe el argumento extra `context`, el cual provee acceso a los datos contextuales dado que los componentes funcionales no son instanciables.

  - **Lee también:**
    - [Funciones de renderizado](../guide/render-function)

## Opciones / Hooks de ciclo de vida

Todos los _hooks_ de ciclo de vida tienen su contexto `this` enlazado a la instancia, para que puedas acceder a los datos, propiedades computadas y métodos. Esto significa que __no deberías utilizar una función flecha para definir un método del ciclo de vida__ (por ejemplo: `created: () => this.fetchTodos()`). La razón es que las funciones flecha enlazan el contexto padre, por lo que `this` no será la instancia de Vue como esperarías y `this.fetchTodos` tendrá un valor `undefined`.

### beforeCreate

- **Tipo:** `Function`

- **Detalles:**

  Llamada asíncronicamente apenas la instancia ha sido inicializada, antes que la observación de datos y la configuración de eventos/observadores.

- **Lee también:** [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)

### created

- **Tipo:** `Function`

- **Detalles:**

  Llamada sincrónicamente luego de que la instancia ha sido creada. En este momento, la instancia ha terminado de procesar las opciones lo cual significa que ya han sido configuradas: la observación de datos, propiedades computadas, métodos, funciones _callback_ para eventos/observadores. Sin embargo, la fase de montaje no ha sido iniciada, y la propiedad `$el` no estára disponible todavía.

- **Lee también:** [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)

### beforeMount

- **Tipo:** `Function`

- **Detalles:**

  Llamada apenas comienza el montaje: la función `render` está por ser llamada por primera vez.

  **Este _hook_ no se llama durante el renderizado del lado servidor.**

- **Lee también:** [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)

### mounted

- **Tipo:** `Function`

- **Detalles:**

  Llamada apenas ha sido montada la instancia en el lugar de `el` reemplazado con el recientemente creado `vm.$el`. Si la instancia principal es montada en un elemento dentro del documento, `vm.$el` estará también en el documento cuando `mounted` es llamada.

  **Este _hook_ no se llama durante el renderizado del lado servidor.**

- **Lee también:** [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)

### beforeUpdate

- **Tipo:** `Function`

- **Detalles:**

  Llamada cuando los datos cambian, antes que el DOM virtual sea re-renderizado y actualizado.

  Puedes realizar otros cambios de estado en este _hook_ y no dispararán re-renderizaciones adicionales.

  **Este _hook_ no se llama durante el renderizado del lado servidor.**

- **Lee también:** [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)

### updated

- **Tipo:** `Function`

- **Detalles:**

  Llamada luego que un cambio en los datos hayan generado que el DOM virtual haya sido re-renderizado y actualizado.

  El DOM del componente ya habrá sido actualizado cuando este _hook_ es llamado, por lo que puedes realizar operaciones dependientes del DOM aquí. Sin embargo, en la mayoría de los casos deberías evitar cambiar el estado dentro de este _hook_. Para reaccionar ante cambios de estado, normalmente es mejor utilizar [propiedades computadas](#computed) u [observadores](#watch) en su lugar.

  **Este _hook_ no se llama durante el renderizado del lado servidor.**

- **Lee también:** [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)

### activated

- **Tipo:** `Function`

- **Detalles:**

  Llamada cuando un componente `keep-alive` es activado.

  **Este _hook_ no se llama durante el renderizado del lado servidor.**

- **Lee también:**
  - [Componentes incorporados - keep-alive](#keep-alive)
  - [Componentes dinámicos - keep-alive](../guide/components.html#keep-alive)

### deactivated

- **Tipo:** `Function`

- **Detalles:**

  Llamada cuando un componente `keep-alive` es desactivado.

  **Este _hook_ no se llama durante el renderizado del lado servidor.**

- **Lee también:**
  - [Componentes incorporados - keep-alive](#keep-alive)
  - [Componentes dinámicos - keep-alive](../guide/components.html#keep-alive)

### beforeDestroy

- **Tipo:** `Function`

- **Detalles:**

  Llamada apenas antes que una instancia Vue es destruida. En este momento la instancia todavía es completamente funcional.

  **Este _hook_ no se llama durante el renderizado del lado servidor.**

- **Lee también:** [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)

### destroyed

- **Tipo:** `Function`

- **Detalles:**

  Llamada luego que una instancia de Vue ha sido destruida. Cuando este _hook_ es llamado, todas la directivas de la instancia de Vue han sido desenlazadas, todos los _listeners_ de eventos han sido quitados y todas las intancias Vue hijas también han sido destruidas.

  **Este _hook_ no se llama durante el renderizado del lado servidor.**

- **Lee también:** [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)

## Opciones / Recursos

### directives

- **Tipo:** `Object`

- **Detalles:**

  Un hash de directivas para hacer disponibles a la instancia de Vue.

- **Lee también:**
  - [Directivas personalizadas](../guide/custom-directive.html)
  - [Convención de nombres de recursos](../guide/components.html#Assets-Naming-Convention)

### filters

- **Tipo:** `Object`

- **Detalles:**

  Un hash de filtros para hacer disponibles a la instancia de Vue.

- **Lee también:**
  - [`Vue.filter`](#Vue-filter)

### components

- **Tipo:** `Object`

- **Detalles:**

  A hash of componentes to be made available to the Vue instance.

- **Lee también:**
  - [Components](../guide/components.html)

## Opciones / Misc

### parent

- **Tipo:** `Vue instance`

- **Detalles:**

  Especifica la instancia padre de la instancia a ser creada. Establece una relación padre-hijo entre las dos. El padre será accesible a través de `this.$parent` para el hijo, y el hijo será agregado al vector `$children` del padre.

  <p class="tip">Utiliza `$parent` y `$children` con mesura - sirven principalmente como una vía de escape. Es preferible utilizar propiedades y eventos para la comunicación padre-hijo.</p>

### mixins

- **Tipo:** `Array<Object>`

- **Detalles:**

  La opción `mixins` acepta un vector de objetos _mixin_. Estos objetos pueden contener opciones de instancia como cualquier objeto de instancia normal, y pueden ser fusionados con las opciones finales utilizando la misma lógica encontrada en `Vue.extend()`. Si tu _mixin_ contiene un _hook created_ y el componente también, ambas funciones serán ejecutadas.

  Los _hooks de mixin_ son llamados en el orden que son provistos y luego de los propios del componente.

- **Ejemplo:**

  ``` js
  var mixin = {
    created: function () { console.log(1) }
  }
  var vm = new Vue({
    created: function () { console.log(2) },
    mixins: [mixin]
  })
  // -> 1
  // -> 2
  ```

- **Lee también:** [Mixins](../guide/mixins.html)

### name

- **Tipo:** `string`

- **Restricciones:** solo respetado cuando se utiliza como opción de componente.

- **Detalles:**

  Permite al componente invocarse recursivamente en su plantilla. Nota que cuando un componente es registrado globalemente con `Vue.component()`, el ID global es automáticamente establecido como su nombre.

  Otro beneficio de especificar la opción `name` es la depuración. Los componentes con nombre permiten entregar mensajes de advertencia de más ayuda. También, cuando se inspecciona una aplicación con [vue-devtools](https://github.com/vuejs/vue-devtools), los componentes sin nombre aparecerán como `<AnonymousComponent>`, lo cual no es muy informativo. Al proveer la opción `name`, obtendrás un árbol de componentes mucho más descriptivo.

### extends

- **Tipo:** `Object | Function`

- **Detalles:**

  Permite extender declarativamente otro componente (puede ser tanto un objeto de componentes como un constructor) sin tener que utilizar `Vue.extend`. Esto está pensado principalmente para hacer más sencillo el extender componentes de un solo archivo.

  Es similar a `mixins`, siendo la diferencia que las opciones propias de los componentes tienen mayor prioridad que el componente base que está siendo extendido.

- **Ejemplo:**

  ``` js
  var CompA = { ... }

  // extiende CompA sin tener que llamar a Vue.extend
  var CompB = {
    extends: CompA,
    ...
  }
  ```

### delimiters

- **Tipo:** `Array<string>`

- **Valor por defecto:** `{% raw %}["{{", "}}"]{% endraw %}`

- **Detalles:**

  Cambia los demilitadores de interpolación de texto plano. **Esta opción solo está disponible en la versión independiente.**

- **Ejemplo:**

  ``` js
  new Vue({
    delimiters: ['${', '}']
  })

  // Delimitadores cambiados al estilo ES6
  ```

### functional

- **Tipo:** `boolean`

- **Detalles:**

  Genera que un componente no tenga estado (sin `data`) ni instancia (sin contexto `this`). Son simplemente funciones `render` que devuelven nodos virtuales haciéndolos mucho menos costosos de renderizar.

- **Lee también:** [Componentes funcionales](../guide/render-function.html#Functional-Components)

## Propiedades de instancia

### vm.$data

- **Tipo:** `Object`

- **Detalles:**

  El objecto de datos que la instancia de Vue está observando. La instancia de Vue delega acceso a las propiedades en su objeto de datos.

- **Lee también:** [Opciones - data](#data)

### vm.$el

- **Tipo:** `HTMLElement`

- **Solo lectura**

- **Detalles:**

  El elemento raíz del DOM que la instancia de Vue está controlando.

### vm.$options

- **Tipo:** `Object`

- **Solo lectura**

- **Detalles:**

  Las opciones de instanciación usadas para la instancia actual de Vue. Esto es útil cuando quieres incluir propiedades personalizadas en las opciones:

  ``` js
  new Vue({
    customOption: 'foo',
    created: function () {
      console.log(this.$options.customOption) // -> 'foo'
    }
  })
  ```

### vm.$parent

- **Tipo:** `Vue instance`

- **Solo lectura**

- **Detalles:**

  La instancia padre, si la instancia actual tiene una.

### vm.$root

- **Tipo:** `Vue instance`

- **Solo lectura**

- **Detalles:**

  La instancia raíz del árbol del componente actual. Si la instancia no tiene padres, este valor la retornará a ella.

### vm.$children

- **Tipo:** `Array<Vue instance>`

- **Solo lectura**

- **Detalles:**

  Los componentes hijos directos de la instancia actual. **Nota que no se garantiza el orden de `$children`, y no es reactivo.** Si te encuentras intentando utilizar `$children` para enlazar datos, considera la opción de un arreglo y `v-for` para generar componentes hijos, y utilizar el arreglo como fuente de verdad.

### vm.$slots

- **Tipo:** `{ [name: string]: ?Array<VNode> }`

- **Solo lectura**

- **Detalles:**

  Utilizado para acceder programáticamente a contenido [distribuído en slots](../guide/components.html#Content-Distribution-with-Slots). Cada [slot con nombre](../guide/components.html#Named-Slots) tiene su propiedad correspondiente (por ejemplo, el contenido de `slot="foo"` se encontrará en `vm.$slots.foo`). La propiedad `default` contiene cualquier nodo no incluido en un slot con nombre.

  Acceder a `vm.$slots` is particularmente útil cuando se escriben componentes con una [función de renderizado](../guide/render-function.html).

- **Ejemplo:**

  ```html
  <blog-post>
    <h1 slot="header">
      About Me
    </h1>

    <p>Here's some page content, which will be included in vm.$slots.default, because it's not inside a named slot.</p>

    <p slot="footer">
      Copyright 2016 Evan You
    </p>

    <p>If I have some content down here, it will also be included in vm.$slots.default.</p>.
  </blog-post>
  ```

  ```js
  Vue.component('blog-post', {
    render: function (createElement) {
      var header = this.$slots.header
      var body   = this.$slots.default
      var footer = this.$slots.footer
      return createElement('div', [
        createElement('header', header),
        createElement('main', body),
        createElement('footer', footer)
      ])
    }
  })
  ```

- **Lee también:**
  - [Componente `<slot>`](#slot-1)
  - [Distribución de contenido con slots](../guide/components.html#Content-Distribution-with-Slots)
  - [Funciones de renderizado: Slots](../guide/render-function.html#Slots)

### vm.$scopedSlots

> Nuevo en 2.1.0

- **Tipo:** `{ [name: string]: props => VNode | Array<VNode> }`

- **Solo lectura**

- **Detalles:**

  Utilizado para acceder programáticamente a [slots con ámbito](../guide/components.html#Scoped-Slots). Para cada slot, incluido el `default`, el objeto contiene la función correspondiente que devuelve VNodes.

  Acceder a `vm.$slots` is particularmente útil cuando se escriben componentes con una [función de renderizado](../guide/render-function.html).

- **Lee también:**
  - [Componente `<slot>`](#slot-1)
  - [Distribución de contenido con slots](../guide/components.html#Content-Distribution-with-Slots)
  - [Funciones de renderizado: Slots](../guide/render-function.html#Slots)

### vm.$refs

- **Tipo:** `Object`

- **Solo lectura**

- **Detalles:**

  Un objeto que contiene componentes hijos registrados con `ref`.

- **Lee también:**
  - [Referencia a componentes hijos](../guide/components.html#Child-Component-Refs)
  - [ref](#ref)

### vm.$isServer

- **Tipo:** `boolean`

- **Solo lectura**

- **Detalles:**

  Indica si la instancia actual de Vue está corriendo en el servidor.

- **Lee también:** [Renderizado lado servidor](../guide/ssr.html)

## Métodos de instancia / Datos

<h3 id="vm-watch">vm.$watch( expOrFn, callback, [options] )</h3>

- **Argumentos:**
  - `{string | Function} expOrFn`
  - `{Function} callback`
  - `{Object} [options]`
    - `{boolean} deep`
    - `{boolean} immediate`

- **Devuelve:** `{Function} unwatch`

- **Uso:**

  Observa cambios en una expresión o función computada en la instancia de Vue. La función callback es llamada con el nuevo valor y el viejo valor como parámetros. La expresión solo acepta rutas simples delimitadas con puntos. Para expresiones más complejas, utiliza una función.

<p class="tip">Nota: cuando muta (en lugar de reemplazar) un objeto o un arreglo, el viejo valor será igual al nuevo valor debido a que referencian al mismo objeto/arreglo. Vue no mantiene una copia del valor previo a la mutación.</p>

- **Ejemplo:**

  ``` js
  // keypath
  vm.$watch('a.b.c', function (newVal, oldVal) {
    // haz algo
  })

  // Función
  vm.$watch(
    function () {
      return this.a + this.b
    },
    function (newVal, oldVal) {
      // haz algo
    }
  )
  ```

  `vm.$watch` devuelve una funcíon `unwatch` que frena la ejecución de la función callback.

  ``` js
  var unwatch = vm.$watch('a', cb)
  // luego, destruye el observador
  unwatch()
  ```

- **Opción: deep**

  Para detectar también cambios en valores anidados dentro de objetos, necesitas pasar `deep: true` en las opciones. Nota que no necesitas hacerlo para observar mutaciones en arreglos.

  ``` js
  vm.$watch('someObject', callback, {
    deep: true
  })
  vm.someObject.nestedValue = 123
  // la función callback es ejecutada
  ```

- **Opción: immediate**

  Pasar `immediate: true` en las opciones disparará la función callback inmediatamente con el valor actual de la expresión:

  ``` js
  vm.$watch('a', callback, {
    immediate: true
  })
  // la función callback es ejecutada inmediatamente con el valor actual de `a`
  ```

<h3 id="vm-set">vm.$set( object, key, value )</h3>

- **Argumentos:**
  - `{Object} object`
  - `{string} key`
  - `{any} value`

- **Devuelve:** the set value.

- **Uso:**

  Un **alias** del global `Vue.set`.

- **Lee también:** [Vue.set](#Vue-set)

<h3 id="vm-delete">vm.$delete( object, key )</h3>

- **Argumentos:**
  - `{Object} object`
  - `{string} key`

- **Uso:**

  Un **alias** del global `Vue.delete`.

- **Lee también:** [Vue.delete](#Vue-delete)

## Métodos de instancia / Eventos

<h3 id="vm-on">vm.$on( event, callback )</h3>

- **Argumentos:**
  - `{string} event`
  - `{Function} callback`

- **Uso:**

  Escucha eventos personalizados en la vm actual. Los eventos pueden ser disparados utilizando `vm.$emit`. La función callback recibirá todos los argumentos adicionales pasados a estos métodos disparadores de eventos.

- **Ejemplo:**

  ``` js
  vm.$on('test', function (msg) {
    console.log(msg)
  })
  vm.$emit('test', 'hi')
  // -> "hi"
  ```

<h3 id="vm-once">vm.$once( event, callback )</h3>

- **Argumentos:**
  - `{string} event`
  - `{Function} callback`

- **Uso:**

  Escucha eventos personalizados, pero solo una vez. La función listener será eliminada una vez que se ejecute por primera vez.

<h3 id="vm-off">vm.$off( [event, callback] )</h3>

- **Argumentos:**
  - `{string} [event]`
  - `{Function} [callback]`

- **Uso:**

  Elimina funciones listener de eventos.

  - Si no se proveen argumentos, elimina todas las funciones.

  - Si solo se provee el evento, elimina todas las funciones listener para ese evento.

  - Si se proveen tanto un evento como una función callback, se elimina la función listener para esa función callback específica.

<h3 id="vm-emit">vm.$emit( event, [...args] )</h3>

- **Argumentos:**
  - `{string} event`
  - `[...args]`

  Dispara un evento en la instancia actual. Cualquier argumento adiciona será pasado a la función listener.

## Métodos de instancia / Ciclo de vida

<h3 id="vm-mount">vm.$mount( [elementOrSelector] )</h3>

- **Argumentos:**
  - `{Element | string} [elementOrSelector]`
  - `{boolean} [hydrating]`

- **Devuelve:** `vm` - la instancia misma

- **Uso:**

  Si la instancia de Vue no recibio la opción `el` cuando fue instanciada, estará en estado "unmounted", sin un elemento DOM asociado. `vm.$mount()` puede utilizarse para iniciar manualmente el montaje de una instancia Vue sin montar.

  Si no se provee el argumento `elementOrSelector`, la plantilla será renderizada como un elemento fuera del documento, y tendrás que utilizar la API del DOM nativa para insertarlo tú mismo en el documento.

  El método devuelve la instancia para que puedas encadenar otros métodos de instancia luego.

- **Ejemplo:**

  ``` js
  var MyComponent = Vue.extend({
    template: '<div>Hello!</div>'
  })

  // Crea y monta en #app (reemplazará #app)
  new MyComponent().$mount('#app')

  // Lo anterior es igual a:
  new MyComponent({ el: '#app' })

  // o renderiza fuera del documento y agrega el componente luego:
  var component = new MyComponent().$mount()
  document.getElementById('app').appendChild(component.$el)
  ```

- **Lee también:**
  - [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)
  - [Renderizado lado servidor](../guide/ssr.html)

<h3 id="vm-forceUpdate">vm.$forceUpdate()</h3>

- **Uso:**

  Fuerza a la instancia de Vue a re-renderizarse. Nota que no afecta a todos los componentes hijo, solo a la instancia misma y los componentes hijo con contenido insertado en slots.slot content.

<h3 id="vm-nextTick">vm.$nextTick( [callback] )</h3>

- **Argumentos:**
  - `{Function} [callback]`

- **Uso:**

  Posterta la ejecución de la función callback luego del próximo ciclo de actualización del DOM. Utilízala inmediatamente después de modificar algún dato para esperar a la próxima actualización del DOM. Es igual al global `Vue.nextTick`, excepto que el contexto `this` de la función callback será enlazada automáticamente a la instancia que llame a este método.

  > Nuevo en 2.1.0: devuelve una Promise si no se provee una función callback y Promise es soportada en el ambiente de ejecución.

- **Ejemplo:**

  ``` js
  new Vue({
    // ...
    methods: {
      // ...
      example: function () {
        // Modifica data
        this.message = 'changed'
        // El DOM no ha sido actualizado todavía
        this.$nextTick(function () {
          // DOM está actualizado
          // `this` está enlazado a la instancia actual
          this.doSomethingElse()
        })
      }
    }
  })
  ```

- **Lee también:**
  - [Vue.nextTick](#Vue-nextTick)
  - [Cola de actualizaciones asíncronas](../guide/reactivity.html#Async-Update-Queue)

<h3 id="vm-destroy">vm.$destroy()</h3>

- **Uso:**

  Destruye completamente una vm. Elimina todas las conexiones con otras vm existentes, desenlaza todas sus directivas y borra todas las funciones listener de eventos.

  Dispara los hooks `beforeDestroy` y `destroyed`.

  <p class="tip">En casos de uso normales no deberías tener que llamar a este método. Es preferible controlar el ciclo de vida de los componentes hijos utilizando `v-if` y `v-for`.</p>

- **Lee también:** [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)

## Directivas

### v-text

- **Espera:** `string`

- **Detalles:**

  Actualiza el `textContent` del elemento. Si necesitas actualizar la porción de `textContent`, deberías utilizar interpolaciones `{% raw %}{{ Mustache }}{% endraw %}`.

- **Ejemplo:**

  ```html
  <span v-text="msg"></span>
  <!-- lo mismo que -->
  <span>{{msg}}</span>
  ```

- **Lee también:** [Sintaxis de enlace de datos - interpolaciones](../guide/syntax.html#Text)

### v-html

- **Espera:** `string`

- **Detalles:**

  Actualiza el `innerHTML` del elemento. **Nota que el contenido es insertado como HTML plano - no será compilado como plantilla de Vue**. Si te encuentras intentando componer plantillas utilizando `v-html`, intenta repensar la solución utilizando componentes en su lugar.

  <p class="tip">Renderizar dinámicamente HTML arbitrario en tu sitio web puede ser muy peligroso porque puede conducir a [ataques XSS](https://en.wikipedia.org/wiki/Cross-site_scripting). Utilizar `v-html` solo con contenido de confianza y **nunca** con contenido provisto por el usuario.</p>

- **Ejemplo:**

  ```html
  <div v-html="html"></div>
  ```
- **Lee también:** [Sintaxis de enlace de datos - interpolaciones](../guide/syntax.html#Raw-HTML)

### v-show

- **Espera:** `any`

- **Uso:**

  Alterna el valor de la propiedad CSS `display` del elemento, basado en la veracidad del valor de la expresión.

  Esta directiva dispara transiciones cuando su condición cambia.

- **Lee también:** [Renderizado condicional - v-show](../guide/conditional.html#v-show)

### v-if

- **Espera:** `any`

- **Uso:**

  Renderiza condicionalmente el elemento basado en la veracidad del valor de la expresión. El elemento y los componentes / directivas que contiene son destruidos y reconstruidos cuando cambia el valor de verdad. Si un elemento es un `<template>`, su contenido será extraído como bloque condicional.

  Esta directiva dispara transiciones cuando su condición cambia.

  <p class="tip">Cuando se utiliza en conjunto con v-if, v-for tiene mayor prioridad. Lee la <a href="../guide/list.html#v-for-with-v-if">guía de renderizado de listas</a> para más detalles.</p>

- **Lee también:** [Renderizado condicional - v-if](../guide/conditional.html)

### v-else

- **No espera expresión**

- **Restricciones:** el elemento hermano anterior debe tener `v-if` o `v-else-if`.

- **Uso:**

  Representa el "bloque else" para una cadena `v-if` o `v-if`/`v-else-if`.

  ```html
  <div v-if="Math.random() > 0.5">
    Now you see me
  </div>
  <div v-else>
    Now you don't
  </div>
  ```

- **Lee también:**
  - [Renderizado condicional - v-else](../guide/conditional.html#v-else)

### v-else-if

> Nuevo en 2.1.0

- **Espera:** `any`

- **Restricciones:** el elemento hermano anterior debe tener `v-if` o `v-else-if`.

- **Uso:**

  Representa el "bloque else if" para `v-if`. Puede encadenarse.

  ```html
  <div v-if="type === 'A'">
    A
  </div>
  <div v-else-if="type === 'B'">
    B
  </div>
  <div v-else-if="type === 'C'">
    C
  </div>
  <div v-else>
    Not A/B/C
  </div>
  ```

- **Lee también:** [Renderizado condicional - v-else-if](../guide/conditional.html#v-else-if)

### v-for

- **Espera:** `Array | Object | number | string`

- **Uso:**

  Renderiza el elemento o bloque de plantillas múltiples veces basado en los datos de origen. El valor de la directiva debe utilizar la sintaxis especial `alias in expression` para proveer un alias para el elemento actual sobre el que se está iterando:

  ``` html
  <div v-for="item in items">
    {{ item.text }}
  </div>
  ```

  Alternativamente, puedes especificar un alias para el índice (o la llave si estás trabajando con un objeto):

  ``` html
  <div v-for="(item, index) in items"></div>
  <div v-for="(val, key) in object"></div>
  <div v-for="(val, key, index) in object"></div>
  ```

  Por defecto `v-for` intentará parchear los elementos en su lugar sin moverlos. Para forzar el reordenamiento de los elementos, necesitas proveer una "pista" de ordenamiento con el atributo especial `key`:

  ``` html
  <div v-for="item in items" :key="item.id">
    {{ item.text }}
  </div>
  ```

  <p class="tip">Cuando se utiliza en conjunto con v-if, v-for tiene mayor prioridad. Lee la <a href="../guide/list.html#v-for-with-v-if">guía de renderizado de listas</a> para más detalles.</p>

  El uso detallado de `v-for` se explica en la sección de la guía enlazada debajo.

- **Lee también:**
  - [Renderizado de listas](../guide/list.html)
  - [key](../guide/list.html#key)

### v-on

- **Atajo:** `@`

- **Espera:** `Function | Inline Statement`

- **Argumento:** `event (requerido)`

- **Modificadores:**
  - `.stop` - llama a `event.stopPropagation()`.
  - `.prevent` - llama a `event.preventDefault()`.
  - `.capture` - agrega la función listener en modo captura.
  - `.self` - solo dispara la función controladora si el evento fue emitido desde este elemento.
  - `.{keyCode | keyAlias}` - solo dispara la función controladora para ciertas teclas.
  - `.native` - escucha un evento nativo en el elemento raíz del componente.
  - `.once` - dispara la función controladora una vez como máximo.

- **Uso:**

  Añade una función listener al elemento. El tupo de evento es especificado por el argumento. La expresión puede ser tanto el nombre de un método como una declaración en línea, o simplemente ser omitida cuando hay modificadores presentes.

  Cuando se utiliza con un elemento normal, escucha solo **eventos nativos del DOM**. Cuando se utiliza en un componente personalizado, tambien escucha **eventos personalizados** emitidos en ese componente hijo.

  Cuando se escuchan eventos nativos del DOM, el método recibe el evento nativo como su único argumento. Si se utiliza una declaración en línea, esta tiene acceso a la propiedad especial `$event`: `v-on:click="handle('ok', $event)"`.

- **Ejemplo:**

  ```html
  <!-- método controlador -->
  <button v-on:click="doThis"></button>

  <!-- declaración en línea -->
  <button v-on:click="doThat('hello', $event)"></button>

  <!-- Atajo -->
  <button @click="doThis"></button>

  <!-- detiene la propagación -->
  <button @click.stop="doThis"></button>

  <!-- previene el comportamiento por defecto -->
  <button @click.prevent="doThis"></button>

  <!-- previene el comportamiento por defecto sin expresión -->
  <form @submit.prevent></form>

  <!-- modificadores en cadena -->
  <button @click.stop.prevent="doThis"></button>

  <!-- modificadores de teclas utilizando keyAlias -->
  <input @keyup.enter="onEnter">

  <!-- modificadores de teclas utilizando keyCode -->
  <input @keyup.13="onEnter">

  <!-- el evento click será disparado como máximo una vez -->
  <button v-on:click.once="doThis"></button>
  ```

  Escuchar eventos personalizados en un componente hijo (la función controladora es ejecutada cuando "my-event" es emitido en el hijo):

  ```html
  <my-component @my-event="handleThis"></my-component>

  <!-- declaración en línea -->
  <my-component @my-event="handleThis(123, $event)"></my-component>

  <!-- evento nativo en el componente -->
  <my-component @click.native="onClick"></my-component>
  ```

- **Lee también:**
  - [Métodos y manejo de eventos](../guide/events.html)
  - [Componentes - Eventos personalizados](../guide/components.html#Custom-Events)

### v-bind

- **Atajo:** `:`

- **Espera:** `any (with argument) | Object (sin argumentos)`

- **Argumento:** `attrOrProp (opcional)`

- **Modificadores:**
  - `.prop` - Enlaza como una propiedad DOM en lugar de un atributo. ([¿Cuál es la diferencia?](http://stackoverflow.com/questions/6003819/properties-and-attributes-in-html#answer-6004028))
  - `.camel` - transforma el nombre del atributo de kebab-case a camelCase. (soportado desde 2.1.0)

- **Uso:**

  Enlaza dinámicamente uno o más atributos o una propiedad de un componente a una expresión.

  Cuando se utiliza para enlazar los atributos `class` o `style`, soporta tipos de valores adicionales, como arreglos u objetos. Lee la sección de la guía enlazada abajo para más detalles.

  Cuando se utiliza para enlazar propiedades, la propiedad debe estar declarada en el componente hijo.

  Cuando se utiliza sin un argumento, puede ser usada para enlazar un objeto que contiene pares nombre-valor de atributos. Nota que en este modo, `class` y `style` no soportan arreglos u objetos.

- **Ejemplo:**

  ```html
  <!-- enlaza un atributo -->
  <img v-bind:src="imageSrc">

  <!-- Atajo -->
  <img :src="imageSrc">

  <!-- con contatenación de texto en línea -->
  <img :src="'/path/to/images/' + fileName">

  <!-- enlace de clases -->
  <div :class="{ red: isRed }"></div>
  <div :class="[classA, classB]"></div>
  <div :class="[classA, { classB: isB, classC: isC }]">

  <!-- enlace de estilos -->
  <div :style="{ fontSize: size + 'px' }"></div>
  <div :style="[styleObjectA, styleObjectB]"></div>

  <!-- enlazando un objeto de atributos -->
  <div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>

  <!-- enlace de atributo DOM con el modificador prop -->
  <div v-bind:text-content.prop="text"></div>

  <!-- enlace de propiedades. "prop" debe ser declarada en my-component. -->
  <my-component :prop="someThing"></my-component>

  <!-- XLink -->
  <svg><a :xlink:special="foo"></a></svg>
  ```

  El modificador `.camel` permite "camelizar" el nombre de un atributo `v-bind` cuando se utiliza en plantillas dentro del DOM. Por ejemplo, el atributo SVG `viewBox`:

  ``` html
  <svg :view-box.camel="viewBox"></svg>
  ```

  `.camel` no es necesario si estás utilizando plantillas de texto, o compilando con `vue-loader`/`vueify`.

- **Lee también:**
  - [Enlace de clases y estilos](../guide/class-and-style.html)
  - [Componentes - Propiedades de componentes](../guide/components.html#Props)

### v-model

- **Espera:** varía basado en el valor del elemento del formulario o la salida de componentes

- **Limitado a :**
  - `<input>`
  - `<select>`
  - `<textarea>`
  - componentes

- **Modificadores:**
  - [`.lazy`](../guide/forms.html#lazy) - escucha el evento `change` en lugar de `input`
  - [`.number`](../guide/forms.html#number) - castea el valor de los input a número
  - [`.trim`](../guide/forms.html#trim) - aplica 'trim' al valor del input

- **Uso:**

  Crea un enlace de dos vías en un elemento input o un componente. Para más detalles del uso y otras notas, lee la sección de la guía enlazada debajo.

- **Lee también:**
  - [Enlace de elementos input de formularios](../guide/forms.html)
  - [Componentes - Componentes input de formularios utilizando eventos personalizados](../guide/components.html#Form-Input-Components-using-Custom-Events)

### v-pre

- **No espera expresión**

- **Uso:**

  Saltea la compilación de este elemento y todos sus hijos. Puede utilizarlo para mostrar etiquetas 'mustache'. Saltear un gran número de nodos sin directivas en ellos puede acelerar la compilación.

- **Ejemplo:**

  ```html
  <span v-pre>{{ this will not be compiled }}</span>
   ```

### v-cloak

- **No espera expresión**

- **Uso:**

  Esta directiva permanecerá en el elemento hasta que la instancia Vue asociada termine de compilar. Combinado con una regla CSS como `[v-cloak] { display: none }`, esta directiva puede ser utilizada para esconder enlaces 'mustache' sin compilar hasta que la instancia de Vue esté lista.

- **Ejemplo:**

  ```css
  [v-cloak] {
    display: none;
  }
  ```

  ```html
  <div v-cloak>
    {{ message }}
  </div>
  ```

  El `<div>` no será visible hasta que la compilación esté terminada.

### v-once

- **No espera expresión**

- **Detalles:**

  Renderiza el elemento y componente solo **una vez**. En re-renderizados subsecuentes, el elemento/componente y todos sus hijos seran tratados como contenido estático y se saltearán. Esto puede ser utilizado para optimizar las actualizaciones.

  ```html
  <!-- elemento único -->
  <span v-once>This will never change: {{msg}}</span>
  <!-- el elemento tiene hijos -->
  <div v-once>
    <h1>comment</h1>
    <p>{{msg}}</p>
  </div>
  <!-- componente -->
  <my-component v-once :comment="msg"></my-component>
  <!-- directiva v-for -->
  <ul>
    <li v-for="i in list" v-once>{{i}}</li>
  </ul>
  ```

- **Lee también:**
  - [Sintaxis de enlace de datos - interpolaciones](../guide/syntax.html#Text)
  - [Componentes - Componentes estáticos baratos con v-once](../guide/components.html#Cheap-Static-Components-with-v-once)

## Atributos especiales

### key

- **Espera:** `string`

  El atributo especial `key` es utilizado principalmente como una pista para el algoritmo de DOM virtual de Vue para identificar VNodes cuando se compara la nueva lista de nodos con la antigua. Sin `key`, Vue utiliza un algoritmo que minimiza el movimiento de elementos e intenta actualizar/reutilizar elementos del mismo tipo en su lugar siempre que sea posible. Con `key`, reordenará los elementos basado en el cambio de orden de `key`, y los elementos sin este que no esten más presentes serán removidos/destruidos siempre.

  Los hijos del mismo padre deben tener **keys únicas**. Las `key` duplicadas causarán errores de renderizado.

  El caso de uso más común es en combinación con `v-for`:

  ``` html
  <ul>
    <li v-for="item in items" :key="item.id">...</li>
  </ul>
  ```

  Puede ser utilizado para forzar el reemplazo de un elemento/componente en lugar de reutilizarlo. Esto puede ser útil cuando deseas:

  - Disparar correctamente hooks de ciclo de vida de un componente
  - Disparar transiciones

  Por ejemplo:

  ``` html
  <transition>
    <span :key="text">{{ text }}</span>
  </transition>
  ```

  Cuando `text` cambia, el `<span>` será reemplazado siempre en lugar de actualizado, por lo que una transición será disparada.

### ref

- **Espera:** `string`

  `ref` es utilizada para registrar una referencia a un elemento o componente hijo. La referencia será registrada en el objeto `$refs` del componente padre. Si se utiliza en un elemento DOM plano, la referencia será ese elemento; si se utiliza en un componente hijo, la referencia será la instancia del componente:

  ``` html
  <!-- vm.$refs.p será el nodo del DOM -->
  <p ref="p">hello</p>

  <!-- vm.$refs.child será la instancia del componente hijo -->
  <child-comp ref="child"></child-comp>
  ```

  Cuando se utiliza en elementos/componentes con `v-for`, la referencia registrada será un arreglo que contendrá nodos DOM o instancias de componentes.

  Una nota importante acerca del timing del registro: debido a que las refs son creadas como resultado de una función de renderizado, no puede accederlas durante el renderizado inicial porque ¡todavía no existen!. Además `$refs` no es reactivo, por lo tanto, no debes intentar utilizarlo en las plantillas para enlazado de datos.

- **Lee también:** [Referencias a componentes hijos](../guide/components.html#Child-Component-Refs)

### slot

- **Espera:** `string`

  Utilizado con el contenido insertado dentro de componentes hijos para indicar a que slot con nombre pertenece ese contenido.

  Para un uso detallado, lee la sección de la guía enlazada debajo.

- **Lee también:** [Slots con nombre](../guide/components.html#Named-Slots)

## Componentes incorporados

### component

- **Props:**
  - `is` - string | ComponentDefinition | ComponentConstructor
  - `inline-template` - boolean

- **Uso:**

  Un "meta componente" para renderizar componentes dinámicos. El componente a renderizar es determinado por la propiedad `is`:

  ```html
  <!-- un componente dinámico controlado por -->
  <!-- la propiedad `componentId` de la vm -->
  <component :is="componentId"></component>

  <!-- también puede renderizar componentes registrados o pasados como propiedada -->
  <component :is="$options.components.child"></component>
  ```

- **Lee también:** [Componentes dinámicos](../guide/components.html#Dynamic-Components)

### transition

- **Props:**
  - `name` - string, utilizado para generar automáticamente nombres de clases CSS de transiciones. Por ejemplo, `name: 'fade'` se auto-expanderá a `.fade-enter`, `.fade-enter-active`, etc. Valor por defecto: `"v"`.
  - `appear` - boolean, indica si aplicar o no transiciones en el renderizado inicial. Valor por defecto: `false`.
  - `css` - boolean, indica si aplicar o no clases de transición CSS. Valor por defecto: `true`. Si se setea como `false`, solo disparará hooks JavaScript registrados a travpes de eventos de componente.
  - `type` - string, especifica que tipo de eventos de transición esperar para determinar el tiempo de finalización de la transición. Los valores disponibles son `"transition"` y `"animation"`. Por defecto, detectará automáticamente el tipo con una mayor duración.
  - `mode` - string, controla la secuencia de temporización para las transiciones de entrada/salida. Los modos disponibles son `"out-in"` y `"in-out"`.
  - `enter-class` - string
  - `leave-class` - string
  - `appear-class` - string
  - `enter-to-class` - string
  - `leave-to-class` - string
  - `appear-to-class` - string
  - `enter-active-class` - string
  - `leave-active-class` - string
  - `appear-active-class` - string

- **Events:**
  - `before-enter`
  - `before-leave`
  - `before-appear`
  - `enter`
  - `leave`
  - `appear`
  - `after-enter`
  - `after-leave`
  - `after-appear`
  - `enter-cancelled`
  - `leave-cancelled` (solo `v-show`)
  - `appear-cancelled`

- **Uso:**

  `<transition>` sirve como efecto de transición para **un solo** elemento/componente. `<transition>` no renderiza un elemento DOM extra, ni aparece en la jerarquía de componentes si se inspecciona. Simplemente aplica el comportamiento de transición al contenido envuelto.

  ```html
  <!-- elemento simple -->
  <transition>
    <div v-if="ok">toggled content</div>
  </transition>

  <!-- componente dinámico -->
  <transition name="fade" mode="out-in" appear>
    <component :is="view"></component>
  </transition>

  <!-- hook de evento -->
  <div id="transition-demo">
    <transition @after-enter="transitionComplete">
      <div v-show="ok">toggled content</div>
    </transition>
  </div>
  ```

  ``` js
  new Vue({
    ...
    methods: {
      transitionComplete: function (el) {
        // for passed 'el' that DOM element as the argument, something ...
      }
    }
    ...
  }).$mount('#transition-demo')
  ```

- **Lee también:** [Transiciones: entrada, salida y listas](../guide/transitions.html)

### transition-group

- **Props:**
  - `tag` - string, valor por defecto: `span`.
  - `move-class` - sobreescribe las clases CSS aplicadas durante las transiciones de movimiento.
  - expone las mismas propiedades que `<transition>` excepto `mode`.

- **Events:**
  - expone los mismos eventos que `<transition>`.

- **Uso:**

  `<transition-group>` sirve como efecto de transición para **múltiples** elementos/componentes. `<transition-group>` renderiza un elemento del DOM real. Por defecto, un `<span>`, y puedes configurar que elemento debería ser renderizado a través del atributo `tag`.

  Nota que cada hijo en un `<transition-group>` debe estar **identificado unívocamente** para que las animaciones funcionen correctamente.

  `<transition-group>` soporta transiciones de movimiento a través de transformaciones CSS. Cuando la posición de un hijo en la pantalla ha cambiado luego de una actualización, se le aplicará una clase CSS de movimiento (generada automáticamente desde el atributo `name` o configurada con el atributo `move-class`). Si la propiedad CSS `transform` puede "transicionar" cuando la clase de movimiento es aplicada, el elemento será animado suavemente hasta su posición utilizando la [técnica FLIP](https://aerotwist.com/blog/flip-your-animations/).

  ```html
  <transition-group tag="ul" name="slide">
    <li v-for="item in items" :key="item.id">
      {{ item.text }}
    </li>
  </transition-group>
  ```

- **Lee también:** [Transiciones: entrada, salida y listas](../guide/transitions.html)

### keep-alive

- **Props:**
  - `include` - string o RegExp. Solo los componentes que coincidan con esto serán guardadas en caché.
  - `exclude` - string o RegExp. Cualquier componente que coincida con esto no será guardado en caché.

- **Uso:**

  Cuando envuelva un componente dinámico, `<keep-alive>` guarda en memoria caché las instancias del componente inactivo sin destruirlas. De manera similar a `<transition>`, `<keep-alive>` es un componente abstracto: no renderiza un elemento DOM extra y no aparece en el árbol de jerarquía del padre.

  Cuando un componente es mostrado/escondido dentro de `<keep-alive>`, sus hooks de ciclo de vida `activated` y `deactivated` serán invocados según corresponda.

  Es utilizado principalmente para preservar el estado de los componentes o para evitar el re-renderizado.

  ```html
  <!-- basico -->
  <keep-alive>
    <component :is="view"></component>
  </keep-alive>

  <!-- múltiples hijos condicionales -->
  <keep-alive>
    <comp-a v-if="a > 1"></comp-a>
    <comp-b v-else></comp-b>
  </keep-alive>

  <!-- utilizado en conjunto con <transition> -->
  <transition>
    <keep-alive>
      <component :is="view"></component>
    </keep-alive>
  </transition>
  ```

- **`include` y `exclude`**

  > Nuevo en 2.1.0

  Las propiedades `include` y `exclude` permiten a los componentes ser guardados en caché condicionalmente. Ambas propiedades pueden ser tanto una cadena de texto delimitada por comas como una RegExp:

  ``` html
  <!-- cadena de texto delimitada por comas -->
  <keep-alive include="a,b">
    <component :is="view"></component>
  </keep-alive>

  <!-- regex (utiliza v-bind) -->
  <keep-alive :include="/a|b/">
    <component :is="view"></component>
  </keep-alive>
  ```

  La concordancia se chequea primero contra la opción `name` propia del componente y luego contra el nombre de registro local (el `key` en la opción `components` del padre) si la opción `name` no está disponible. Los componentes anónimos no pueden ser testeados.

  <p class="tip">`<keep-alive>` no funciona con componentes funcionales porque no tienen instancias que puedan ser guardadas en memoria caché.</p>

- **Lee también:** [Componentes dinámicos - keep-alive](../guide/components.html#keep-alive)

### slot

- **Props:**
  - `name` - string, utilizado para slots con nombre.

- **Uso:**

  `<slot>` sirve como contenedor de distribución de contenido en plantillas de componentes. `<slot>` será reemplazado.

  Para un uso detallado, lee la sección de la guía enlazada debajo.

- **Lee también:** [Distribución de contenido mediante slots](../guide/components.html#Content-Distribution-with-Slots)

## Interface VNode

- Por favor, lee la [declaración de clase VNode](https://github.com/vuejs/vue/blob/dev/src/core/vdom/vnode.js).

## Renderizado lado servidor

- Por favor, lee la [documentación del paquete vue-server-renderer](https://github.com/vuejs/vue/tree/dev/packages/vue-server-renderer).
