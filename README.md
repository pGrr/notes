# INSTALLATION

* [via npm or cdn](https://vuejs.org/v2/guide/installation.html)

# BASIC WORKING

* In an html page, you have to load vue.js as a js library (before your js code)
* Vue will replace a specified element in the html page with its own-managed virtual dom, providing reactivity and all its features
* To do that a Vue instance must be created (Vue object, root of the Vue app) with `new Vue( {options})`, where `options.el` selects the html element to be replaced

# VUE INSTANCE AND OPTIONS API

The options object is the most important API of Vue:

* `el` - Provide the Vue instance an existing DOM element to mount on.
    * It can be a CSS selector string or an actual HTMLElement
* `template` - A string template to be used as the markup for the Vue instance. The template will replace the mounted element. 
    * In the template, you can use Vue's template syntax.
    * `render` can be used instead of template, to define it with js (or even jsx, react-wise)
* `components` - A hash of components to be made available to the Vue instance
    * components can be used in the template (or directly in the html bound with the vue instance)
* `data` - The data object for the Vue instance.
    * Vue will recursively convert its properties into getter/setters to make it “reactive”
    * variables inside data can be used in the template (or directly in the html bound with the vue instance)
* `props` - A list/hash of attributes that are exposed to accept data from the parent component.
    * It has an Array-based simple syntax and an alternative Object-based syntax that allows advanced configurations such as type checking, custom validation and default values
    * props values will be passed as html properties of the custom element
    * `propsData` can be used only on the root Vue instance (`new Vue(...)`) and will pass the given values as props, instead of passing them via the template (for simplifying testing)
* `methods` - Methods to be mixed into the Vue instance.
    * You can access these methods directly on the VM instance, or use them in directive expressions. 
    * All methods will have their this context automatically bound to the Vue instance. __You should not use an arrow function to define a method__ (or this won't be the Vue instance, it will be undefined) 
* `computed` - Computed properties to be mixed into the Vue instance.
    * Computed properties are cached, and only re-computed on reactive dependency changes (unlike methods, [see more](https://vuejs.org/v2/guide/computed.html#Computed-Caching-vs-Methods)). 
    * For some tasks, e.g. async fetching, watch is more appropriate ([see more](https://vuejs.org/v2/guide/computed.html#Computed-vs-Watched-Property)). 
    * All getters and setters have their this context automatically bound to the Vue instance. __If you use an arrow function with a computed property, this won’t be the component’s instance, but you can still access the instance as the function’s first argument__.
* `watch` - An object where keys are expressions to watch and values are the corresponding callbacks.
    * The value can also be a string of a method name, or an Object that contains additional options. The Vue instance will call `$watch()` for each entry in the object at instantiation.
    * Computed properties should be preferred when you have some data that needs to change based on some other data - [more](https://vuejs.org/v2/guide/computed.html#Computed-vs-Watched-Property)
    * __You should not use an arrow function to define a method__ (or this won't be the Vue instance, it will be undefined)
* [Lifecycle hooks](https://vuejs.org/v2/api/#Options-Lifecycle-Hooks) provide a way to define callbacks to be executed on certain moments in the instance's lifecycle
* [See reference](https://vuejs.org/v2/api/) for advanded options such as
    * [`directives`, `filters`](https://vuejs.org/v2/api/#Options-Assets)
    * [`parent`, `mixins`, `extends`](https://vuejs.org/v2/api/#Options-Composition)
    * [`name` (recursively invoke component), `model` (custom v-model), `inheritAttrs`](https://vuejs.org/v2/api/#Options-Misc)
    * ...etc

```html
<!-- html (the view) -->
<div id="app"></div>
```

```js
// js (the model)
var vm = new Vue({
    el: '#app', // actual html element that will be replaced by template
    template: `
      <div>
        <span>
            a is {{ a }}, b is {{ b }}. 
            Prop1 is {{ prop1 }}, 
            Prop2 is {{ prop2 }}.
            aDouble is {{ aDouble }}
        </span>
        <component-a></component-a>
      </div>
    `,
    components: {
        'component-a': ComponentA,
    },
    data: { // instance inner variables
        a: 1,
        b: 2,
    },
    props: { // variables passed as arguments
        // a hash with type checking:
        prop1: Number,
        prop2: {
            type: Number,
            default: 0,
            required: true,
            validator: function (value) {
                return value >= 0
            }
        }
        // or without type-checking as list: ['prop1', 'prop2']
    },
    methods: { // methods that operates on data and/or props
        plus: function () {
            this.a++
        }
    },
    computed: { // The function will be used as the getter of the property
        // arrow function: 
        // fist parameter is the vue instance
        aDouble: vm => vm.a * 2,
        bDouble: {
            // not arrow-function:
            // "this" is the Vue instance
            get: function() {
                return this.b * 2;
            },
            // you can define setters:
            // (on bDouble set, update b)
            set: function(newValue) {
                this.b = newValue / 2;
            }
        }
    },
    watch: {
        a: function (val, oldVal) {
            console.log('new: %s, old: %s', val, oldVal)
        },
        prop1: function (val, oldVal) {
            console.log('new: %s, old: %s', val, oldVal)
        }, 
    }
})
```

```js
// In js code you can access userd-defined data, methods, etc
// from the instance object: 
vm.a
vm.b
vm.plus()
// ...and instance properties and methods with the "$" prefix
// (to differentiate them from user-defined ones):
vm.$el
vm.$data
vm.$props
vm.$options
```

# TEMPLATE SYNTAX API

* This is Vue's own syntax, usable inside a template's definition or directly inside a Vue instance (the root vue instance or any vue component)

```html
<!------------------------------------------------------------->
<!-- INTERPOLATION -->
<!------------------------------------------------------------->

<span>My name is: {{ myName }}</span>

<span>I'm {{ myAge + 5 }} years old!</span>

<span>My pets' names are {{ myPets.join(", ") }}</span>

<span>I'm a {{ myAge > 50 ? 'kinda old' : 'young' }}!</span>

<!-- js Expressions, not statements! Following are wrong: -->
{{ let msg = 'Hello World!'; }}
{{ if(true) { return 'Yes!' } }}

<!------------------------------------------------------------->
<!-- HTML RENDERING -->
<!------------------------------------------------------------->

<span v-html="myHTMLData"></span>

<!------------------------------------------------------------->
<!-- IMMUTABILITY (v-once) -->
<!------------------------------------------------------------->

<span v-once>This will never change: {{ msg }}</span>

<!------------------------------------------------------------->
<!-- ONE-WAY DATA BINDING (v-bind = :) -->
<!------------------------------------------------------------->

<span v-bind:class="myClass"></span>

<!-- shorthand syntax is ":" -->

<span :class="myClass"></span>

<div :class="{green: true, red: false}"></div>

<div :class="{green: 5 > 1, red: false && 9 < 16}"></div>

<!-- you can concatenate strings -->

<a :href="baseURL + '/post/' + postId">Read more</a>

<!-- no-value attributes (required, disabled, etc) -->

<button :required="true"></button>

<button :disabled="password.length < 6"></button>

<!------------------------------------------------------------->
<!-- TWO WAY DATA BINDING (v-model <-> form inputs) -->
<!------------------------------------------------------------->

<input v-model="message" placeholder="edit me">
<!-- ...where message is a variable of a vue instance's data -->

<!------------------------------------------------------------->
<!-- EVENTS (v-on = @)-->
<!------------------------------------------------------------->

<button v-on:click="myFunction"></button>

<button v-on:click="counter += 1">Add</button>
<!-- ...where myFunction is a function of a vue instance's methods -->

<button v-on:click="() => alert('Hello my friend')"></button>

<!-- you can pass the original DOM event to the function -->
<!-- with the special variable "$event" -->
<button v-on:click="warn('Warning!', $event)"></button>

<!-- shorthand syntax is "@" -->
<button @click="() => alert('Hello my friend')"></button>

<!-- Event modifiers -->

<form v-on:submit.prevent="onSubmit"> ... </form>

<!-- Key event modifiers -->

<!-- called if $event.key == `Enter` -->
<input v-on:keyup.enter="submit">

<!-- Alt + C -->
<input v-on:keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div v-on:click.ctrl="doSomething">Do something</div>

<!-- Ctrl and no other keys -->
<button v-on:click.ctrl.exact="onCtrlClick">A</button>

<!-- only click -->
<button v-on:click.exact="onClick">A</button>

<!------------------------------------------------------------->
<!-- DYNAMIC ARGUMENTS -->
<!------------------------------------------------------------->

<div v-bind:[myAttribute]="myData"></div>

<div v-on:[myEvent]="doSomething"></div>

<button v-on="{[myAttr]: true}">Click on me if you can</button>
<!-- myAttr is a variable of a vue instance's data and could be e.g. 'disabled' -->

<button v-on="{[myEvent]: function() { alert("Hello world!") }}">Hi</button>
<!-- myEvent is a variable of a vue instance's data and could be e.g. 'click' -->

<!------------------------------------------------------------->
<!-- CONDITIONAL RENDERING (v-if, v-show) -->
<!------------------------------------------------------------->

<!-- v-if/v-elseif/v-else (true conditional rendering) -->

<div v-if="type === 'A'">A</div>

<div v-else-if="type === 'B'">B</div>

<div v-else>Not A nor B</div>

<!-- v-show (always rendered, conditional display) -->

<h1 v-show="ok">Hello!</h1>

<!-- use template tag to apply it to a group of elements -->

<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>

<!------------------------------------------------------------->
<!-- LIST RENDERING (v-for) -->
<!------------------------------------------------------------->

<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>

<!-- iterate through properties of an object -->
<div v-for="(value, name, index) in object">
  {{ index }}. {{ name }}: {{ value }}
</div>

<!-- giving an integer n will repeat the element n times -->
<span v-for="n in 10">{{ n }} </span>

<!-- use template tag to group multiple elements -->
<template v-for="item in items">
  <li>{{ item.msg }}</li>
  <li class="divider" role="presentation"></li>
</template>

<!-- it’s not recommended to use v-if and v-for together -->

<!------------------------------------------------------------->
<!-- DOM PARSING CAVEATS ("is" special attribute ) -->
<!------------------------------------------------------------->

<!-- some elements, such as <ul>, <ol>, <table> and <select> --> 
<!-- have restrictions on what elements can appear inside them, -->
<!-- and some elements such as <li>, <tr>, and <option> can only -->
<!-- appear inside certain other elements. -->
<!-- The special "is" attribute is a solution for this problem: -->

<table>
  <blog-post-row></blog-post-row> <!-- not valid -->
</table>

<table>
  <tr is="blog-post-row"></tr> <!-- ok -->
</table>

<!------------------------------------------------------------->
<!-- DYNAMIC COMPONENTS -->
<!------------------------------------------------------------->

<!-- Component changes when currentTabComponent changes -->
<component v-bind:is="currentTabComponent"></component>
```

# CLASS AND STYLE BINDINGS

## Binding classes

* Vue provides special enhancements when v-bind is used with class and style: in addition to strings, the expressions can also evaluate to objects or arrays
* `v-bind:class` can coexist with a `class` attribute 
* classes added with `v-bind` will be added to the component’s root element and existing classes on this element will not be overwritten - [more](https://vuejs.org/v2/guide/class-and-style.html#With-Components)

```html
<!-- the presence of the active class will be determined by the data property isActive -->
<div v-bind:class="{ active: isActive }"></div>

<!-- v-bind:class can coexist with class and have multiple properties -->
<div
  class="static"
  v-bind:class="{ active: isActive, 'text-danger': hasError }"
></div>

<!-- and can be defined as object of data: -->
<div v-bind:class="classObject"></div>
```
```js
data: {
  classObject: {
    active: true,
    'text-danger': false
  }
}
```

```html
<!-- or as a computed property -->
<div v-bind:class="classObject"></div>
```
```js
data: {
  isActive: true,
  error: null
},
computed: {
  classObject: function () {
    return {
      active: this.isActive && !this.error,
      'text-danger': this.error && this.error.type === 'fatal'
    }
  }
}
```

```html
<!-- Multiple classes can be specified with the array syntax -->
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
<!-- and you can combine it with the object syntax (same as above, but less verbose) -->
<div v-bind:class="[{ active: isActive }, errorClass]"></div>
```
```js
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

## Binding inline styles

* The object syntax for `v-bind:style` is pretty straightforward - it looks almost like CSS, except it’s a JavaScript object. __You can use either camelCase or kebab-case (use quotes with kebab-case) for the CSS property names__ - [more](https://vuejs.org/v2/guide/class-and-style.html#Binding-Inline-Styles)
* Note: When you use a CSS property that requires vendor prefixes in v-bind:style, for example transform, Vue will automatically detect and add appropriate prefixes to the applied styles.

```html
<div v-bind:style="styleObject"></div>
```
```js
data: {
  styleObject: {
    color: 'red',
    fontSize: '13px'
  }
} // you can as well computed properties
```

```html
<!-- you can apply multiple style objects to the same element with the array syntax -->
<div v-bind:style="[baseStyles, overridingStyles]"></div>
```

# EVENTS

## v-on (@) directive

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
```

## $event = original DOM event

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

## Key events modifiers

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

## Mouse event modifiers

* These modifiers restrict the handler to events triggered by a specific mouse button - [more](https://vuejs.org/v2/guide/events.html#Mouse-Button-Modifiers)
  * `.left`
  * `.right`
  * `.middle`

## System key events modifiers

* You can use the following modifiers to trigger mouse or keyboard event listeners only when the corresponding modifier key is pressed (e.g. keyup.ctrl will only trigger if you release a key while holding down ctrl. It won’t trigger if you release the ctrl key alone):
  * `.ctrl`
  * `.alt`
  * `.shift`
  * `.meta` (Windows key, Macintosh key, etc)

```html
<!-- Alt + C -->
<input v-on:keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div v-on:click.ctrl="doSomething">Do something</div>
```

## Exact

* The `.exact` modifier allows control of the exact combination of system modifiers needed to trigger an event. - [more](https://vuejs.org/v2/guide/events.html#exact-Modifier)

```html
<!-- this will only fire when Ctrl and no other keys are pressed -->
<button v-on:click.ctrl.exact="onCtrlClick">A</button>

<!-- this will only fire when no system modifiers are pressed -->
<button v-on:click.exact="onClick">A</button>
```

# TWO WAY DATA-BINDING (v-model and FORM INPUTs)

* You can use the v-model directive to create two-way data bindings on form input: it automatically picks the correct way (specific properties, events, etc) to update the element based on the input type
* Vue data is treated as single source of truth (existing `value`, `checked`, or `selected` will be ignored)
* [more](https://vuejs.org/v2/guide/forms.html)


```html
<input type="checkbox" id="jack" value="Jack" v-model="checkedNames">
<label for="jack">Jack</label>
<input type="checkbox" id="john" value="John" v-model="checkedNames">
<label for="john">John</label>
<input type="checkbox" id="mike" value="Mike" v-model="checkedNames">
<label for="mike">Mike</label>
<br>
<span>Checked names: {{ checkedNames }}</span>
```
```js
new Vue({
  el: '...',
  data: {
    checkedNames: []
  }
})
```

```html
<select v-model="selected">
  <option v-for="option in options" v-bind:value="option.value">
    {{ option.text }}
  </option>
</select>
<span>Selected: {{ selected }}</span>
```
```js
new Vue({
  el: '...',
  data: {
    selected: 'A',
    options: [
      { text: 'One', value: 'A' },
      { text: 'Two', value: 'B' },
      { text: 'Three', value: 'C' }
    ]
  }
})
```

## v-model value Bindings

* For radio, checkbox and select options, the v-model binding values are usually static strings (or booleans for checkboxes). But sometimes, we may want to bind the value to a dynamic property on the Vue instance. We can use `v-bind` to achieve that. In addition, using `v-bind` allows us to bind the input value to non-string values
* [more](https://vuejs.org/v2/guide/forms.html#Value-Bindings)

```html
<!-- `picked` is a string "a" when checked -->
<input type="radio" v-model="picked" value="a">

<!-- `picked` is the value of the variable a when checked -->
<input type="radio" v-model="picked" v-bind:value="a">
```

## Checkboxes true-value and false-value

```html
<input
  type="checkbox"
  v-model="toggle"
  true-value="yes"
  false-value="no"
>
```

## v-model modifiers

```html
<!-- synced after "change" instead of "input" -->
<input v-model.lazy="msg">

<!-- input to be automatically typecast as a Number (even with type="number", the value of HTML input elements always returns a string) -->
<input v-model.number="age" type="number">

<!-- whitespace to be trimmed automatically -->
<input v-model.trim="msg">
```

## v-model with Components

* HTML’s built-in input types won’t always meet your needs. Vue components allow you to build reusable inputs with completely customized behavior. These inputs even work with `v-model`! - [more](https://vuejs.org/v2/guide/components.html#Using-v-model-on-Components)


# COMPONENTS

* Components are reusable Vue instances with a name
  * __they accept the same options as `new Vue`__. The only exceptions are a few root-specific options like `el`
  * __`data` must be a function, not an object__ (else changing the data of one instance would affect the data of all other instances)
  * __components must have only one root element__
  * __components must be registered, globally or locally__, in order for Vue to use them
* Components can be nested in a hierarchial tree. Communication between nodes of this tree happens in two way:
    * __variable data of a component can be passed down the tree via `props`__ (like a arguments for a function) - [more](https://vuejs.org/v2/guide/components.html#Passing-Data-to-Child-Components-with-Props)
    * __a component can throw an event (and a value) and its parent can choose to listen of any event using `v-on`__ (in the same way as with any native DOM event) - [more](https://vuejs.org/v2/guide/components.html#Listening-to-Child-Components-Events)
* Custom events can also be used to create custom inputs that work with v-model - [more](https://vuejs.org/v2/guide/components.html#Using-v-model-on-Components)
* [more](https://vuejs.org/v2/guide/components.html)

## Props (downward)

* downwards, from the parent to the component, via props, which become html attributes (alike to arguments of a function or of the constructor of a class) - [more](https://vuejs.org/v2/guide/components.html#Passing-Data-to-Child-Components-with-Props)

```js
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <div v-html="post.content"></div>
    </div>
  `
})
```
```js
new Vue({
  el: '#blog-post-demo',
  data: {
    posts: [
      { id: 1, title: 'My journey with Vue' },
      { id: 2, title: 'Blogging with Vue' },
      { id: 3, title: 'Why Vue is so fun' }
    ]
  }
})
```

```html
<blog-posts
  v-for="post in posts"
  v-bind:key="post.id"
  v-bind:post="post"
></blog-posts>
```

## Events (upwards)

* __The child component can emit an event on itself by calling the built-in `$emit` method, passing the name of the event as first parameter and, if needed, a single value as second parameter__
* __The parent can choose to listen to any event on the child component instance with `v-on` (providing the event name). The emitted event’s associated value, if present, can be accessed with `$event`__
    * __Or, if the event handler is a method, the value will be passed as the first parameter of that method__

Root Vue instance:

```js
new Vue({
  el: '#blog-posts-events-demo', // parent's id
  data: {
    posts: [/* ... */],
    postFontSize: 1
  }
})
```

Child:

```js
Vue.component('blog-post', {
  props: ['post'],
  template: `
    <div class="blog-post">
      <h3>{{ post.title }}</h3>
      <button v-on:click="$emit('enlarge-text', 0.1)">
        Enlarge text
      </button>
      <div v-html="post.content"></div>
    </div>
  `
})
```

Parent: 

```html
<div id="blog-posts-events-demo">
  <div :style="{ fontSize: postFontSize + 'em' }">
    <blog-post
      v-for="post in posts"
      v-bind:key="post.id"
      v-bind:post="post"
      v-on:enlarge-text="postFontSize += $event"
    ></blog-post>
  </div>
</div>
```

Parent (if the event handler is a function):

```js
// ...
methods: {
  onEnlargeText: function (enlargeAmount) {
    this.postFontSize += enlargeAmount
  }
}
// ...
```
```html
<blog-post
  ...
  v-on:enlarge-text="onEnlargeText"
></blog-post>
```

# SLOTS (custom element's content)

* Just like with HTML elements, it’s often useful to be able to pass content to a component - [more](https://vuejs.org/v2/guide/components.html#Content-Distribution-with-Slots)
* we just add the `<slot></slot>` where we want it to go – and that’s it. We’re done!
* [more (full guide on slots)](https://vuejs.org/v2/guide/components-slots.html)

```html
<alert-box>
  Something bad happened.
</alert-box>
```
```js
Vue.component('alert-box', {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot>
    </div>
  `
})
```
# DYNAMIC COMPONENTS (switching components)

* Sometimes, it’s useful to dynamically switch between components, i.e. changing dinamically what component should be displayed (such as in a tabbed menu). This is made possible by Vue’s `<component>` element with the `is` special attribute, to which we can pass the name of a registered component, or a component’s options object - [more](https://vuejs.org/v2/guide/components.html#Dynamic-Components)
  * this attribute can also be used with regular HTML elements, but with caveats 
* [full guide](https://vuejs.org/v2/guide/components-dynamic-async.html)

```html
<!-- Component changes when currentTabComponent changes -->
<component v-bind:is="currentTabComponent"></component>
```

# DOM TEMPLATE PARSING CAVEATS

* Some HTML elements, such as `<ul>`, `<ol>`, `<table>` and `<select>` have restrictions on what elements can appear inside them, and some elements such as `<li>`, `<tr>`, and `<option>` can only appear inside certain other elements: this will lead to hoisting and rendering errors
* Fortunately, the `is` special attribute offers a workaround:
* This limitation does not apply if you are using string templates from one of the following sources:
    * String templates (e.g. `template: '...'`)
    * Single-file (`.vue`) components
    * `<script type="text/x-template">`
* [more](https://vuejs.org/v2/guide/components.html#DOM-Template-Parsing-Caveats)

```html
<!-- error -->
<table>
  <blog-post-row></blog-post-row>
</table>
```
```html
<!-- valid -->
<table>
  <tr is="blog-post-row"></tr>
</table>
```


