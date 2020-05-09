# Mutations

* The only way to actually change state in a Vuex store is by committing a mutation. 
* Vuex mutations are very __similar to events__
  * each mutation has a string type and a handler
  * __The handler function will receive the state as the first argument__ and is where we perform actual state modifications
    * may receive a __payload__ as second parameter
  * __Mutation handler functions must be synchronous__
    * when you call two methods both with async callbacks that mutate the state, how do you know when they are called and which callback was called first? 
    * This is exactly why we want to separate the two concepts. In Vuex, mutations are synchronous transactions, whereas actions are asynchronous operations
* Committing (triggering a mutation handler)
  * You cannot directly call a mutation handler: you need to call `store.commit` with its type
  * Think of it more like event registration: "When a mutation with type increment is triggered, call this handler."
  * An alternative way to commit a mutation is by directly using an object that has a type property
    * When using object-style commit, the entire object will be passed as the payload, so the handler remains the same
* You can commit mutations in components with 
  * `this.$store.commit('xxx')`, or 
  * `mapMutations` helper 
    * which maps component methods to `store.commit` calls (requires __root `store` injection__)
  * this is to allow debugging features (see above)

store.commit({
  type: 'increment',
  amount: 10
})

```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state) {
      // mutate state
      state.count++
    }
  }
})

// triggers the mutation of type 'increment' 
store.commit('increment')

// with payload as 2nd argument
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}
store.commit('increment', {
  amount: 10
})

// Object-style commit (object with a type property)
store.commit({
  type: 'increment',
  amount: 10
})

// trigger a mutation from inside a component
import { mapMutations } from 'vuex'
export default {
  // ...
  methods: {
    ...mapMutations([
      'increment', // map `this.increment()` to `this.$store.commit('increment')`

      // `mapMutations` also supports payloads:
      'incrementBy' // map `this.incrementBy(amount)` to `this.$store.commit('incrementBy', amount)`
    ]),
    ...mapMutations({
      add: 'increment' // map `this.add()` to `this.$store.commit('increment')`
    })
  }
}
```

## Mutations caveats

* Since a Vuex store's state is made reactive by Vue, when we mutate the state, Vue components observing the state will update automatically. 
* This also means Vuex mutations are subject to the same reactivity caveats when working with plain Vue:
  * Prefer initializing your store's initial state with all desired fields upfront.
  * When adding new properties to an Object, you should either:
    * Use `Vue.set(obj, 'newProp', 123)`, or
    * Replace that Object with a fresh one. For example, using the object spread syntax:
      * `state.obj = { ...state.obj, newProp: 123 }`

## Constants as mutation types, all in a single file

* It is a commonly seen pattern to use constants for mutation types in various Flux implementations. This allows the code to take advantage of tooling like linters, and putting all constants in a single file allows your collaborators to get an at-a-glance view of what mutations are possible in the entire application:

```js
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'
// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // we can use the ES2015 computed property name feature
    // to use a constant as the function name
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```

