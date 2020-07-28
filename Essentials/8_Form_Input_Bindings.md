# FORM INPUT BINDINGS

* You can use the v-model directive to create two-way data bindings on form input
* It automatically picks the correct way to update the element based on the input type
* v-model internally uses different properties and emits different events for different input elements:
  * text and textarea elements use `value` property and `input` event;
  * checkboxes and radiobuttons use `checked` property and `change` event;
  * select fields use `value` as a prop and `change` as an event.
* `v-model` will ignore the initial `value`, `checked`, or `selected` attributes found on any form elements. It will always treat the Vue instance data as the source of truth. You should declare the initial value on the JavaScript side, inside the data option of your component.
* [more](https://vuejs.org/v2/guide/forms.html)

```html
<input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
<label for="jack">Jack</label>
<input type="checkbox" id="john" value="John" v-model="checkedNames">
<label for="john">John</label>
<input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
<label for="mike">Mike</label>
<br>
<span>Checked names: {{ checkedNames }}</span>
```
```js
new Vue({
  el: '...',
  data: {
    checkedNames: []
  }
})
```

```html
<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{ option.text }}
  </option>
</select>
<span>Selected: {{ selected }}</span>
```
```js
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

## Value Bindings

* For radio, checkbox and select options, the v-model binding values are usually static strings (or booleans for checkboxes)
* But sometimes, we may want to bind the value to a dynamic property on the Vue instance. We can use `v-bind` to achieve that. In addition, using `v-bind` allows us to bind the input value to non-string values
* [more](https://vuejs.org/v2/guide/forms.html#Value-Bindings)

```html
<!-- `picked` is a string "a" when checked -->
<input type="radio" v-model="picked" value="a">

<!-- `picked` is the value of the variable a when checked -->
<input type="radio" v-model="picked" v-bind:value="a">
```

* With checkboxes, `true-value` and `false-value` can be used to define the true and false values directly in the template. They don’t affect the input’s value attribute, because browsers don’t include unchecked boxes in form submissions. To guarantee that one of two values is submitted in a form (i.e. “yes” or “no”), use radio inputs instead.

```html
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>
```

## Modifiers

```html
<!-- synced after "change" instead of "input" -->
<input v-model.lazy="msg">

<!-- input to be automatically typecast as a Number (even with type="number", the value of HTML input elements always returns a string) -->
<input v-model.number="age" type="number">

<!-- whitespace to be trimmed automatically -->
<input v-model.trim="msg">
```

## v-model with Components

* HTML’s built-in input types won’t always meet your needs. Vue components allow you to build reusable inputs with completely customized behavior. These inputs even work with `v-model`! - [more](https://vuejs.org/v2/guide/components.html#Using-v-model-on-Components)

