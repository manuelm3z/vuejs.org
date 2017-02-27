---
title: Propiedades computadas y observadores
type: guia
order: 5
---

## Propiedades computadas

Las expresiones dentro de las plantillas son muy cómodas, pero están pensadas solo para operaciones sencillas. Agregar demasiada lógica en tus plantillas puede hacerlas engorrosas y difíciles de mantener. Por ejemplo:

``` html
<div id="example">
  {{ message.split('').reverse().join('') }}
</div>
```

En este punto, la plantilla ya no es tan sencilla y declarativa. Tienes que observarla durante un momento antes de entender que muestra el valor de `mesagge` invertido y el problema se agrava cuando quieres incluir el mismo mensaje en más de un lugar dentro de tu plantilla.

Es por esto que para cualquier lógica compleja, deberías utilizar una **propiedad computada**.

### Ejemplo básico

``` html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```

``` js
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // un getter computado
    reversedMessage: function () {
      // `this` apunta a la instancia de vm
      return this.message.split('').reverse().join('')
    }
  }
})
```

Resultado:

{% raw %}
<div id="example" class="demo">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
<script>
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    reversedMessage: function () {
      return this.message.split('').reverse().join('')
    }
  }
})
</script>
{% endraw %}

Aquí hemos declarado una propiedad computada: `reversedMessage`. La función que codificamos será usada como función _getter_ para la propiedad `vm.reversedMessage`:

``` js
console.log(vm.reversedMessage) // -> 'olleH'
vm.message = 'Goodbye'
console.log(vm.reversedMessage) // -> 'eybdooG'
```

Puedes abrir la consola y jugar con el ejemplo. El valor de `vm.reversedMessage` siempre depende del valor de `vm.message`.

También puedes crear enlaces de datos a propiedades computadas en las plantillas como harías con una propiedad normal. Vue está al tanto que `vm.reversedMessage` depende de `vm.message`, por lo que actualizará cualquier enlace que dependa de `vm.reversedMessage` cuando `vm.message` cambie. La mejor parte es que hemos creado esta relación de dependencias declarativamente: la función _getter_ computada no tiene efectos secundarios, lo cual hace sencillo entenderla y probarla.

### Cacheo computado vs métodos

Puede que hayas notado que podemos lograr el mismo resultado invocando un método en la expresión:

``` html
<p>Reversed message: "{{ reverseMessage() }}"</p>
```

``` js
// dentro del componente
methods: {
  reverseMessage: function () {
    return this.message.split('').reverse().join('')
  }
}
```

En lugar de una propiedad computada, podemos definir la misma función como un método. Desde el punto de vista del resultado, ambos enfoques son exactamente iguales. Sin embargo, la diferencia está en que las **propiedades computadas son guardadas en memoria caché basándose en sus dependencias**. Una propiedad computada solo se reevaluará cuando alguna de sus dependencias haya cambiado. Esto significa que mientras `message` no haya cambiado, acceder reiteradamente a la propiedad computada `reversedMessage` devolverá inmediatamente el resultado computado previamente sin tener que ejecutar la función de nuevo.

Esto también significa que la siguiente propiedad computada nunca se actualizará, porque `Date.now()` no es una dependencia reactiva:

``` js
computed: {
  now: function () {
    return Date.now()
  }
}
```

En comparación, la invocación a un método **siempre** ejecutará la función cada vez que se produzca un renderizado.

¿Por qué necesitamos guardar en caché? Imagina que tenemos una propiedad computada **A** costosa en términos de rendimiento, la cual requiere iterar a través de un vector enorme y hacer muchos cálculos. Además, podemos tener otras propiedades computadas que dependan de **A**. Sin este guardado en memoria caché, ¡estaríamos ejecutando la función _getter_ de **A** muchas más veces de las necesarias! En los casos donde no requieras guardar en caché, utiliza un método.

### Propiedades computadas vs propiedades observadas

Vue provee una forma más genérica de observar cambios de datos en una instancia y de reaccionar frente a ellos con : **observar propiedades**. Cuando tienes algunos datos que deben cambiar basándose en otros datos, es tentador utilizar excesivamente `watch` - especialmente si tienes experiencia con AngularJS. Por lo general, es una mejor idea utilizar una propiedad computada en lugar de una llamada imperativa a la función _callback_ `watch`. Por ejemplo:

``` html
<div id="demo">{{ fullName }}</div>
```

``` js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar',
    fullName: 'Foo Bar'
  },
  watch: {
    firstName: function (val) {
      this.fullName = val + ' ' + this.lastName
    },
    lastName: function (val) {
      this.fullName = this.firstName + ' ' + val
    }
  }
})
```

El código anterior es imperativo y repetitivo. Compáralo con la versión de propiedades computadas:

``` js
var vm = new Vue({
  el: '#demo',
  data: {
    firstName: 'Foo',
    lastName: 'Bar'
  },
  computed: {
    fullName: function () {
      return this.firstName + ' ' + this.lastName
    }
  }
})
```

Mucho mejor, ¿no?

### Función _setter_ computada

Las propiedades computadas solo tienen una función _getter_ por defecto, pero puedes agregarles una función _setter_ cuando lo necesites:

``` js
// ...
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
// ...
```

Ahora cuando ejecutes `vm.fullName = 'John Doe'`, la función _setter_ se ejecutará y `vm.firstName` y `vm.lastName` se actualizarán según corresponda.

## Observadores

Mientras que las propiedades computadas son más apropiadas en la mayoría de los casos, hay veces que es necesario un observador personalizado. Es por ello que Vue provee una forma más genérica de reaccionar a los cambios en los datos a través de la opción `watch`. Esto es útil por lo general cuando quieres realizar operaciones asíncronas o costosas en respuesta a esos cambios en los datos.

Por ejemplo:

``` html
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
```

``` html
<!-- Dado que ya existe un rico ecosistema de bibliotecas AJAX                       -->
<!-- y colecciones de métodos utilitarios de propósito general, el núcleo de Vue     -->
<!-- es capaz de mantenerse pequeño ya que no crea los suyos propios. Esto también   -->
<!-- te da la libertad de utilizar cualquier cosa con la que ya estés familiarizado  -->
<script src="https://unpkg.com/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://unpkg.com/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // cuando 'question' cambie, se ejecutará esta función
    question: function (newQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.getAnswer()
    }
  },
  methods: {
    // _.debounce es una función que se provee con lodash para limitar qué tan
    // seguido puede ejecutarse una operación particularmente costosa.
    // En este caso, queremos limitar las peticiones a 
    // yesno.wtf/api, esperando a que el usuario haya finalizado
    // de teclear antes de hacer la petición ajax. Para aprender 
    // más acerca de la función _.debounce (y su prima 
    // _.throttle), visita: https://lodash.com/docs#debounce
    getAnswer: _.debounce(
      function () {
        if (this.question.indexOf('?') === -1) {
          this.answer = 'Questions usually contain a question mark. ;-)'
          return
        }
        this.answer = 'Thinking...'
        var vm = this
        axios.get('https://yesno.wtf/api')
          .then(function (response) {
            vm.answer = _.capitalize(response.data.answer)
          })
          .catch(function (error) {
            vm.answer = 'Error! Could not reach the API. ' + error
          })
      },
      // Este es el número de milisegundos que esperamos
      // hasta que el usuario termine de teclear.
      500
    )
  }
})
</script>
```

Resultado:

{% raw %}
<div id="watch-example" class="demo">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>
<script src="https://unpkg.com/axios@0.12.0/dist/axios.min.js"></script>
<script src="https://unpkg.com/lodash@4.13.1/lodash.min.js"></script>
<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    question: function (newQuestion) {
      this.answer = 'Waiting for you to stop typing...'
      this.getAnswer()
    }
  },
  methods: {
    getAnswer: _.debounce(
      function () {
        var vm = this
        if (this.question.indexOf('?') === -1) {
          vm.answer = 'Questions usually contain a question mark. ;-)'
          return
        }
        vm.answer = 'Thinking...'
        axios.get('https://yesno.wtf/api')
          .then(function (response) {
            vm.answer = _.capitalize(response.data.answer)
          })
          .catch(function (error) {
            vm.answer = 'Error! Could not reach the API. ' + error
          })
      },
      500
    )
  }
})
</script>
{% endraw %}

En este caso, utilizar la opción `watch` nos permite realizar operaciones asíncronas (acceder a una API), limitando qué tan seguido ejecutamos esa operación, y establecer así estados intermedios hasta que tengamos una respuesta final. Nada de esto sería posible sin una propiedad computada.

Además de la opción `watch`, puedes utilizar la [API de vm.$watch](../api/#vm-watch).
