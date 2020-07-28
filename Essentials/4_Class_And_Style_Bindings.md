# CLASS AND STYLE BINDINGS

## Binding HTML classes

* we can use `v-bind` to handle an element's classes and inline styles - [more](https://vuejs.org/v2/guide/class-and-style.html#Binding-HTML-Classes)
* To avoid meddling with string concatenation Vue provides special enhancements when v-bind is used with class and style: in addition to strings, the expressions can also evaluate to objects or arrays
* When you use the class attribute on a custom component, those classes will be added to the component’s root element. Existing classes on this element will not be overwritten - [more](https://vuejs.org/v2/guide/class-and-style.html#With-Components)

```html
<!-- the presence of the active class will be determined by the data property isActive -->
<div v-bind:class="{ active: isActive }"></div>

<!-- v-bind:class can coexist with class and have multiple properties -->
<div
  class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"
></div>

<!-- and can be defined as object of data: -->
<div v-bind:class="classObject"></div>
```
```js
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```

```html
<!-- or as a computed property -->
<div v-bind:class="classObject"></div>
```
```js
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

```html
<!-- Multiple classes can be specified with the array syntax -->
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
<!-- and you can combine it with the object syntax (same as above, but less verbose) -->
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```
```js
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

## Binding inline styles

* The object syntax for v-bind:style is pretty straightforward - it looks almost like CSS, except it’s a JavaScript object. You can use either camelCase or kebab-case (use quotes with kebab-case) for the CSS property names - [more](https://vuejs.org/v2/guide/class-and-style.html#Binding-Inline-Styles)
* Note: When you use a CSS property that requires vendor prefixes in v-bind:style, for example transform, Vue will automatically detect and add appropriate prefixes to the applied styles.

```html
<div v-bind:style="styleObject"></div>
```
```js
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
} // you can as well computed properties
```

```html
<!-- you can apply multiple style objects to the same element with the array syntax -->
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```