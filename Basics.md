# Install

* via npm or cdn

# The View instance

* The vue instance is "the heart" of the application
  * `new Vue(optionsObject)`
  * it binds to the element `el`
  * on that element, any referenced `data` variable (e.g. `{{ product }}`, which references `app.data.product`) is changed reactively

```js
//js
var app = new Vue({
    el: '#app',
    data: {
        product: 'Socks',
    }
});
// html
```
```html
<h1 id="app">{{ product }}</h1>
```

# v-bind

* you can bind an html element attribute to a vue's data variable with `v-bind`
  * e.g. `<img v-bind:src="image">` equals to `<img src="{{ image }}">`
* short syntax is just `:`:
  * `<img :src="image">`
* Using v-bind you can make any property a variable (href, title, class, style, etc)

```html
<!-- normal syntax -->
<img v-bind:src="image">
<!-- shortcut syntax -->
<img :src="image"> 
```

# v-x directives

## v-if v-elseif v-else

```html
<!-- add or remove to the dom conditionally one or the other basing on inventory variable -->
<p v-if="inventory > 10">In stock</p>
<p v-else-if="inventory <= 10 && inventory > 0">Almost sold out!</p>
<p v-else>Not available</p>
```

## v-show

```html
<!-- v-show toggle hidden property basing on the condition value -->
<p v-show="inStock">In stock.</p>
```

## v-for 

* When using v-for it is recommended to give each rendered element its own unique key, and thus helping vue to diffs the list in order to reuse and reorder existing elements

```html
<!-- Loops over the elements variable (create multiple li) -->
<ul>
    <li v-for="element in elements" :key="element.id"> {{ element.name }} </li>
</ul>
<!-- We can also have the index of the current element, if needed -->
<ul>
    <li v-for="(element, index) in elements" :key="element.id"> 
      {{ index + ': ' + element.name }} 
    </li>
</ul>
```

## v-on

* you can set event listeners with `v-on:event="..."` or `@event="..."`
  * common events are e.g. `click`, `submit`, `mouseover`, `keyup`, etc
* you can set event modifiers with `.`, e.g. `keyup.enter` will listen for keyup event on enter key
* you can set an expression using `data` variables or a `methods` function
  * if you pass arguments to the function, they must match with the method

```html
<div id="app">
  <!-- on button click, execute the expression -->
  <button v-on:click="cart += 1" 
      :disabled="!notInStock" 
      :class="{ disabledButton: !inStock }">
  <!-- or trigger a function -->
  <button v-on:click="addToCart" 
      :disabled="!notInStock" 
      :class="{ disabledButton: !inStock }">
  <!-- or with shortcut syntax -->
  <button @click="addToCart" 
      :disabled="!notInStock" 
      :class="{ disabledButton: !inStock }">
</div>
```
```js
var app = new Vue({
  el: '#app',
  data: {
    cart: 0,
    notInStock: false,
  },
  methods: {
    addToCart: () => {
      cart += 1;
    }
  }
});
```

# Styles

## Inline styles

```html
<!-- styles can be provided as javascript object -->
<p :style="{ fontSize: fontSize }"></p>
<!-- or you can use standard syntax, but using quotes -->
<p :style="{ 'font-size': fontSize }"></p>
<!-- or bind to a style object for multiple properties -->
<p :style="styleObject"></p>
<!-- or bind to multiple style objects (as js array) -->
<p :style=" [ styleObject, styleObject2 ] "></p>
```
```js
var app = new Vue({
  el: '#app',
  data: {
    fontSize: '13px',
    styleObject: {
      fontSize: '13px',
      color: 'black';
    },
    styleObject2: {
      'margin-bottom': '5px',
      marginTop: '3px',
      'margin-bottom': '2px',

    }
  },
});
```

## Classes binding

```html
<!-- classes can be binded and changed dinamically -->
<button :class="{ active: activeClass, 'text-danger': errorClass }">
<!-- we can bind objects -->
<button :class="classObject">
<!-- or arrays -->
<button :class="[ classes.activeClass, classes.errorClass ]">
<!-- or can be evaluated with expressions -->
<button :class="isActive ? classes.activeClass : '' ">
```
```js
var app = new Vue({
  el: '#app',
  data: {
    isActive: true,
    activeClass: true,
    errorClass: false,
    classObject: {
      active: true,
      'text-danger': false,
    }
    classes: {
      activeClass: 'active',
      errorClass: 'text-danger',
    }
  },
});
```

# Computed properties

* variables to be derived from other data variables can be declared as methods inside the `computed` key object
* obviously, whenever one of the derived variables changes, the computed property changes too, reactively. But until one of the dependency changes, the computed property is __cached (not evaluated at every call)__ thus is preferrable to `methods` whenever applicable

```js
var app = new Vue({
  el: '#app',
  data: {
    title: 'My title',
    product: 'My product',
  },
  computed: {
    title: () => {
        return this.title + ' ' + this.product;
    }
  }
});
```
