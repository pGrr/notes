# VUEX

* Vuex is a state management pattern + library for Vue.js applications.
* It implements a **SINGLE STATE TREE** = **CENTRALIZED STORE** = **SINGLE SOURCE OF TRUTH**, which provides an easy way to manage shared state between components, ensuring that the state can only be mutated in a predictable fashion and thus simplifying code and its maintanance and debugging:
  * a single state tree makes it straightforward to locate a specific piece of state, and allows us to easily share state between components
  * state changing in a predictable way allow us to take snapshots of the current app state for debugging purposes

## The Vue problem, the Vuex solution

* Normal Vue apps are a __one-way data flow__:
  * The state, the source of truth that drives our app;
  * The view, a declarative mapping of the state;
  * The actions, the possible ways the state could change in reaction to user inputs from the view.
  * ![](img/vuex1.png)
* Without vuex you have 
  * events to comunicate state up the hierarchy
  * props to communicate state down the hierarchy
* The simplicity quickly breaks down when we have multiple components that share a common state:
  * Multiple views may depend on the same piece of state.
    * passing props can be tedious for deeply nested components, and simply doesn't work for sibling components
  * Actions from different views may need to mutate the same piece of state (such as in memory shared parallel processes).
    * we often find ourselves resorting to solutions such as reaching for direct parent/child instance references or trying to mutate and synchronize multiple copies of the state via events
* Vuex __extracts the shared state out of the components, and manage it in a global singleton__ thus making our component tree become a big "view", and __any component can access the state or trigger actions, no matter where they are in the tree__
  * __a single state tree__ - that is, this single object contains all your application level state and serves as the __"single source of truth"__
    * This also means usually you will have only one store for each application 
  * This is the basic idea inspired by Flux, Redux and The Elm Architecture.

![](img/vuex2.png)

* When to use it?
  * if your app is simple, you will most likely be fine without Vuex. [A simple store pattern](https://vuejs.org/v2/guide/state-management.html#Simple-State-Management-from-Scratch) may be all you need
  * if you are building a medium-to-large-scale SPA, chances are you have run into situations that make you think about how to better handle state outside of your Vue components, and Vuex will be the natural next step for you.
    * _Flux libraries are like glasses: you’ll know when you need them_

## ASSUMPTIONS: THE RULES OF THE GAME

* Vuex assumptions are:
  1. Application-level state is centralized in the store.
  1. The only way to mutate the state is by committing mutations, which are synchronous transactions.
  1. Asynchronous logic should be encapsulated in, and can be composed with, actions.

# THE STORE

* The store is basically **a container that holds your application state**. There are two things that make a Vuex store different from a plain global object:
  * **Vuex stores are reactive**. When Vue components retrieve state from it, they will reactively and efficiently update if the store's state changes
  * **You cannot directly mutate the store's state. The only way to change a store's state is by explicitly committing mutations**. This ensures every state change leaves a track-able record, and enables tooling that helps us better understand our applications, e.g. log every mutation, take state snapshots, or even perform time travel debugging

```js
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})

// ...and then:
store.commit('increment') // change the state commiting a mutation
console.log(store.state.count) // retrieve and use the state

// By declaring a store into a component:
new Vue({
  el: '#app',
  store // es6 syntax for store: store
})

// ...you can then access the store inside the component with this.$store
methods: {
  increment() {
    this.$store.commit('increment')
    console.log(this.$store.state.count)
  }
}
```

# APPLICATION STRUCTURE AND STORE FILE SPLITTING

* As long as you follow the Vuex assumptions it's up to you how to structure your project. Anyway, rules of thumb are: 
  * if your store file gets too big you should start **splitting into separate files**, e.g. `actions.js`, `mutations.js` and `getters.js` into separate files with an `index.js` file importing them
  * For any non-trivial app, you should use **MODULES**
  
Here's an example project structure leveraging both **file splitting** and **modules**:

```text
├── index.html
├── main.js
├── api
│   └── ... # abstractions for making API requests
├── components
│   ├── App.vue
│   └── ...
└── store
    ├── index.js          # where we assemble modules and export the store
    ├── actions.js        # root actions
    ├── mutations.js      # root mutations
    └── modules
        ├── cart.js       # cart module
        └── products.js   # products module
```

## MODULES

* Following Vuex assumptions, there should be only one store for the whole application
* For complex applications Vuex allows us to divide our store into modules. Each module can contain its own state, mutations, actions, getters, and even nested modules - it's fractal all the way down:

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
```