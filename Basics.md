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
<!-- We can also use the index of the current element -->
<ul>
    <li v-for="(element, index) in elements" :key="index"> 
      {{ index + ': ' + element.name }} 
    </li>
</ul>
```

## v-on

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