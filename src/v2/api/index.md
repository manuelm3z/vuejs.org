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

  Change the plain text interpolation delimiters. **This option is only available in the standalone build.**

- **Ejemplo:**

  ``` js
  new Vue({
    delimiters: ['${', '}']
  })

  // Delimiters changed to ES6 template string style
  ```

### functional

- **Tipo:** `boolean`

- **Detalles:**

  Causes a component to be stateless (no `data`) and instanceless (no `this` context). They are simply a `render` function that returns virtual nodes making them much cheaper to render.

- **Lee también:** [Functional Components](../guide/render-function.html#Functional-Components)

## Instance Properties

### vm.$data

- **Tipo:** `Object`

- **Detalles:**

  The data object that the Vue instance is observing. The Vue instance proxies access to the properties on its data object.

- **Lee también:** [Options - data](#data)

### vm.$el

- **Tipo:** `HTMLElement`

- **Read only**

- **Detalles:**

  The root DOM element that the Vue instance is managing.

### vm.$options

- **Tipo:** `Object`

- **Read only**

- **Detalles:**

  The instantiation options used for the current Vue instance. This is useful when you want to include custom properties in the options:

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

- **Read only**

- **Detalles:**

  The parent instance, if the current instance has one.

### vm.$root

- **Tipo:** `Vue instance`

- **Read only**

- **Detalles:**

  The root Vue instance of the current component tree. If the current instance has no parents this value will be itself.

### vm.$children

- **Tipo:** `Array<Vue instance>`

- **Read only**

- **Detalles:**

  The direct child components of the current instance. **Note there's no order guarantee for `$children`, and it is not reactive.** If you find yourself trying to use `$children` for data binding, consider using an Array and `v-for` to generate child components, and use the Array as the source of truth.

### vm.$slots

- **Tipo:** `{ [name: string]: ?Array<VNode> }`

- **Read only**

- **Detalles:**

  Used to programmatically access content [distributed by slots](../guide/components.html#Content-Distribution-with-Slots). Each [named slot](../guide/components.html#Named-Slots) has its own corresponding property (e.g. the contents of `slot="foo"` will be found at `vm.$slots.foo`). The `default` property contains any nodes not included in a named slot.

  Accessing `vm.$slots` is most useful when writing a component with a [render function](../guide/render-function.html).

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
  - [`<slot>` Component](#slot-1)
  - [Content Distribution with Slots](../guide/components.html#Content-Distribution-with-Slots)
  - [Render Functions: Slots](../guide/render-function.html#Slots)

### vm.$scopedSlots

> New in 2.1.0

- **Tipo:** `{ [name: string]: props => VNode | Array<VNode> }`

- **Read only**

- **Detalles:**

  Used to programmatically access [scoped slots](../guide/components.html#Scoped-Slots). For each slot, including the `default` one, the object contains a corresponding function that returns VNodes.

  Accessing `vm.$scopedSlots` is most useful when writing a component with a [render function](../guide/render-function.html).

- **Lee también:**
  - [`<slot>` Component](#slot-1)
  - [Scoped Slots](../guide/components.html#Scoped-Slots)
  - [Render Functions: Slots](../guide/render-function.html#Slots)

### vm.$refs

- **Tipo:** `Object`

- **Read only**

- **Detalles:**

  An object that holds child components that have `ref` registered.

- **Lee también:**
  - [Child Component Refs](../guide/components.html#Child-Component-Refs)
  - [ref](#ref)

### vm.$isServer

- **Tipo:** `boolean`

- **Read only**

- **Detalles:**

  Whether the current Vue instance is running on the server.

- **Lee también:** [Server-Side Rendering](../guide/ssr.html)

## Instance Methods / Data

<h3 id="vm-watch">vm.$watch( expOrFn, callback, [options] )</h3>

- **Argumentos:**
  - `{string | Function} expOrFn`
  - `{Function} callback`
  - `{Object} [options]`
    - `{boolean} deep`
    - `{boolean} immediate`

- **Devuelve:** `{Function} unwatch`

- **Uso:**

  Watch an expression or a computed function on the Vue instance for changes. The callback gets called with the new value and the old value. The expression only accepts simple dot-delimited paths. For more complex expression, use a function instead.

<p class="tip">Note: when mutating (rather than replacing) an Object or an Array, the old value will be the same as new value because they reference the same Object/Array. Vue doesn't keep a copy of the pre-mutate value.</p>

- **Ejemplo:**

  ``` js
  // keypath
  vm.$watch('a.b.c', function (newVal, oldVal) {
    // do something
  })

  // function
  vm.$watch(
    function () {
      return this.a + this.b
    },
    function (newVal, oldVal) {
      // do something
    }
  )
  ```

  `vm.$watch` returns an unwatch function that stops firing the callback:

  ``` js
  var unwatch = vm.$watch('a', cb)
  // later, teardown the watcher
  unwatch()
  ```

- **Option: deep**

  To also detect nested value changes inside Objects, you need to pass in `deep: true` in the options argument. Note that you don't need to do so to listen for Array mutations.

  ``` js
  vm.$watch('someObject', callback, {
    deep: true
  })
  vm.someObject.nestedValue = 123
  // callback is fired
  ```

- **Option: immediate**

  Passing in `immediate: true` in the option will trigger the callback immediately with the current value of the expression:

  ``` js
  vm.$watch('a', callback, {
    immediate: true
  })
  // callback is fired immediately with current value of `a`
  ```

<h3 id="vm-set">vm.$set( object, key, value )</h3>

- **Argumentos:**
  - `{Object} object`
  - `{string} key`
  - `{any} value`

- **Devuelve:** the set value.

- **Uso:**

  This is the **alias** of the global `Vue.set`.

- **Lee también:** [Vue.set](#Vue-set)

<h3 id="vm-delete">vm.$delete( object, key )</h3>

- **Argumentos:**
  - `{Object} object`
  - `{string} key`

- **Uso:**

  This is the **alias** of the global `Vue.delete`.

- **Lee también:** [Vue.delete](#Vue-delete)

## Instance Methods / Events

<h3 id="vm-on">vm.$on( event, callback )</h3>

- **Argumentos:**
  - `{string} event`
  - `{Function} callback`

- **Uso:**

  Listen for a custom event on the current vm. Events can be triggered by `vm.$emit`. The callback will receive all the additional arguments passed into these event-triggering methods.

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

  Listen for a custom event, but only once. The listener will be removed once it triggers for the first time.

<h3 id="vm-off">vm.$off( [event, callback] )</h3>

- **Argumentos:**
  - `{string} [event]`
  - `{Function} [callback]`

- **Uso:**

  Remove event listener(s).

  - If no arguments are provided, remove all event listeners;

  - If only the event is provided, remove all listeners for that event;

  - If both event and callback are given, remove the listener for that specific callback only.

<h3 id="vm-emit">vm.$emit( event, [...args] )</h3>

- **Argumentos:**
  - `{string} event`
  - `[...args]`

  Trigger an event on the current instance. Any additional arguments will be passed into the listener's callback function.

## Instance Methods / Lifecycle

<h3 id="vm-mount">vm.$mount( [elementOrSelector] )</h3>

- **Argumentos:**
  - `{Element | string} [elementOrSelector]`
  - `{boolean} [hydrating]`

- **Devuelve:** `vm` - the instance itself

- **Uso:**

  If a Vue instance didn't receive the `el` option at instantiation, it will be in "unmounted" state, without an associated DOM element. `vm.$mount()` can be used to manually start the mounting of an unmounted Vue instance.

  If `elementOrSelector` argument is not provided, the template will be rendered as an off-document element, and you will have to use native DOM API to insert it into the document yourself.

  The method returns the instance itself so you can chain other instance methods after it.

- **Ejemplo:**

  ``` js
  var MyComponent = Vue.extend({
    template: '<div>Hello!</div>'
  })

  // create and mount to #app (will replace #app)
  new MyComponent().$mount('#app')

  // the above is the same as:
  new MyComponent({ el: '#app' })

  // or, render off-document and append afterwards:
  var component = new MyComponent().$mount()
  document.getElementById('app').appendChild(component.$el)
  ```

- **Lee también:**
  - [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)
  - [Server-Side Rendering](../guide/ssr.html)

<h3 id="vm-forceUpdate">vm.$forceUpdate()</h3>

- **Uso:**

  Force the Vue instance to re-render. Note it does not affect all child components, only the instance itself and child components with inserted slot content.

<h3 id="vm-nextTick">vm.$nextTick( [callback] )</h3>

- **Argumentos:**
  - `{Function} [callback]`

- **Uso:**

  Defer the callback to be executed after the next DOM update cycle. Use it immediately after you've changed some data to wait for the DOM update. This is the same as the global `Vue.nextTick`, except that the callback's `this` context is automatically bound to the instance calling this method.

  > New in 2.1.0: returns a Promise if no callback is provided and Promise is supported in the execution environment.

- **Ejemplo:**

  ``` js
  new Vue({
    // ...
    methods: {
      // ...
      example: function () {
        // modify data
        this.message = 'changed'
        // DOM is not updated yet
        this.$nextTick(function () {
          // DOM is now updated
          // `this` is bound to the current instance
          this.doSomethingElse()
        })
      }
    }
  })
  ```

- **Lee también:**
  - [Vue.nextTick](#Vue-nextTick)
  - [Async Update Queue](../guide/reactivity.html#Async-Update-Queue)

<h3 id="vm-destroy">vm.$destroy()</h3>

- **Uso:**

  Completely destroy a vm. Clean up its connections with other existing vms, unbind all its directives, turn off all event listeners.

  Triggers the `beforeDestroy` and `destroyed` hooks.

  <p class="tip">In normal use cases you shouldn't have to call this method yourself. Prefer controlling the lifecycle of child components in a data-driven fashion using `v-if` and `v-for`.</p>

- **Lee también:** [Diagrama del ciclo de vida](../guide/instance.html#Lifecycle-Diagram)

## Directives

### v-text

- **Expects:** `string`

- **Detalles:**

  Updates the element's `textContent`. If you need to update the part of `textContent`, you should use `{% raw %}{{ Mustache }}{% endraw %}` interpolations.

- **Ejemplo:**

  ```html
  <span v-text="msg"></span>
  <!-- same as -->
  <span>{{msg}}</span>
  ```

- **Lee también:** [Data Binding Syntax - interpolations](../guide/syntax.html#Text)

### v-html

- **Expects:** `string`

- **Detalles:**

  Updates the element's `innerHTML`. **Note that the contents are inserted as plain HTML - they will not be compiled as Vue templates**. If you find yourself trying to compose templates using `v-html`, try to rethink the solution by using components instead.

  <p class="tip">Dynamically rendering arbitrary HTML on your website can be very dangerous because it can easily lead to [XSS attacks](https://en.wikipedia.org/wiki/Cross-site_scripting). Only use `v-html` on trusted content and **never** on user-provided content.</p>

- **Ejemplo:**

  ```html
  <div v-html="html"></div>
  ```
- **Lee también:** [Data Binding Syntax - interpolations](../guide/syntax.html#Raw-HTML)

### v-show

- **Expects:** `any`

- **Uso:**

  Toggle's the element's `display` CSS property based on the truthy-ness of the expression value.

  This directive triggers transitions when its condition changes.

- **Lee también:** [Conditional Rendering - v-show](../guide/conditional.html#v-show)

### v-if

- **Expects:** `any`

- **Uso:**

  Conditionally render the element based on the truthy-ness of the expression value. The element and its contained directives / components are destroyed and re-constructed during toggles. If the element is a `<template>` element, its content will be extracted as the conditional block.

  This directive triggers transitions when its condition changes.

  <p class="tip">When used together with v-if, v-for has a higher priority than v-if. See the <a href="../guide/list.html#v-for-with-v-if">list rendering guide</a> for details.</p>

- **Lee también:** [Conditional Rendering - v-if](../guide/conditional.html)

### v-else

- **Does not expect expression**

- **Restricciones:** previous sibling element must have `v-if` or `v-else-if`.

- **Uso:**

  Denote the "else block" for `v-if` or a `v-if`/`v-else-if` chain.

  ```html
  <div v-if="Math.random() > 0.5">
    Now you see me
  </div>
  <div v-else>
    Now you don't
  </div>
  ```

- **Lee también:**
  - [Conditional Rendering - v-else](../guide/conditional.html#v-else)

### v-else-if

> New in 2.1.0

- **Expects:** `any`

- **Restricciones:** previous sibling element must have `v-if` or `v-else-if`.

- **Uso:**

  Denote the "else if block" for `v-if`. Can be chained.

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

- **Lee también:** [Conditional Rendering - v-else-if](../guide/conditional.html#v-else-if)

### v-for

- **Expects:** `Array | Object | number | string`

- **Uso:**

  Render the element or template block multiple times based on the source data. The directive's value must use the special syntax `alias in expression` to provide an alias for the current element being iterated on:

  ``` html
  <div v-for="item in items">
    {{ item.text }}
  </div>
  ```

  Alternatively, you can also specify an alias for the index (or the key if used on an Object):

  ``` html
  <div v-for="(item, index) in items"></div>
  <div v-for="(val, key) in object"></div>
  <div v-for="(val, key, index) in object"></div>
  ```

  The default behavior of `v-for` will try to patch the elements in-place without moving them. To force it to reorder elements, you need to provide an ordering hint with the `key` special attribute:

  ``` html
  <div v-for="item in items" :key="item.id">
    {{ item.text }}
  </div>
  ```

  <p class="tip">When used together with v-if, v-for has a higher priority than v-if. See the <a href="../guide/list.html#v-for-with-v-if">list rendering guide</a> for details.</p>

  The detailed usage for `v-for` is explained in the guide section linked below.

- **Lee también:**
  - [List Rendering](../guide/list.html)
  - [key](../guide/list.html#key)

### v-on

- **Shorthand:** `@`

- **Expects:** `Function | Inline Statement`

- **Argument:** `event (required)`

- **Modifiers:**
  - `.stop` - call `event.stopPropagation()`.
  - `.prevent` - call `event.preventDefault()`.
  - `.capture` - add event listener in capture mode.
  - `.self` - only trigger handler if event was dispatched from this element.
  - `.{keyCode | keyAlias}` - only trigger handler on certain keys.
  - `.native` - listen for a native event on the root element of component.
  - `.once` - trigger handler at most once.

- **Uso:**

  Attaches an event listener to the element. The event type is denoted by the argument. The expression can either be a method name or an inline statement, or simply omitted when there are modifiers present.

  When used on a normal element, it listens to **native DOM events** only. When used on a custom element component, it also listens to **custom events** emitted on that child component.

  When listening to native DOM events, the method receives the native event as the only argument. If using inline statement, the statement has access to the special `$event` property: `v-on:click="handle('ok', $event)"`.

- **Ejemplo:**

  ```html
  <!-- method handler -->
  <button v-on:click="doThis"></button>

  <!-- inline statement -->
  <button v-on:click="doThat('hello', $event)"></button>

  <!-- shorthand -->
  <button @click="doThis"></button>

  <!-- stop propagation -->
  <button @click.stop="doThis"></button>

  <!-- prevent default -->
  <button @click.prevent="doThis"></button>

  <!-- prevent default without expression -->
  <form @submit.prevent></form>

  <!-- chain modifiers -->
  <button @click.stop.prevent="doThis"></button>

  <!-- key modifier using keyAlias -->
  <input @keyup.enter="onEnter">

  <!-- key modifier using keyCode -->
  <input @keyup.13="onEnter">

  <!-- the click event will be triggered at most once -->
  <button v-on:click.once="doThis"></button>
  ```

  Listening to custom events on a child component (the handler is called when "my-event" is emitted on the child):

  ```html
  <my-component @my-event="handleThis"></my-component>

  <!-- inline statement -->
  <my-component @my-event="handleThis(123, $event)"></my-component>

  <!-- native event on component -->
  <my-component @click.native="onClick"></my-component>
  ```

- **Lee también:**
  - [Methods and Event Handling](../guide/events.html)
  - [Components - Custom Events](../guide/components.html#Custom-Events)

### v-bind

- **Shorthand:** `:`

- **Expects:** `any (with argument) | Object (without argument)`

- **Argument:** `attrOrProp (optional)`

- **Modifiers:**
  - `.prop` - Bind as a DOM property instead of an attribute. ([what's the difference?](http://stackoverflow.com/questions/6003819/properties-and-attributes-in-html#answer-6004028))
  - `.camel` - transform the kebab-case attribute name into camelCase. (supported since 2.1.0)

- **Uso:**

  Dynamically bind one or more attributes, or a component prop to an expression.

  When used to bind the `class` or `style` attribute, it supports additional value types such as Array or Objects. See linked guide section below for more details.

  When used for prop binding, the prop must be properly declared in the child component.

  When used without an argument, can be used to bind an object containing attribute name-value pairs. Note in this mode `class` and `style` does not support Array or Objects.

- **Ejemplo:**

  ```html
  <!-- bind an attribute -->
  <img v-bind:src="imageSrc">

  <!-- shorthand -->
  <img :src="imageSrc">

  <!-- with inline string concatenation -->
  <img :src="'/path/to/images/' + fileName">

  <!-- class binding -->
  <div :class="{ red: isRed }"></div>
  <div :class="[classA, classB]"></div>
  <div :class="[classA, { classB: isB, classC: isC }]">

  <!-- style binding -->
  <div :style="{ fontSize: size + 'px' }"></div>
  <div :style="[styleObjectA, styleObjectB]"></div>

  <!-- binding an object of attributes -->
  <div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>

  <!-- DOM attribute binding with prop modifier -->
  <div v-bind:text-content.prop="text"></div>

  <!-- prop binding. "prop" must be declared in my-component. -->
  <my-component :prop="someThing"></my-component>

  <!-- XLink -->
  <svg><a :xlink:special="foo"></a></svg>
  ```

  The `.camel` modifier allows camelizing a `v-bind` attribute name when using in-DOM templates, e.g. the SVG `viewBox` attribute:

  ``` html
  <svg :view-box.camel="viewBox"></svg>
  ```

  `.camel` is not needed if you are using string templates, or compiling with `vue-loader`/`vueify`.

- **Lee también:**
  - [Class and Style Bindings](../guide/class-and-style.html)
  - [Components - Component Props](../guide/components.html#Props)

### v-model

- **Expects:** varies based on value of form inputs element or output of components

- **Limited to:**
  - `<input>`
  - `<select>`
  - `<textarea>`
  - components

- **Modifiers:**
  - [`.lazy`](../guide/forms.html#lazy) - listen to `change` events instead of `input`
  - [`.number`](../guide/forms.html#number) - cast input string to numbers
  - [`.trim`](../guide/forms.html#trim) - trim input

- **Uso:**

  Create a two-way binding on a form input element or a component. For detailed usage and other notes, see the Guide section linked below.

- **Lee también:**
  - [Form Input Bindings](../guide/forms.html)
  - [Components - Form Input Components using Custom Events](../guide/components.html#Form-Input-Components-using-Custom-Events)

### v-pre

- **Does not expect expression**

- **Uso:**

  Skip compilation for this element and all its children. You can use this for displaying raw mustache tags. Skipping large numbers of nodes with no directives on them can also speed up compilation.

- **Ejemplo:**

  ```html
  <span v-pre>{{ this will not be compiled }}</span>
   ```

### v-cloak

- **Does not expect expression**

- **Uso:**

  This directive will remain on the element until the associated Vue instance finishes compilation. Combined with CSS rules such as `[v-cloak] { display: none }`, this directive can be used to hide un-compiled mustache bindings until the Vue instance is ready.

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

  The `<div>` will not be visible until the compilation is done.

### v-once

- **Does not expect expression**

- **Detalles:**

  Render the element and component **once** only. On subsequent re-renders, the element/component and all its children will be treated as static content and skipped. This can be used to optimize update performance.

  ```html
  <!-- single element -->
  <span v-once>This will never change: {{msg}}</span>
  <!-- the element have children -->
  <div v-once>
    <h1>comment</h1>
    <p>{{msg}}</p>
  </div>
  <!-- component -->
  <my-component v-once :comment="msg"></my-component>
  <!-- v-for directive -->
  <ul>
    <li v-for="i in list" v-once>{{i}}</li>
  </ul>
  ```

- **Lee también:**
  - [Data Binding Syntax - interpolations](../guide/syntax.html#Text)
  - [Components - Cheap Static Components with v-once](../guide/components.html#Cheap-Static-Components-with-v-once)

## Special Attributes

### key

- **Expects:** `string`

  The `key` special attribute is primarily used as a hint for Vue's virtual DOM algorithm to identify VNodes when diffing the new list of nodes against the old list. Without keys, Vue uses an algorithm that minimizes element movement and tries to patch/reuse elements of the same type in-place as much as possible. With keys, it will reorder elements based on the order change of keys, and elements with keys that are no longer present will always be removed/destroyed.

  Children of the same common parent must have **unique keys**. Duplicate keys will cause render errors.

  The most common use case is combined with `v-for`:

  ``` html
  <ul>
    <li v-for="item in items" :key="item.id">...</li>
  </ul>
  ```

  It can also be used to force replacement of an element/component instead of reusing it. This can be useful when you want to:

  - Properly trigger lifecycle hooks of a component
  - Trigger transitions

  For example:

  ``` html
  <transition>
    <span :key="text">{{ text }}</span>
  </transition>
  ```

  When `text` changes, the `<span>` will always be replaced instead of patched, so a transition will be triggered.

### ref

- **Expects:** `string`

  `ref` is used to register a reference to an element or a child component. The reference will be registered under the parent component's `$refs` object. If used on a plain DOM element, the reference will be that element; if used on a child component, the reference will be component instance:

  ``` html
  <!-- vm.$refs.p will be the DOM node -->
  <p ref="p">hello</p>

  <!-- vm.$refs.child will be the child comp instance -->
  <child-comp ref="child"></child-comp>
  ```

  When used on elements/components with `v-for`, the registered reference will be an Array containing DOM nodes or component instances.

  An important note about the ref registration timing: because the refs themselves are created as a result of the render function, you cannot access them on the initial render - they don't exist yet! `$refs` is also non-reactive, therefore you should not attempt to use it in templates for data-binding.

- **Lee también:** [Child Component Refs](../guide/components.html#Child-Component-Refs)

### slot

- **Expects:** `string`

  Used on content inserted into child components to indicate which named slot the content belongs to.

  For detailed usage, see the guide section linked below.

- **Lee también:** [Named Slots](../guide/components.html#Named-Slots)

## Built-In Components

### component

- **Props:**
  - `is` - string | ComponentDefinition | ComponentConstructor
  - `inline-template` - boolean

- **Uso:**

  A "meta component" for rendering dynamic components. The actual component to render is determined by the `is` prop:

  ```html
  <!-- a dynamic component controlled by -->
  <!-- the `componentId` property on the vm -->
  <component :is="componentId"></component>

  <!-- can also render registered component or component passed as prop -->
  <component :is="$options.components.child"></component>
  ```

- **Lee también:** [Dynamic Components](../guide/components.html#Dynamic-Components)

### transition

- **Props:**
  - `name` - string, Used to automatically generate transition CSS class names. e.g. `name: 'fade'` will auto expand to `.fade-enter`, `.fade-enter-active`, etc. Defaults to `"v"`.
  - `appear` - boolean, Whether to apply transition on initial render. Defaults to `false`.
  - `css` - boolean, Whether to apply CSS transition classes. Defaults to `true`. If set to `false`, will only trigger JavaScript hooks registered via component events.
  - `type` - string, Specify the type of transition events to wait for to determine transition end timing. Available values are `"transition"` and `"animation"`. By default, it will automatically detect the type that has a longer duration.
  - `mode` - string, Controls the timing sequence of leaving/entering transitions. Available modes are `"out-in"` and `"in-out"`; defaults to simultaneous.
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
  - `leave-cancelled` (`v-show` only)
  - `appear-cancelled`

- **Uso:**

  `<transition>` serve as transition effects for **single** element/component. The `<transition>` does not render an extra DOM element, nor does it show up in the inspected component hierarchy. It simply applies the transition behavior to the wrapped content inside.

  ```html
  <!-- simple element -->
  <transition>
    <div v-if="ok">toggled content</div>
  </transition>

  <!-- dynamic component -->
  <transition name="fade" mode="out-in" appear>
    <component :is="view"></component>
  </transition>

  <!-- event hooking -->
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

- **Lee también:** [Transitions: Entering, Leaving, and Lists](../guide/transitions.html)

### transition-group

- **Props:**
  - `tag` - string, defaults to `span`.
  - `move-class` - overwrite CSS class applied during moving transition.
  - exposes the same props as `<transition>` except `mode`.

- **Events:**
  - exposes the same events as `<transition>`.

- **Uso:**

  `<transition-group>` serve as transition effects for **multiple** elements/components. The `<transition-group>` renders a real DOM element. By default it renders a `<span>`, and you can configure what element is should render via the `tag` attribute.

  Note every child in a `<transition-group>` must be **uniquely keyed** for the animations to work properly.

  `<transition-group>` supports moving transitions via CSS transform. When a child's position on screen has changed after an updated, it will get applied a moving CSS class (auto generated from the `name` attribute or configured with the `move-class` attribute). If the CSS `transform` property is "transition-able" when the moving class is applied, the element will be smoothly animated to its destination using the [FLIP technique](https://aerotwist.com/blog/flip-your-animations/).

  ```html
  <transition-group tag="ul" name="slide">
    <li v-for="item in items" :key="item.id">
      {{ item.text }}
    </li>
  </transition-group>
  ```

- **Lee también:** [Transitions: Entering, Leaving, and Lists](../guide/transitions.html)

### keep-alive

- **Props:**
  - `include` - string or RegExp. Only components matched by this will be cached.
  - `exclude` - string or RegExp. Any component matched by this will not be cached.

- **Uso:**

  When wrapped around a dynamic component, `<keep-alive>` caches the inactive component instances without destroying them. Similar to `<transition>`, `<keep-alive>` is an abstract component: it doesn't render a DOM element itself, and doesn't show up in the component parent chain.

  When a component is toggled inside `<keep-alive>`, its `activated` and `deactivated` lifecycle hooks will be invoked accordingly.

  Primarily used with preserve component state or avoid re-rendering.

  ```html
  <!-- basic -->
  <keep-alive>
    <component :is="view"></component>
  </keep-alive>

  <!-- multiple conditional children -->
  <keep-alive>
    <comp-a v-if="a > 1"></comp-a>
    <comp-b v-else></comp-b>
  </keep-alive>

  <!-- used together with <transition> -->
  <transition>
    <keep-alive>
      <component :is="view"></component>
    </keep-alive>
  </transition>
  ```

- **`include` and `exclude`**

  > New in 2.1.0

  The `include` and `exclude` props allow components to be conditionally cached. Both props can either be a comma-delimited string or a RegExp:

  ``` html
  <!-- comma-delimited string -->
  <keep-alive include="a,b">
    <component :is="view"></component>
  </keep-alive>

  <!-- regex (use v-bind) -->
  <keep-alive :include="/a|b/">
    <component :is="view"></component>
  </keep-alive>
  ```

  The match is first checked on the component's own `name` option, then its local registration name (the key in the parent's `components` option) if the `name` option is not available. Anonymous components cannot be matched against.

  <p class="tip">`<keep-alive>` does not work with functional components because they do not have instances to be cached.</p>

- **Lee también:** [Dynamic Components - keep-alive](../guide/components.html#keep-alive)

### slot

- **Props:**
  - `name` - string, Used for named slot.

- **Uso:**

  `<slot>` serve as content distribution outlets in component templates. `<slot>` itself will be replaced.

  For detailed usage, see the guide section linked below.

- **Lee también:** [Content Distribution with Slots](../guide/components.html#Content-Distribution-with-Slots)

## VNode Interface

- Please refer to the [VNode class declaration](https://github.com/vuejs/vue/blob/dev/src/core/vdom/vnode.js).

## Server-Side Rendering

- Please refer to the [vue-server-renderer package documentation](https://github.com/vuejs/vue/tree/dev/packages/vue-server-renderer).
