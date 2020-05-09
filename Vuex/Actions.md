# Actions

* Actions are similar to mutations, but:
  * __Instead of mutating the state, actions commit mutations__
  * Actions __can contain arbitrary asynchronous operations__
* Action handlers receive a __`context` object__ which exposes the same set of methods/properties on the store instance, so you can 
  * call `context.commit` to commit a mutation, or 
  * access the state and getters via `context.state` and `context.getters`
  * call other actions with `context.dispatch`
  * But it's not the store instance itself! Why? see modules
* Actions are __triggered with the `store.dispatch` method__
  * e.g. `store.dispatch('commit');`
  * why don't we just call `store.commit('increment')` directly? Remember that __mutations have to be synchronous. Actions don't__. We can perform asynchronous operations inside an action
* Actions __support the same payload format and object-style dispatch as mutations__
* You can dispatch actions in components with 
  * `this.$store.dispatch('xxx')`, or 
  * `mapActions` helper 
    * which maps component methods to `store.dispatch` calls (__requires root store injection__)

```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})

// dispatch (trigger an action)
store.dispatch('increment')

// dispatch with a payload
store.dispatch('incrementAsync', {
  amount: 10
})

// dispatch with an object
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})

// realistic example, with asynch operation
// and multiple mutations:
actions: {
  checkout ({ commit, state }, products) {
    // save the items currently in the cart
    const savedCartItems = [...state.cart.added]
    // send out checkout request, and optimistically
    // clear the cart
    commit(types.CHECKOUT_REQUEST)
    // the shop API accepts a success callback and a failure callback
    shop.buyProducts(
      products,
      // handle success
      () => commit(types.CHECKOUT_SUCCESS),
      // handle failure
      () => commit(types.CHECKOUT_FAILURE, savedCartItems)
    )
  }
}

// dispatching actions from inside components
import { mapActions } from 'vuex'
export default {
  // ...
  methods: {
    ...mapActions([
      'increment', // map `this.increment()` to `this.$store.dispatch('increment')`

      // `mapActions` also supports payloads:
      'incrementBy' // map `this.incrementBy(amount)` to `this.$store.dispatch('incrementBy', amount)`
    ]),
    ...mapActions({
      add: 'increment' // map `this.add()` to `this.$store.dispatch('increment')`
    })
  }
}
```

## Composing actions

* Actions are often asynchronous, so how do we know when an action is done? And more importantly, how can we compose multiple actions together to handle more complex async flows?
* __`store.dispatch` can handle Promise returned by the triggered action handler and it also returns Promise__
* __`store.dispatch` can trigger multiple action handlers in different modules: the returned value will be a Promise that resolves when all triggered handlers have been resolved__

```js
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}

// Now you can do:
store.dispatch('actionA').then(() => {
  // ...
})

// And also in another action:
actions: {
  // ...
  actionB ({ dispatch, commit }) {
    return dispatch('actionA').then(() => {
      commit('someOtherMutation')
    })
  }
}

// Async / Await syntax
// assuming `getData()` and `getOtherData()`
// return Promises
actions: {
  async actionA ({ commit }) {
    commit('gotData', await getData())
  },
  async actionB ({ dispatch, commit }) {
    await dispatch('actionA') // wait for `actionA` to finish
    commit('gotOtherData', await getOtherData())
  }
}
```
