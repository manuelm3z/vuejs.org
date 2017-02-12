---
title: Reactividad en profundidad
type: guia
order: 12
---

Ya hemos cubierto la mayoría de los aspectos básicos de Vue, ¡es hora de sumergirnos en las profundidades! Una de las caraterísticas más distintivas de Vue es un sistema de reactividad poco intrusivo. Los modelos son simples objetos planos de JavaScript. Cuando los modificas, la vista se actualiza. Esto hace al manejo de estado algo muy simple e intuitivo, pero también es importante entender como trabaja para evitar algunas trampas comunes. En esta sección, vamos a introducirnos en algunos de los detalles de bajo nivel del sistema de reactividad de Vue.

## Cómo son registrados los cambios

Cuando pasas un objeto plano de JavaScript a una instancia de Vue como su opción `data`, Vue recorrerá todas sus propiedades y las convertirá en _getters_/_setters_ usando [Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty). Esto funciona solo con ES5 y es una característica irreproducible, razón por la cual Vue no soporta IE8 ni versiones anteriores.

Los _getters/setters_ son invisibles para el usuario, pero internamente permiten a Vue realizar seguimiento de dependencias y notificación de cambios cuando las propiedades son accedidas o modificadas. Una contra es que la consola del navegador da un formato diferente a los _getters/setters_ cuando se guarda un registro de los objetos de datos convertidos, por lo que pueda que quieras instalar [vue-devtools](https://github.com/vuejs/vue-devtools) para obtener una interface de inspección más amigable.

Cada instancia de un componente tiene su correspondiente instancia de **observador**, la cual registra cualquier propiedad "tocada" como dependencia durante la renderizacion del componente. Luego, cuando un _setter_ de una dependencia es disparado, notifica al observador, lo cual genera que el componente se renderice nuevamente.

![Reactivity Cycle](/images/data.png)

## Advertencias para la detección de cambios

Debido a las limitaciones de JavaScript moderno (y el abandono de `Object.observe`), Vue **no puede detectar cuando se agrega o quita una propiedad**. Dado que Vue realiza la el proceso de conversión a _getters/setters_ durante la inicialización de la instancia, las propiedades deben estar presentes en el objeto `data` para que puedan convertirse y ser reactivos. Por ejemplo:

``` js
var vm = new Vue({
  data: {
    a: 1
  }
})
// `vm.a` es reactiva ahora

vm.b = 2
// `vm.b` NO es reactiva
```

Vue no permite añadir dinámicamente propiedades reactivas a la raíz de una instancia ya creada. Sin embargo, es posible añadir propiedades reactivas a objetos anidados utilizando el método `Vue.set(object, key, value)`:

``` js
Vue.set(vm.someObject, 'b', 2)
```

También puedes utilizar el método de instancia `vm.$set`, el cual es simplemente un alias para el método global `Vue.set`:

``` js
this.$set(this.someObject, 'b', 2)
```

En ocasiones puede que quieras asignar algunas propiedades a un objeto existente, por ejemplo, utilizando `Object.assign()` o `_.extend()`. Sin embargo, nuevas propiedades agregadas al objeto no dispararan cambios. En esos casos, crea un nuevo objeto con las propiedades del objeto original y el que estás intentando fusionar:

``` js
// en lugar de `Object.assign(this.someObject, { a: 1, b: 2 })`
this.someObject = Object.assign({}, this.someObject, { a: 1, b: 2 })
```

También hay algunas advertencias relacionadas a los arreglos, las cuales fueron discutidas anteriormente en la [sección de renderización de listas](list.html#Caveats).

## Declarando propiedades reactivas

Dado que Vue no permite añadir dinámicamente propiedades reactivas en la raíz , debes inicializar las instancias de Vue declarando por adelantado todas las propiedades de datos, incluso si solo contienen un valor vacío:

``` js
var vm = new Vue({
  data: {
    // declara "message" con un valor vacío
    message: ''
  },
  template: '<div>{{ message }}</div>'
})
// luego dale un valor a `message`
vm.message = 'Hello!'
```

Si no declaras `message` en la opción `data`, Vue te advertirá que la funcion de renderizado está tratando de acceder a una propiedad que no existe.

Hay razones técnicas detrás de esta restricción: elimina un tipo de caso extremo en el sistema de seguimiento de dependencias y también hace que las instancias de Vue trabajen mejor en conjunto a sistemas de verificación de tipos. Pero hay otra consideración importante en terminos de mantenibilidad de código: el objeto `data` es como un esquema del estado de tu componente. Declarar todas las propiedades reactivas por adelantado hace que el componente sea más simple de entender cuando tengas que revisar el código u otro desarrollador tenga que leerlo.

## Cola de actualizaciones asíncronas

En caso que no lo hayas notado todavía, Vue realiza actualizaciones del DOM **asíncronicamente**. Cuando se observa un cambio en los datos, iniciará una cola y pondrá en espera todos los cambios en los datos que sucedan en el mismo ciclo de eventos. Si el mismo observador se dispara varias veces, será colocado en la cola solo una vez. Esta desduplicación de los cambios almacenados es importante para evitar cálculos innecesarios y manipulaciones del DOM. Luego, en el siguiente _tick_ del ciclo de eventos, Vue vacía la cola y realiza el trabajo real (el cual ya se encuentra desduplicado). Internamente, Vue intenta utilizar versiones nativas de `Promise.then` y `MutationObserver` para las colas asíncronas o recurre a`setTimeout(fn, 0)` en otro caso.

Por ejemplo, cuando estableces `vm.someData = 'new value'`, el componente no se volvera a renderizar inmediatamente. Se actualizará en el siguiente _tick_, cuando la cola sea vaciada. La mayor parte del tiempo no necesitamos preocuparnos por esto pero puede ser un tanto engorroso cuando intentas hacer algo que depende del estado del DOM luego de la actualización. Aunque Vue generalmente alienta a los desarrolladores a pensar de una forma "orientada a los datos" y evitar manipular el DOM directamente, a veces es puede ser necesario ensuciarse las manos. Para esperar a que Vue haya finalizado de actualizar el DOM luego de un cambio en los datos, puedes utilizar `Vue.nextTick(callback)` inmediatamente después de que hayan sido modificados. La _callback_ será ejecutada luego de que haya sido actualizado en DOM. Por ejemplo:

``` html
<div id="example">{{ message }}</div>
```

``` js
var vm = new Vue({
  el: '#example',
  data: {
    message: '123'
  }
})
vm.message = 'new message' // cambia los datos
vm.$el.textContent === 'new message' // falso
Vue.nextTick(function () {
  vm.$el.textContent === 'new message' // verdadero
})
```

Existe también el método de instancia `vm.$nextTick()`, el cual es especialmente útil dentro de componentes, porque no necesita la variable global `Vue` y el contexto `this` de la _callback_ estará enlazado con la instancia Vue actual:

``` js
Vue.component('example', {
  template: '<span>{{ message }}</span>',
  data: function () {
    return {
      message: 'not updated'
    }
  },
  methods: {
    updateMessage: function () {
      this.message = 'updated'
      console.log(this.$el.textContent) // => 'not updated'
      this.$nextTick(function () {
        console.log(this.$el.textContent) // => 'updated'
      })
    }
  }
})
```
