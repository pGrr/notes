# Modules

* Due to using a single state tree, all state of our application is contained inside one big object. However, as our application grows in scale, the store can get really bloated.
* To help with that, Vuex allows us to divide our store into modules
  * __Each module can contain its own state, mutations, actions, getters, and even nested modules__
* __Inside a module's mutations and getters the first argument received will be the module's local state__
* __Inside module getters, the root state will be exposed as their 3rd argument__
* __Inside module actions, `context.state` will expose the local state, and root state will be exposed as `context.rootState`__

```js
const moduleA = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... },
  getters: { ... }
}

const moduleB = {
  state: () => ({ ... }),
  mutations: { ... },
  actions: { ... }
}

const store = new Vuex.Store({
  modules: {
    a: moduleA,
    b: moduleB
  }
})

store.state.a // -> `moduleA`'s state
store.state.b // -> `moduleB`'s state

const moduleA = {
  state: () => ({
    count: 0
  }),
  mutations: {
    increment (state) {
      // the first argument is the local module state
      state.count++
    }
  },

  getters: {
    doubleCount (state) {
      return state.count * 2
    }
  }
}

const moduleA = {
  // context.state will expose the local state
  // and root state will be exposed as context.rootState
  actions: {
    incrementIfOddOnRootSum ({ state, commit, rootState }) {
      if ((state.count + rootState.count) % 2 === 1) {
        commit('increment')
      }
    }
  }
}

const moduleA = {
  // ...
  getters: {
    // rootState is the 3rd argument
    sumWithRootCount (state, getters, rootState) {
      return state.count + rootState.count
    }
  }
}
```

## Namespacing

* By default, actions, mutations and getters inside modules are still registered under the global namespace
* If you want your modules to be more self-contained or reusable, you can mark it as namespaced with `namespaced: true`
  * When the module is registered, all of its getters, actions and mutations will be automatically namespaced based on the path the module is registered at
  * Namespaced getters and actions will receive localized getters, dispatch and commit. In other words, you can use the module assets without writing prefix in the same module. Toggling between namespaced or not does not affect the code inside the module
* To access global state and getters:
  * `rootState` and `rootGetters` are passed as the 3rd and 4th arguments to getter functions
  * and also exposed as properties on the `context` object passed to action functions.
* To dispatch actions or commit mutations in the global namespace, pass `{ root: true }` as the 3rd argument to dispatch and commit
* If you want to register global actions in namespaced modules, you can mark it with `root: true` and place the action definition to function handler


```js
const store = new Vuex.Store({
  modules: {
    account: {
      namespaced: true,
      // module assets
      state: () => ({ ... }), // module state is already nested and not affected by namespace option
      getters: {
        isAdmin () { ... } // -> getters['account/isAdmin']
      },
      actions: {
        login () { ... } // -> dispatch('account/login')
      },
      mutations: {
        login () { ... } // -> commit('account/login')
      },
      // nested modules
      modules: {
        // inherits the namespace from parent module
        myPage: {
          state: () => ({ ... }),
          getters: {
            profile () { ... } // -> getters['account/profile']
          }
        },
        // further nest the namespace
        posts: {
          namespaced: true,

          state: () => ({ ... }),
          getters: {
            popular () { ... } // -> getters['account/posts/popular']
          }
        }
      }
    }
  }
})

// Accessing Global Assets in Namespaced Modules
modules: {
  foo: {
    namespaced: true,

    getters: {
      // `getters` is localized to this module's getters
      // you can use rootGetters via 4th argument of getters
      someGetter (state, getters, rootState, rootGetters) {
        getters.someOtherGetter // -> 'foo/someOtherGetter'
        rootGetters.someOtherGetter // -> 'someOtherGetter'
      },
      someOtherGetter: state => { ... }
    },

    actions: {
      // dispatch and commit are also localized for this module
      // they will accept `root` option for the root dispatch/commit
      someAction ({ dispatch, commit, getters, rootGetters }) {
        getters.someGetter // -> 'foo/someGetter'
        rootGetters.someGetter // -> 'someGetter'

        dispatch('someOtherAction') // -> 'foo/someOtherAction'
        dispatch('someOtherAction', null, { root: true }) // -> 'someOtherAction'

        commit('someMutation') // -> 'foo/someMutation'
        commit('someMutation', null, { root: true }) // -> 'someMutation'
      },
      someOtherAction (ctx, payload) { ... }
    }
  }
}

// Register Global Action in Namespaced Modules
{
  actions: {
    someOtherAction ({dispatch}) {
      dispatch('someAction')
    }
  },
  modules: {
    foo: {
      namespaced: true,

      actions: {
        someAction: {
          root: true,
          handler (namespacedContext, payload) { ... } // -> 'someAction'
        }
      }
    }
  }
}

// binding a namespaced module to components 
// with the mapState, mapGetters, mapActions and mapMutations helpers
computed: {
  ...mapState('some/nested/module', {
    a: state => state.a,
    b: state => state.b
  })
},
methods: {
  ...mapActions('some/nested/module', [
    'foo', // -> this.foo()
    'bar' // -> this.bar()
  ])
}

// create namespaced helpers by using createNamespacedHelpers
// (returns an object having new component binding helpers 
// that are bound with the given namespace value)
import { createNamespacedHelpers } from 'vuex'
const { mapState, mapActions } = createNamespacedHelpers('some/nested/module')
export default {
  computed: {
    // look up in `some/nested/module`
    ...mapState({
      a: state => state.a,
      b: state => state.b
    })
  },
  methods: {
    // look up in `some/nested/module`
    ...mapActions([
      'foo',
      'bar'
    ])
  }
}

// get namespace value via plugin option
// and returns Vuex plugin function
export function createPlugin (options = {}) {
  return function (store) {
    // add namespace to plugin module's types
    const namespace = options.namespace || ''
    store.dispatch(namespace + 'pluginAction')
  }
}
```

## Dynamic module registration 

* You can register a module after the store has been created with the `store.registerModule` method
  * Dynamic module registration makes it possible for other Vue plugins to also leverage Vuex for state management by attaching a module to the application's store
    * For example, the `vuex-router-sync` library integrates `vue-router` with `vuex` by managing the application's route state in a dynamically attached module
* You can remove a dynamically registered module with `store.unregisterModule(moduleName)` 
  * you cannot remove static modules (declared at store creation) with this method.
* you may check if the module is already registered to the store or not via store.hasModule(moduleName) method

```js
import Vuex from 'vuex'
const store = new Vuex.Store({ /* options */ })

// register a module `myModule`
// The module's state will be exposed as store.state.myModule
store.registerModule('myModule', {
  // ...
})

// register a nested module `nested/myModule`
// The module's state will be exposed as store.state.nested.myModule
store.registerModule(['nested', 'myModule'], {
  // ...
})
```

### Preserving state when attaching a module

* you can prevent state changes when you attach a module (assuming that your store state already contains state for that module and you don't want to overwrite it), e.g. preserving state from a Server Side Rendered app. You can achieve this with `preserveState` option: 
  * `store.registerModule('a', module, { preserveState: true })`
    * When you set preserveState: true, the module is registered, actions, mutations and getters are added to the store, but the state is not. It's assumed

# Module Reuse

* Sometimes we may need to create multiple instances of a module: this is actually the exact same problem with data inside Vue components. So the solution is also the same - __use a function for declaring module state__ (supported in 2.3.0+):

```js
const MyReusableModule = {
  state: () => ({
    foo: 'bar'
  }),
  // mutations, actions, getters...
}
```