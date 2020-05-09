# Getters

* Vuex allows us to define "getters" in the store. 
* You can think of them as computed properties for stores
  * Like computed properties, a getter's result is cached based on its dependencies, and will only re-evaluate when some of its dependencies have changed.
* Getters will
  * receive the state as their 1st argument
  * may receive other getters as the 2nd argument
  * be exposed on the `store.getters` object, and you access values as properties (__Property-Style Access__)
    * getters accessed as properties __are cached__ as part of Vue's reactivity system
  * You can also __pass arguments to getters by returning a function__. This is particularly useful when you want to query an array in the store (__Method style access__)
    * will run at each call (not cached)
  * The `mapGetters` helper simply maps store getters to local computed properties

```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  getters: {
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
  }
})

// Property-style access (cached)
store.getters.doneTodos 
// -> [{ id: 1, text: '...', done: true }]

// can receive other getters as 2nd arg
getters: {
  // ...
  doneTodosCount: (state, getters) => {
    return getters.doneTodos.length
  }
}

// use from other components through this.$store
computed: {
  doneTodosCount () {
    return this.$store.getters.doneTodosCount
  }
}

// Method style access (not cached)
getters: {
  // ...
  getTodoById: (state) => (id) => {
    return state.todos.find(todo => todo.id === id)
  }
}
store.getters.getTodoById(2) // -> { id: 2, text: '...', done: false }

// mapGetters Helper
import { mapGetters } from 'vuex'
export default {
  // ...
  computed: {
    // mix the getters into computed with object spread operator
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}

// if you want to map a getter to a different name, use an object:
...mapGetters({
  // map `this.doneCount` to `this.$store.getters.doneTodosCount`
  doneCount: 'doneTodosCount'
})
```
