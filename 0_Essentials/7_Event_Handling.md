# EVENT HANDLING (v-on)

* We can use the v-on directive to listen to DOM events and run some JavaScript when they’re triggered.
* [more](https://vuejs.org/v2/guide/events.html#Listening-to-Events)

```html
<div id="example-1">
  <button v-on:click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
```
```js
var example1 = new Vue({
  el: '#example-1',
  data: {
    counter: 0
  }
})
```

* v-on can also accept the name of a method you’d like to call - [more](https://vuejs.org/v2/guide/events.html#Method-Event-Handlers)

```html
<div id="example-2">
  <!-- `greet` is the name of a method defined below -->
  <button v-on:click="greet">Greet</button>
</div>
```
```js
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },
  // define methods under the `methods` object
  methods: {
    greet: function (event) {
      // `this` inside methods points to the Vue instance
      alert('Hello ' + this.name + '!')
      // `event` is the native DOM event
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})
// you can invoke methods in JavaScript too
example2.greet() // => 'Hello Vue.js!'
```

## Methods in Inline Handlers

* Sometimes we also need to access the original DOM event in an inline statement handler. You can pass it into a method using the special $event variable - [more](https://vuejs.org/v2/guide/events.html#Methods-in-Inline-Handlers)

```html
<button v-on:click="warn('Form cannot be submitted yet.', $event)">
  Submit
</button>
```
```js
// ...
methods: {
  warn: function (message, event) {
    // now we have access to the native event
    if (event) {
      event.preventDefault()
    }
    alert(message)
  }
}
```

## Event modifiers

* To help event handlers contain only logic, instead of calling event modifiers such as `event.preventDefault()` or `event.stopPropagation()` inside event handlers, Vue provides event modifiers for `v-on` (directive postfixes denoted by a dot) - [more](https://vuejs.org/v2/guide/events.html#Event-Modifiers)

```html
<!-- the click event's propagation will be stopped -->
<a v-on:click.stop="doThis"></a>

<!-- the submit event will no longer reload the page -->
<form v-on:submit.prevent="onSubmit"></form>

<!-- modifiers can be chained -->
<a v-on:click.stop.prevent="doThat"></a>

<!-- just the modifier -->
<form v-on:submit.prevent></form>

<!-- use capture mode when adding the event listener -->
<!-- i.e. an event targeting an inner element is handled here before being handled by that element -->
<div v-on:click.capture="doThis">...</div>

<!-- only trigger handler if event.target is the element itself -->
<!-- i.e. not from a child element -->
<div v-on:click.self="doThat">...</div>

<!-- the click event will be triggered at most once (this is valid for components as well, not only dom events) -->
<a v-on:click.once="doThis"></a>

<!-- the scroll event's default behavior (scrolling) will happen -->
<!-- immediately, instead of waiting for `onScroll` to complete  -->
<!-- in case it contains `event.preventDefault()`. This is especially useful for improving performance on mobile devices, but don’t use .passive and .prevent together-->
<div v-on:scroll.passive="onScroll">...</div>
```

## Key modifiers (keyboard events)

* When listening for keyboard events, we often need to check for specific keys. Vue allows adding key modifiers for v-on when listening for key events
* You can directly use any valid key names exposed via `KeyboardEvent.key` as modifiers by converting them to kebab-case.
* Vue provides aliases for the most commonly used key codes when necessary for legacy browser support:
  * `.enter`
  * `.tab`
  * `.delete` (captures both “Delete” and “Backspace” keys)
  * `.esc`
  * `.space`
  * `.up`
  * `.down`
  * `.left`
  * `.right`
* [more](https://vuejs.org/v2/guide/events.html#Key-Modifiers)

```html
<!-- only call `vm.submit()` when the `key` is `Enter` -->
<input v-on:keyup.enter="submit">

<!-- the handler will only be called if $event.key is equal to 'PageDown' -->
<input v-on:keyup.page-down="onPageDown">
```

### System modifiers key

* You can use the following modifiers to trigger mouse or keyboard event listeners only when the corresponding modifier key is pressed:
  * `.ctrl`
  * `.alt`
  * `.shift`
  * `.meta` (Windows key, Macintosh key, etc)
* Note that modifier keys are different from regular keys and when used with keyup events, they have to be pressed when the event is emitted. In other words, keyup.ctrl will only trigger if you release a key while holding down ctrl. It won’t trigger if you release the ctrl key alone.
* [more](https://vuejs.org/v2/guide/events.html#System-Modifier-Keys)

```html
<!-- Alt + C -->
<input v-on:keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div v-on:click.ctrl="doSomething">Do something</div>
```

### `exact` modifier

* The `.exact` modifier allows control of the exact combination of system modifiers needed to trigger an event. - [more](https://vuejs.org/v2/guide/events.html#exact-Modifier)

```html
<!-- this will fire even if Alt or Shift is also pressed -->
<button v-on:click.ctrl="onClick">A</button>

<!-- this will only fire when Ctrl and no other keys are pressed -->
<button v-on:click.ctrl.exact="onCtrlClick">A</button>

<!-- this will only fire when no system modifiers are pressed -->
<button v-on:click.exact="onClick">A</button>
```

## Mouse button modifiers

* These modifiers restrict the handler to events triggered by a specific mouse button - [more](https://vuejs.org/v2/guide/events.html#Mouse-Button-Modifiers)
  * `.left`
  * `.right`
  * `.middle`

  