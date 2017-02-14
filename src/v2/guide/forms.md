---
title: Enlaces con campos de formulario
type: guia
order: 10
---

## Uso básico

Puedes utilizar la directiva `v-model` para crear enlaces de datos de dos vías en los campos de un formulario. Esta directiva elige automáticamente la forma correcta de actualizar el elemento basado en el tipo de campo. A pesar de parecer algo mágico, `v-model` es esencialmente azúcar sintáctico para actualizar datos debido a eventos de entrada de los usuarios, además de algunos cuidados para casos extremos.

<p class="tip">`v-model` descarta los valores iniciales provistos por los campos de un formulario. Siempre tratará a los datos en la instancia de Vue como fuente de verdad.</p>

<p class="tip" id="vmodel-ime-tip">Para lenguajes que requieran un [IME](https://en.wikipedia.org/wiki/Input_method) (Chinese, Japanese, Korean etc.), verás que `v-model` no se actualiza durante la composición IME. Si deseas realizar estas actualizaciones también, utiliza el evento `instead` en su lugar.</p>

### Texto

``` html
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```

{% raw %}
<div id="example-1" class="demo">
  <input v-model="message" placeholder="edit me">
  <p>Message is: {{ message }}</p>
</div>
<script>
new Vue({
  el: '#example-1',
  data: {
    message: ''
  }
})
</script>
{% endraw %}

### Texto multilínea

``` html
<span>Multiline message is:</span>
<p style="white-space: pre">{{ message }}</p>
<br>
<textarea v-model="message" placeholder="add multiple lines"></textarea>
```

{% raw %}
<div id="example-textarea" class="demo">
  <span>Multiline message is:</span>
  <p style="white-space: pre">{{ message }}</p>
  <br>
  <textarea v-model="message" placeholder="add multiple lines"></textarea>
</div>
<script>
new Vue({
  el: '#example-textarea',
  data: {
    message: ''
  }
})
</script>
{% endraw %}


{% raw %}
<p class="tip">La interpolación en textareas (<code>&lt;textarea&gt;{{text}}&lt;/textarea&gt;</code>) no funcionará. Utiliza <code>v-model</code> en su lugar.</p>
{% endraw %}

### Checkbox

Checkbox simple, valor booleano:

``` html
<input type="checkbox" id="checkbox" v-model="checked">
<label for="checkbox">{{ checked }}</label>
```
{% raw %}
<div id="example-2" class="demo">
  <input type="checkbox" id="checkbox" v-model="checked">
  <label for="checkbox">{{ checked }}</label>
</div>
<script>
new Vue({
  el: '#example-2',
  data: {
    checked: false
  }
})
</script>
{% endraw %}

Checkbox múltiples, enlazados al mismo arreglo:

``` html
<input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
<label for="jack">Jack</label>
<input type="checkbox" id="john" value="John" v-model="checkedNames">
<label for="john">John</label>
<input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
<label for="mike">Mike</label>
<br>
<span>Checked names: {{ checkedNames }}</span>
```

``` js
new Vue({
  el: '...',
  data: {
    checkedNames: []
  }
})
```

{% raw %}
<div id="example-3" class="demo">
  <input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
  <label for="jack">Jack</label>
  <input type="checkbox" id="john" value="John" v-model="checkedNames">
  <label for="john">John</label>
  <input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
  <label for="mike">Mike</label>
  <br>
  <span>Checked names: {{ checkedNames }}</span>
</div>
<script>
new Vue({
  el: '#example-3',
  data: {
    checkedNames: []
  }
})
</script>
{% endraw %}

### Radio


``` html
<input type="radio" id="one" value="One" v-model="picked">
<label for="one">One</label>
<br>
<input type="radio" id="two" value="Two" v-model="picked">
<label for="two">Two</label>
<br>
<span>Picked: {{ picked }}</span>
```
{% raw %}
<div id="example-4" class="demo">
  <input type="radio" id="one" value="One" v-model="picked">
  <label for="one">One</label>
  <br>
  <input type="radio" id="two" value="Two" v-model="picked">
  <label for="two">Two</label>
  <br>
  <span>Picked: {{ picked }}</span>
</div>
<script>
new Vue({
  el: '#example-4',
  data: {
    picked: ''
  }
})
</script>
{% endraw %}

### Select

Select simple:

``` html
<select v-model="selected">
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<span>Selected: {{ selected }}</span>
```
{% raw %}
<div id="example-5" class="demo">
  <select v-model="selected">
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <span>Selected: {{ selected }}</span>
</div>
<script>
new Vue({
  el: '#example-5',
  data: {
    selected: null
  }
})
</script>
{% endraw %}

Select múltiple (enlazado a un arreglo):

``` html
<select v-model="selected" multiple>
  <option>A</option>
  <option>B</option>
  <option>C</option>
</select>
<br>
<span>Selected: {{ selected }}</span>
```
{% raw %}
<div id="example-6" class="demo">
  <select v-model="selected" multiple style="width: 50px">
    <option>A</option>
    <option>B</option>
    <option>C</option>
  </select>
  <br>
  <span>Selected: {{ selected }}</span>
</div>
<script>
new Vue({
  el: '#example-6',
  data: {
    selected: []
  }
})
</script>
{% endraw %}

Opciones dinámicas renderizadas con `v-for`:

``` html
<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{ option.text }}
  </option>
</select>
<span>Selected: {{ selected }}</span>
```
``` js
new Vue({
  el: '...',
  data: {
    selected: 'A',
    options: [
      { text: 'One', value: 'A' },
      { text: 'Two', value: 'B' },
      { text: 'Three', value: 'C' }
    ]
  }
})
```
{% raw %}
<div id="example-7" class="demo">
  <select v-model="selected">
    <option v-for="option in options" v-bind:value="option.value">
      {{ option.text }}
    </option>
  </select>
  <span>Selected: {{ selected }}</span>
</div>
<script>
new Vue({
  el: '#example-7',
  data: {
    selected: 'A',
    options: [
      { text: 'One', value: 'A' },
      { text: 'Two', value: 'B' },
      { text: 'Three', value: 'C' }
    ]
  }
})
</script>
{% endraw %}

## Enlace de valores

Para _radio_, _checkbox_ y opciones de _select_, los valores del enlace de `v-model` son normalmente cadenas de texto estáticas (o booleanos en caso del _checkbox_):

``` html
<!-- `picked` es la cadena de texto "a" cuando se tilda -->
<input type="radio" v-model="picked" value="a">

<!-- `toggle` es 'true' o 'false' -->
<input type="checkbox" v-model="toggle">

<!-- `selected` es la cadena de texto "abc" cuando se selecciona -->
<select v-model="selected">
  <option value="abc">ABC</option>
</select>
```

Pero a veces podemos querer enlazar el valor a una propiedad dinámica en la instancia de Vue. Podemos utilizar `v-bind` para lograr esto. Además, utilizar `v-bind` nos permite enlazar el valor del campo a valores de otros tipos:

### Checkbox

``` html
<input
  type="checkbox"
  v-model="toggle"
  v-bind:true-value="a"
  v-bind:false-value="b"
>
```

``` js
// Cuando se tilda:
vm.toggle === vm.a
// cuando se destila:
vm.toggle === vm.b
```

### Radio

``` html
<input type="radio" v-model="pick" v-bind:value="a">
```

``` js
// cuando se tilda:
vm.pick === vm.a
```

### opciones de Select

``` html
<select v-model="selected">
  <!-- objeto literal en línea -->
  <option v-bind:value="{ number: 123 }">123</option>
</select>
```

``` js
// cuando se selecciona:
typeof vm.selected // -> 'object'
vm.selected.number // -> 123
```

## Modificadores

### `.lazy`

Por defecto, `v-model` sincroniza el campo con los datos luego de cada evento `input` (con la excepción de las composiciones IME [como se mencionó anteriormente](#vmodel-ime-tip)). Puedes agregar el modificador `lazy` para sincronizar luego de eventos `change` en su lugar:

``` html
<!-- sincronizado luego de "change" en lugar de "input" -->
<input v-model.lazy="msg" >
```

### `.number`

Si deseas que un campo de usuario sea convertido automáticamente a un número, puedes agregar el modificador `number` a tus campos controlados por `v-model`:

``` html
<input v-model.number="age" type="number">
```

Esto es útil, debido a que incluso con `type="number"`, el valor de los elementos HTML `<ìnput>` siempre devuelven una cadena de texto.

### `.trim`

Si deseas que a los datos de usuario se le aplique `trim` automáticamente, puedes agregar el modificador `trim` a tus campos controlados por `v-model`:

```html
<input v-model.trim="msg">
```

## `v-model` con componentes

> Si todavía no estas familiarizado con los componentes de Vue, saltea esto por ahora.

Los tipos de campos de formularios nativos de HTML no siempre se ajustarán a tus necesidades. Afortunadamente, los componentes de Vue te permiten construir campos de entrada reutilizables con comportamiento completamente personalizado. ¡Estos campos de entrada incluso funcionan con `v-model`! Para aprender más, lee acerca de los [campos de entrada personalizados](components.html#Form-Input-Components-using-Custom-Events) en la guia de componentes.







