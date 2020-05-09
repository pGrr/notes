# Vuex - state management pattern library

* Vuex is a state management pattern + library for Vue.js applications. It serves as a centralized store for all the components in an application, with rules ensuring that the state can only be mutated in a predictable fashion.

## The problem

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

## The solution

* Vuex __extracts the shared state out of the components, and manage it in a global singleton__ thus making our component tree become a big "view", and __any component can access the state or trigger actions, no matter where they are in the tree__
  * __a single state tree__ - that is, this single object contains all your application level state and serves as the __"single source of truth"__
    * This also means usually you will have only one store for each application 
  * This is the basic idea inspired by Flux, Redux and The Elm Architecture.

![](img/vuex2.png)

* When to use it?
  * Vuex helps us deal with shared state management with the cost of more concepts and boilerplate. It's a trade-off between short term and long term productivity.
  * if your app is simple, you will most likely be fine without Vuex. [A simple store pattern](https://vuejs.org/v2/guide/state-management.html#Simple-State-Management-from-Scratch) may be all you need
  * if you are building a medium-to-large-scale SPA, chances are you have run into situations that make you think about how to better handle state outside of your Vue components, and Vuex will be the natural next step for you.
    * _Flux libraries are like glasses: you’ll know when you need them_
