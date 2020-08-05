# SINGLE-FILE-COMPONENTS

* In many Vue projects, global components will be defined using `Vue.component`, followed by `new Vue({ el: '#container' })` to target a container element in the body of every page. This can work very well for small to medium-sized projects, where JavaScript is only used to enhance certain views. In more complex projects however, single-file components provides a better code organization
* Single file components are components defined in a file with a `.vue` extension, made possible with build tools such as Webpack or Browserify.
* Single file components have 3 html elements: 
  * __template__: skeleton, structure of the component
  * __script__: brain, logic and behaviour
  * __style__: look and feel
    * styles are __scoped to the component__ (a unique identifier will be added to the component in order apply styles only to it, not affecting global styles)
    * since also the root component `src/App.vue` is a component, styles there will be global (scoped to the entire app)    
    * many other way to bind a global style exists, [see more](https://badacadabra.github.io/Using-global-style-rules-in-a-Vue-js-app/)
* you can use any language, by setting the `lang=...` attribute (e.g. `pug`, `babel`, `typescript`, `sass`) by installing as a dependency the appropriate libraries (e.g. for sass is `sass-loader` and `node-sass`) (see vue loader)

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
