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
* this is achieved by emitting an event, through the `$.emit('event-name')` called inside a method
* on the component html we listen for the event using `v-on` and we call a method as action
* the action's method will be called on the view instance, the root parent (it must be one of its `methods`)

```js
// component (child)
Vue.component('product', {
    // ...
    methods: {
        addToCart: () => {
            $.emit('add-to-cart');
        }
    }
})
// vue instance (parent)
var app = new Vue({
    el: 'app',
    data: {
        cart: 0,
    },
    methods: {
        updateCart: () => {
            this.cart += 1;
        }
    }
});
// html
```
```html
<div id="app">
    <product @add-to-cart="updateCart"></product>
</div>
```