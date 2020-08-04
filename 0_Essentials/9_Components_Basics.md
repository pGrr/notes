# COMPONENTS

* Components are reusable Vue instances with a name
* Since components are reusable Vue instances, they accept the same options as `new Vue`, such as `data`, `computed`, `watch`, `methods`, and lifecycle hooks. The only exceptions are a few root-specific options like `el`.
* Being themself Vue instances (though not the root one), they follow the same principles and syntax, but with some caveats:
    * __`data` must be a function, not an object__, so that each instance can maintain an independent copy of the returned data object. If Vue didn’t have this rule, changing the data of one instance would affect the data of all other instances
    * __components must have only one root element__
    * __components must be registered, globally or locally__, in order for Vue to use them
* [more](https://vuejs.org/v2/guide/components.html)

# ORGANIZING COMPONENTS (Component tree, props, events)

* It’s common for an app to be organized into a tree of nested components. The communication between the nodes of the tree happens in two way:
    * __variable data of a component can be passed down the tree via `props`__ (like a arguments for a function). Props are custom attributes you can register on a component. When a value is passed to a prop attribute, it becomes a property on that component instance
    * __a component can communicate an event (and a value) up the tree to its parent via events__: the parent can choose to listen to any event on the child component instance with `v-on`, just as we would with a native DOM event

## Props (downwards)

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

* upwards, from the component to the listening parent, with an event - [more](https://vuejs.org/v2/guide/components.html#Listening-to-Child-Components-Events)
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

### Using v-model with components

* Custom events can also be used to create custom inputs that work with v-model - [more](https://vuejs.org/v2/guide/components.html#Using-v-model-on-Components)


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

* Sometimes, it’s useful to dynamically switch between components, i.e. changing dinamically what component should be displayed (such as in a tabbed menu)
* The above is made possible by Vue’s `<component>` element with the is special attribute, to which we can pass the name of a registered component, or a component’s options object - [more](https://vuejs.org/v2/guide/components.html#Dynamic-Components)
* this attribute can be used with regular HTML elements, but with caveats 
* [full guide](https://vuejs.org/v2/guide/components-dynamic-async.html)

```html
<!-- Component changes when currentTabComponent changes -->
<component v-bind:is="currentTabComponent"></component>
```

# DOM TEMPLATE PARSING CAVEATS

* Some HTML elements, such as `<ul>`, `<ol>`, `<table>` and `<select>` have restrictions on what elements can appear inside them, and some elements such as `<li>`, `<tr>`, and `<option>` can only appear inside certain other elements: this will lead to hoisting and rendering errors
* Fortunately, the `is` special attribute offers a workaround:

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

* It should be noted that this limitation does not apply if you are using string templates from one of the following sources:
    * String templates (e.g. `template: '...'`)
    * Single-file (`.vue`) components
    * `<script type="text/x-template">`
* [more](https://vuejs.org/v2/guide/components.html#DOM-Template-Parsing-Caveats)

