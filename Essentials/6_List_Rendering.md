# LIST RENDERING

## Mapping an Array to Elements with `v-for`

* Inside `v-for` blocks we have full access to parent scope properties. `v-for` also supports an optional second argument for the index of the current item
* You can also use `of` as the delimiter instead of `in`, so that it is closer to JavaScript’s syntax for iterators

```html
<ul id="example-2">
  <li v-for="(item, index) in items">
    {{ parentMessage }} - {{ index }} - {{ item.message }}
  </li>
</ul>
```
```js
var example2 = new Vue({
  el: '#example-2',
  data: {
    parentMessage: 'Parent',
    items: [
      { message: 'Foo' },
      { message: 'Bar' }
    ]
  }
})
```

## v-for with an object

* You can also use v-for to iterate through the properties of an object - [more](https://vuejs.org/v2/guide/list.html#v-for-with-an-Object)
  * When iterating over an object, the order is based on the enumeration order of Object.keys(), which is not guaranteed to be consistent across JavaScript engine implementations
* [more](https://vuejs.org/v2/guide/list.html#v-for-with-an-Object)

```js
new Vue({
  el: '#v-for-object',
  data: {
    object: {
      title: 'How to do lists in Vue',
      author: 'Jane Doe',
      publishedAt: '2016-04-10'
    }
  }
})
```
```html
<div v-for="(value, name, index) in object">
  {{ index }}. {{ name }}: {{ value }}
</div>
```

## Maintaining state

* When Vue is updating a list of elements rendered with v-for, by default it uses an “in-place patch” strategy. If the order of the data items has changed, instead of moving the DOM elements to match the order of the items, Vue will patch each element in-place and make sure it reflects what should be rendered at that particular index.
  * This default mode is efficient, but only suitable when your list render output does not rely on child component state or temporary DOM state (e.g. form input values).
* __To give Vue a hint so that it can track each node’s identity, and thus reuse and reorder existing elements, you need to provide a unique `key` attribute for each item__
  * __Don’t use non-primitive values like objects and arrays as v-for keys. Use string or numeric values instead__
  * It is recommended to provide a key attribute with v-for whenever possible, unless the iterated DOM content is simple, or you are intentionally relying on the default behavior for performance gains
* [more](https://vuejs.org/v2/guide/list.html#Maintaining-State)

```html
<div v-for="item in items" v-bind:key="item.id">
  <!-- content -->
</div>
```

## Array change detection

* Vue wraps an observed array’s __mutation methods__ so they will also trigger view updates. The wrapped methods are: `push()`, `pop()`, `shift()`, `unshift()`, `splice()`, `sort()`, `reverse()`
* In comparison, there are also __non-mutating methods__, e.g. `filter()`, `concat()` and `slice()`, which do not mutate the original array but always return a new array (so if you want to change the original array you have to assign the returned value to it)
* Due to limitations in JavaScript, there are types of changes that Vue cannot detect with arrays and objects - [more](https://vuejs.org/v2/guide/reactivity.html#Change-Detection-Caveats)

## Displaying Filtered/Sorted Results

* Sometimes we want to display a filtered or sorted version of an array without actually mutating or resetting the original data. In this case, you can create a computed property that returns the filtered or sorted array
* In situations where computed properties are not feasible (e.g. inside nested v-for loops), you can use a method
* [more](https://vuejs.org/v2/guide/list.html#Displaying-Filtered-Sorted-Results)

## v-for with a Range

* v-for can also take an integer. In this case it will repeat the template that many times.

```html
<div>
  <span v-for="n in 10">{{ n }} </span>
</div>
```

* Similar to template `v-if`, you can also use a `<template>` tag with `v-for` to render a block of multiple elements. For example:

```html
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```

## v-for with v-if

* it’s not recommended to use v-if and v-for together. When they exist on the same node, `v-for` has a higher priority than `v-if`

## v-for with a Component

* You can directly use v-for on a custom component, like any normal element. However, this won’t automatically pass any data to the component, in order to pass the iterated data into the component, we should also use props
* `is="todo-item"` is necessary in DOM templates, because only an `<li>` element is valid inside a `<ul>`. It does the same thing as `<todo-item>`, but works around a potential browser parsing error - [more](https://vuejs.org/v2/guide/components.html#DOM-Template-Parsing-Caveats)
* [more](https://vuejs.org/v2/guide/list.html#v-for-with-a-Component)

