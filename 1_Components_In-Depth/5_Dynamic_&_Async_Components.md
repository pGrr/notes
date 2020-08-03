# keep-alive WITH DYNAMIC COMPONENTS

* we used the `is` attribute to switch dynamically components in a tabbed interface

```html
<component v-bind:is="currentTabComponent"></component>
```

* When switching between these components though, you’ll sometimes want to maintain their state or avoid re-rendering for performance reasons
* To solve this problem and mantain the state after the first instantiation, even if the component is not rendered anymore, we can wrap it with a `<keep-alive>` element:

```html
<!-- Inactive components will be cached! -->
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

# ASYNC COMPONENTS

* In large applications, we may need to divide the app into smaller chunks and only load a component from the server when it’s needed. To make that easier, Vue allows you to define your component as a factory function that asynchronously resolves your component definition. Vue will only trigger the factory function when the component needs to be rendered and will cache the result for future re-renders.
    * As you can see, the factory function receives a `resolve` callback, which should be called when you have retrieved your component definition from the server. You can also call `reject(reason)` to indicate the load has failed.
* This can be useful for dynamic imports - [more](https://vuejs.org/v2/guide/components-dynamic-async.html#Async-Components)

```js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // Pass the component definition to the resolve callback
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

## Handling Loading State

* The async component factory can also return an object of the following format:

```js
const AsyncComponent = () => ({
  // The component to load (should be a Promise)
  component: import('./MyComponent.vue'),
  // A component to use while the async component is loading
  loading: LoadingComponent,
  // A component to use if the load fails
  error: ErrorComponent,
  // Delay before showing the loading component. Default: 200ms.
  delay: 200,
  // The error component will be displayed if a timeout is
  // provided and exceeded. Default: Infinity.
  timeout: 3000
})
```

