# COMPONENT REGISTRATION

* Components must be registered before the Vue instance is instantiated

## Global registration

* they can be used in the template of any root Vue instance (new Vue) created after registration, as well as in all subcomponents, meaning they also will be available inside each other

```js
Vue.component('component-a', { /* options */ })
Vue.component('component-b', { /* options */ })
Vue.component('component-c', { /* options */ })

new Vue({ el: '#app' })
```
```html
<div id="app">
  <component-a></component-a>
  <component-b></component-b>
  <component-c></component-c>
</div>
```

## Local registration

* you can define your components as plain JavaScript objects, then define the components you’d like to use in a `components` option of each component. This way you declare the components available for each component, and the locally registered component will be usable only where declared.

```js
var ComponentA = { /* options */ }
var ComponentB = {
  // to make ComponentA available in ComponentB:
  components: {
    'component-a': ComponentA
  },
  // ...options
}
new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```

## ES2015 modules alternative syntax

```js
import ComponentA from './ComponentA'
import ComponentC from './ComponentC'

export default {
  components: {
    ComponentA, // ComponentA: ComponentA
    ComponentC // ComponentC: ComponentC
  },
  // ...
}
```

## Automatic Global Registration of Base Components

* [link](https://vuejs.org/v2/guide/components-registration.html#Automatic-Global-Registration-of-Base-Components)