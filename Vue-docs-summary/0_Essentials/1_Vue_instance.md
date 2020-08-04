# THE VUE INSTANCE

* The Vue instance is "the heart" of the application: it is the DOM element which Vue processes and uses to render the application. - [more](https://vuejs.org/v2/guide/instance.html#Creating-a-Vue-Instance)
* It is often referred as `vm` (view-model, though vue is not actually a MVVM pattern, it is heavily inspired by it)
* The Vue instance is formally the root component of the application, but each component can be considered a vue instance as well and use the same API

```js
var vm = new Vue({
  // options
})
```

## Instance's properties and methods - `el`, `data`, `vm.$*`

* The Vue instance takes an `optionsObject`, which most important parts are: - [more](https://vuejs.org/v2/guide/instance.html#Data-and-Methods)
  * `el` - the selector of the DOM element to which the Vue instance should bind
  * `data` - the data bound to the view instance
    * it's content is usable inside the bound DOM element using the "moustache" syntax: `{{ myVariable }}` (references `app.data.myVariable`)
    * Provides __two-way-data-binding__ between the model and the view: when the bound `data` javascript object (the model) changes it's state, the DOM (the view) is updated reactively
    * Properties in data are only reactive if they existed when the instance was created. If you know you’ll need a property later, but it starts out empty or non-existent, you’ll need to set some initial value
* Vue instances expose a number of useful instance properties and methods prefixed with `$` to differentiate them from user-defined ones: - [more](https://vuejs.org/v2/guide/instance.html#Data-and-Methods)


```js
// js (the model)
var vm = new Vue({
    el: '#app', 
    data: {
        myVariable: 'Socks',
    }
});
```
```html
<!-- html (the view) -->
<div id="app">
  <h1>{{ product }}</h1>
</div>
```
```js
// Accessing properties and methods
vm.$data === data // => true
vm.$el === document.getElementById('example') // => true

// $watch is an instance method
vm.$watch('a', function (newValue, oldValue) {
  // This callback will be called when `vm.a` changes
})
```

## Instance lifecycle hooks

* The Vue instance has a lifecycle ([more](https://vuejs.org/v2/guide/instance.html#Lifecycle-Diagram)) and hooks are provided to execute custom functions at appropriate times (`created`, `mounted`, `destroyed`, etc) - [more](https://vuejs.org/v2/guide/instance.html#Instance-Lifecycle-Hooks)

```js
new Vue({
  data: {
    a: 1
  },
  created: function () {
    // `this` points to the vm instance
    console.log('a is: ' + this.a)
  }
})
// => "a is: 1"
```
