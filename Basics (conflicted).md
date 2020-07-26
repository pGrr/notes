# Installation

* [via npm or cdn](https://vuejs.org/v2/guide/installation.html)

# Introduction

* Vue is a progressive framework for building UIs and/or SPAs which is incrementally adoptable (thus has little size): the core is just the view, but you can plug-in a router, etc if you need them -[reference](https://vuejs.org/v2/guide/index.html#What-is-Vue-js)
* Provides two-way-data-binding (reactive automatic sync between the model (js code) and the view (html DOM)) by using the virtual DOM and declarative syntax - [reference](https://vuejs.org/v2/guide/index.html#Declarative-Rendering)
* Provides a way to express conditionals and loops - [reference](https://vuejs.org/v2/guide/index.html#Conditionals-and-Loops)
* Provides an easy way to update the UI in response to events (e.g. user input, etc) - [reference](https://vuejs.org/v2/guide/index.html#Handling-User-Input)
* Provides an easy way to realize re-usable custom html elements (components) - [reference](https://vuejs.org/v2/guide/index.html#Relation-to-Custom-Elements)
* Provides an easy way to organize components into an hierarchy tree and pass data (props = property of the component which is passed as argument) from the top to the bottom of the tree - [reference](https://vuejs.org/v2/guide/index.html#Composing-with-Components) 

# The Vue instance

* The Vue instance is "the heart" of the application: it is the DOM element which Vue processes and uses to render the application. It is often referred as `vm` (view-model, though vue is not actually a MVVM pattern, it is heavily inspired by it) - [reference](https://vuejs.org/v2/guide/instance.html#Creating-a-Vue-Instance)
* It takes an `optionsObject`, which most important parts are:
  * `el` - the selector of the DOM element to which the Vue instance should bind
  * `data` - the data bound to the view instance
    * it's content is usable inside the bound DOM element using the "moustache" syntax: `{{ myVariable }}` (references `app.data.myVariable`)
* Provides __two-way-data-binding__ between the model and the view: when the bound `data` javascript object (the model) changes it's state, the DOM (the view) is updated reactively

```js
// js (the model)
var app = new Vue({
    el: '#app', 
    data: {
        product: 'Socks',
    }
});
```
```html
<!-- html (the view) -->
<div id="app">
  <h1>{{ product }}</h1>
</div>
```

# v-* - Directives

It is a special syntax usable inside the DOM element bound to the Vue instance, which is processed by Vue.

## v-bind = :

* Binds an html element's attribute to a variable of the model (i.e. the bound `data`): you can make any property a variable (href, title, class, style, etc)
  * e.g. `<img v-bind:src="image">` equals to `<img src="{{ image }}">`
  * short syntax is just `:`:

```html
<!-- normal syntax -->
<img v-bind:src="image">

<!-- shortcut syntax -->
<img :src="image"> 
```

## v-on = @

* you can set event listeners with `v-on:event="..."` or `@event="..."`
  * common events are e.g. `click`, `submit`, `mouseover`, `keyup`, etc
* you can set event modifiers with `.` e.g. 
  * `@keyup.enter="..."` -  will listen for keyup event on enter key
  * `@submit.prevent="..."` - wil listen for submit, preventing the default form submission
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

```html
<!-- Loops over the elements variable (create multiple li) -->
<ul>
    <li v-for="element in elements" :key="element.id"> {{ element.name }} </li>
</ul>

<!-- We can also use the index of the current element -->
<ul>
    <li v-for="(element, index) in elements" :key="index"> 
      {{ index + ': ' + element.name }} 
    </li>
</ul>
```

* __When using v-for it is recommended to assign each rendered element its own unique key with `v-bind:key` (`:key`). Don’t use non-primitive values like objects and arrays as v-for keys. Use string or numeric values instead.__
  * When Vue is updating a list of elements rendered with v-for, by default it uses an “in-place patch” strategy. If the order of the data items has changed, instead of moving the DOM elements to match the order of the items, Vue will patch each element in-place and make sure it reflects what should be rendered at that particular index
  * This default mode is efficient, but only suitable when your list render output does not rely on child component state or temporary DOM state (e.g. form input values)
  * To give Vue a hint so that it can track each node’s identity, and thus reuse and reorder existing elements, you need to provide a unique key attribute for each item
  * It is recommended to provide a key attribute with v-for whenever possible, unless the iterated DOM content is simple, or you are intentionally relying on the default behavior for performance gains


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

* In-template expressions are very convenient, but they are meant for simple operations. Putting too much logic in your templates can make them bloated and hard to maintain
* Computed properties are variables to be derived from other data variables, which are
  * cached (not evaluated at every call, thus preferrable to methods)
  * reactively re-evaluated every time the original variable changes

```js
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // a computed getter
    reversedMessage: function () {
      // `this` points to the vm instance
      return this.message.split('').reverse().join('')
    }
  }
})
```

# Components

* components are __reusable and nestable__: an app can be composed by using many components
* components have a name and and an options object, which is nearly identical to the vue instance one
  * __it has a `template` property__ (instead of an `el` as it was in vue instance), which specifies the structure of the component and should have one and only root element
  * __the `data` is not an object, it's a function that returns a fresh data object__ (if it were an object and the component was reused each copy of the component would be using the same data object)
  * `methods`, `computed`, etc are the same (they were already functions)
* each component has an __isolated scope__, so it can't reach for properties of other components
* __`props` are custom attributes for passing data into a component from parents or from the root vue instance__ (kind of like a method arguments)
  * expected props __must be declared__
  * then they __can be used inside the template__ (kind of like `data` variables in the vue instance are used in the `el`)
* components are used:
  * __custom html elements__ (where the component's name is the tag name) 
  * __inside the view instance's 'el' element__ 
  * __passing props as attributes__

```js
Vue.component('product', {
    props: [message]
    template: '<h1> {{ message }} </h1',
    data: () => {
        // ...
    }
});
```
```html
<product message="hello!"></product>
```

* it is a best practice to specify props requirements with using built-in props validation
* this is done by using a props object instead of an array of names

```js
Vue.component('product', {
    props: {
        message: {
            type: String,
            required: true,
            default: 'Hi!',
        }
    }
    template: '<h1> {{ message }} </h1',
    data: () => {
        // ...
    }
});
```

## Communicating events

* sometime nested components need to pass data to another component higher in the hierarchy (inverse than props)
* this is achieved by emitting an event, through the `$.emit('event-name', ...args)` called inside a method
* on the component html we listen for the event using `v-on:...` (`@...`) and specifying a method as action
* the action will be called on the corresponding view instance methods, and all arguments of the events will be automatically passed to that method

```js
// component (child)
Vue.component('product', {
    data: {
        productId: 123,
    }
    methods: {
        addToCart: () => {
            $.emit('add-to-cart', this.productId);
        }
    }
})
// vue instance (parent)
var app = new Vue({
    el: 'app',
    data: {
        cart: [],
    },
    methods: {
        updateCart: (id) => {
            this.cart.push(id);
        }
    }
});
```
```html
<div id="app">
    <product @add-to-cart="updateCart"></product>
</div>
```

# Two way data binding

* Since now we've seen one-way data binding: from the model (component or view instance) to the view (html)
* Sometimes we want __two-way data binding__, e.g. in a form input: we want the model-value to change whenever the user changes input
* `v-model="..."` lets you do that
  * `v-model.number="..."` - automatic type-cast to number

```js
Vue.component('review', {
    template: `
        <input v-model="name">
    `
    data: () => {
        name: null
    }
});
```

# Event bus

* sometimes you want a global event bus to communicate events to all hierarchy level of the application (e.g. a grand-child component emitting an event that must be listened by the grand-parent)
* the event bus is a new Vue instance, that we will emit events to with `eventBus.$emit('name', ...args)`
* the listener listen on the eventBus with `eventBus.$on('name', (...args) => {...})`;
* __As your applications grows, you want to rather use vue's own state management solution: vuex__.

```js
var eventBus = new Vue();

// on our component, we call the emit event on the eventbus
eventBus.$emit('review-submitted', productReview)

// the listener
eventBus.$on('review-submitted', (productReview) => {
    // ...
});
```