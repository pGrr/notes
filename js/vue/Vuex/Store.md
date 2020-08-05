# Store

* it's basically __a container that holds your application state__ and it's the central concept of Vuex applications
* It's different than a plain global object because:
  * __Stores are reactive__
    * exactly as the vue instance's data: when one component updates the state, the global state changes reactively and vice-versa the components update automatically when the global state changes
  * __You cannot directly mutate the store's state__.
    * The only way to change a store's state is by explicitly __committing mutations__. 
    * This is for leaving a trackable record, enabling debug tooling (such as log every mutation, take state snapshots, or even perform time travel debugging) - see vue dev tools

## Store structure

* similar to the View instance
  * `state` handle the state reactively
    * similar to Vue instance's `data`
  * `actions` update the vuex state
    * similar to Vue instance's `methods` that updates data
  * `getters` access the state
    * similar to Vue instance's `computed` properties, that access the data
  * `mutations` commit and track state changes
    * there is __no similar thing in Vue instance__
* Best practice is let __actions call mutations which update state directly__
  * using vue dev tools you can do time-travel debugging and rollback mutations to revert the state to its previous value

```js
const store = new Vuex.Store({
    state: {

    },
    actions: {

    },
    getters: {

    },
    mutations: {

    }
});
```

## Store usage inside components

* Using store state in a component simply involves returning the state within a computed property, because the store state is reactive.
* Triggering changes simply means committing mutations in component methods.
* Vuex provides a mechanism to "inject" the store into all child components from the root component with the store option (enabled by `Vue.use(Vuex)`)
  * By providing the `store` option to the root instance, the store will be injected into all child components of the root and will be available on them as `this.$store`

```js
const app = new Vue({
  el: '#app',
  // provide the store using the "store" option.
  // this will inject the store instance to all child components.
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter></counter>
    </div>
  `
})

// inside the component can be accessed with this.$store
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}

// The mapState Helper
// in full builds helpers are exposed as Vuex.mapState
import { mapState } from 'vuex'

export default {
  // ...
  computed: mapState({
    // arrow functions can make the code very succinct!
    count: state => state.count,

    // passing the string value 'count' is same as `state => state.count`
    countAlias: 'count',

    // to access local state with `this`, a normal function must be used
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}

// pass a string array to mapState when the name of a 
// mapped computed property is the same 
computed: mapState([
  // map this.count to store.state.count
  'count'
])

// object spread operator
computed: {
  localComputed () { /* ... */ },
  // mix this into the outer object with the 
  ...mapState({
    // ...
  })
}
```
