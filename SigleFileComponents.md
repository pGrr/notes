# Vue single file components

* 3 sections: 
  * __template__: skeleton, structure of the component
  * __script__: brain, logic and behaviour
  * __style__: look and feel
    * styles are __scoped to the component__ (a unique identifier will be added to the component in order apply styles only to it, not affecting global styles)
    * since also the root component `src/App.vue` is a component, styles there will be global (scoped to the entire app)    
    * many other way to bind a global style exists, [see more](https://badacadabra.github.io/Using-global-style-rules-in-a-Vue-js-app/)
* you can use any language, by setting the `lang=...` attribute (e.g. pug, typescript, sass) by installing as a dependency the appropriate libraries (e.g. for sass is `sass-loader` and `node-sass`) (see vue loader)

```html
<template>
    <div>
        <h1>{{ title }}</h1>
        <NestedComponent/>
    </div>
</template>

<script>
    import NestedComponent from 'src/components/NestedComponent.vue';
    export default {
        components: {
            NestedComponent: NestedComponent,
        }
        data: () => {
            return {
                title: 'My Title',
            }
        }
    }
</script>

<style scoped>
    h1 {
        color: green;
    }
</style>
```

# Global components

* if some components must be used in all other components, there is a way to avoid importing it in every component
  * Base components, e.g. `BaseIcon.vue`, `BaseButton.vue`, etc
  * Thouse components can be re-used and customized injecting props on them
* In order to make a component global, it must be imported and registered in `main.js`
* if you have many components, you may consider [Automatic global registration](https://vuejs.org/v2/guide/components-registration.html#Automatic-Global-Registration-of-Base-Components)

```js
// main.js
import BaseIcon from src/components/BaseIcon.vue;
Vue.component('BaseIcon', BaseIcon); // must be above "new Vue"
```

# Slots

* Slots are used to make the content of a component dynamic
* dynamic content is wrapped into `<slot>` tags.
  * the content of `<slot>` tags will be the default value
  * if the parent puts any other content inside the child's tags, that will substitute the slot content
  * if the template code that replaces the slot content need to access the child's data, see __scoped-slots__

```js
// Form.vue
<template>
    <div>
        <BaseButton>CUSTOM CONTENT WITH ${{ variable }}</BaseButton>
    </div>
</template>
// BaseButton.vue
<template>
    <div>
        <button><slot>DEFAULT</slot></button>
    </div>
</template>
```
