# INSTALLATION

* [via npm or cdn](https://vuejs.org/v2/guide/installation.html)

# BASIC WORKING

* In an html page, you have to load vue.js as a js library (before your js code)
* Vue will replace a specified element in the html page with its own-managed virtual dom, providing reactivity and all its features
* To do that a Vue instance must be created (Vue object, root of the Vue app) with `new Vue( {options})`, where `options.el` selects the html element to be replaced

# VUE INSTANCE AND OPTIONS API

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
        <span>a is {{ a }}. Prop1 is {{ prop1 }}, Prop2 is {{ prop2 }}.</span>
        <component-a></component-a>
      </div>
    `,
    components: {
        'component-a': ComponentA,
    },
    data: { // instance inner variables
        a: 1,
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
    computed: {
        aDouble: vm => vm.a * 2
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
// In js code you can access the instance
// properties and methods with the "$" prefix, like this:
vm.$el
vm.$data
vm.$props
vm.$options
// ...etc
```

The options object is the most important API of Vue:

* `el` - Provide the Vue instance an existing DOM element to mount on. It can be a CSS selector string or an actual HTMLElement
* `template` - A string template to be used as the markup for the Vue instance. The template will replace the mounted element. In the template, you can use Vue's template syntax.
    * `render` can be used instead of template, to define it with js (or even jsx, react-wise)
* `components` - A hash of components to be made available to the Vue instance (usable in the template).
* `data` - The data object for the Vue instance. Vue will recursively convert its properties into getter/setters to make it “reactive”
* `props` - A list/hash of attributes that are exposed to accept data from the parent component. It has an Array-based simple syntax and an alternative Object-based syntax that allows advanced configurations such as type checking, custom validation and default values
    * `propsData` can be used only on the root Vue instance (`new Vue(...)`) and will pass the given values as props, instead of passing them via the template (for simplifying testing)
* `methods` - Methods to be mixed into the Vue instance. You can access these methods directly on the VM instance, or use them in directive expressions. All methods will have their this context automatically bound to the Vue instance. __You should not use an arrow function to define a method__ (or this won't be the Vue instance, it will be undefined) 
* `computed` - Computed properties to be mixed into the Vue instance. All getters and setters have their this context automatically bound to the Vue instance. Computed properties are cached, and only re-computed on reactive dependency changes. Note that __if you use an arrow function with a computed property, this won’t be the component’s instance, but you can still access the instance as the function’s first argument__.
* `watch` - An object where keys are expressions to watch and values are the corresponding callbacks. The value can also be a string of a method name, or an Object that contains additional options. The Vue instance will call `$watch()` for each entry in the object at instantiation. __You should not use an arrow function to define a method__ (or this won't be the Vue instance, it will be undefined)
* [Lifecycle hooks](https://vuejs.org/v2/api/#Options-Lifecycle-Hooks) provide a way to define callbacks to be executed on certain moments in the instance's lifecycle
* [See reference](https://vuejs.org/v2/api/) for advanded options such as
    * [`directives`, `filters`](https://vuejs.org/v2/api/#Options-Assets)
    * [`parent`, `mixins`, `extends`](https://vuejs.org/v2/api/#Options-Composition)
    * [`name` (recursively invoke component), `model` (custom v-model), `inheritAttrs`](https://vuejs.org/v2/api/#Options-Misc)
    * ...etc

# TEMPLATE SYNTAX

```html
<!-- INTERPOLATION -->
<span>My name is: {{ myName }}</span>
<span>I'm {{ myAge + 5 }} years old!</span>
<span>My pets' names are {{ myPets.join(", ") }}</span>
<span>I'm a {{ myAge > 50 ? 'kinda old' : 'young' }}!</span>
<!-- js Expressions, not statements! Following are wrong: -->
{{ let msg = 'Hello World!'; }}
{{ if(true) { return 'Yes!' } }}

<!-- HTML RENDERING -->
<span v-html="myHTMLData"></span>

<!-- ONE-WAY DATA BINDING (v-bind) -->
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

<!-- TWO WAY DATA BINDING (v-model <-> form inputs) -->
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
<!-- ...where message is a variable of a vue instance's data -->

<!-- EVENTS -->
<button v-on:click="myFunction"></button>
<!-- ...where myFunction is a function of a vue instance's methods -->
<button v-on:click="() => alert('Hello my friend')"></button>
<!-- with modifiers -->
<form v-on:submit.prevent="onSubmit"> ... </form>
<!-- shorthand syntax is "@" -->
<button @click="() => alert('Hello my friend')"></button>

<!-- DYNAMIC ARGUMENTS -->
<div v-bind:[myAttribute]="myData"></div>
<div v-on:[myEvent]="doSomething"></div>
<button v-on="{[myAttr]: true}">Click on me if you can</button>
<!-- myEvent is a variable of a vue instance's data and could be e.g. 'disabled' -->
<button v-on="{[myEvent]: function() { alert("Hello world!") }}">Hi</button>
<!-- myEvent is a variable of a vue instance's data and could be e.g. 'click' -->

<!-- CONDITIONAL RENDERING (v-if, v-show) -->
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

<!-- LIST RENDERING (v-for) -->
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

<!-- DOM PARSING CAVEATS ("is" special attribute ) -->
<!-- some elements, such as <ul>, <ol>, <table> and <select> --> 
<!-- have restrictions on what elements can appear inside them, -->
<!-- and some elements such as <li>, <tr>, and <option> can only -->
<!-- appear inside certain other elements. -->
<table>
  <blog-post-row></blog-post-row> <!-- not valid -->
</table>
<!-- The special "is" attribute is a solution for this problem: -->
<table>
  <tr is="blog-post-row"></tr> <!-- ok -->
</table>

<!-- DYNAMIC COMPONENTS -->
<!-- Component changes when currentTabComponent changes -->
<component v-bind:is="currentTabComponent"></component>

```