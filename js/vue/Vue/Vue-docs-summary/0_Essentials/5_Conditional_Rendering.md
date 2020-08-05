# CONDITIONAL RENDERING (v-if/v-elseif/v-else, v-show)

* The difference is that an element with v-show will always be rendered and remain in the DOM; v-show only toggles the display CSS property of the element.
  * v-if is “real” conditional rendering because it ensures that event listeners and child components inside the conditional block are properly destroyed and re-created during toggles.
  * v-if is also lazy: if the condition is false on initial render, it will not do anything - the conditional block won’t be rendered until the condition becomes true for the first time.
  * Generally speaking, v-if has higher toggle costs while v-show has higher initial render costs
  * So prefer v-show if you need to toggle something very often, and prefer v-if if the condition is unlikely to change at runtime
  * Using v-if and v-for together is not recommended
* [more](https://vuejs.org/v2/guide/conditional.html)

```html
<!-- v-if/v-elseif/v-else (true conditional rendering) -->
<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>

<!-- v-show (always rendered, conditional display) -->
<h1 v-show="ok">Hello!</h1>
```

* Vue tries to render elements as efficiently as possible, often re-using them instead of rendering from scratch. This isn’t always desirable though, so Vue offers a way for you to say, “These two elements are completely separate - don’t re-use them.” Add a key attribute with unique values - [more](https://vuejs.org/v2/guide/conditional.html#Controlling-Reusable-Elements-with-key)

## Conditional Groups with v-if on <template>

* Because `v-if` is a directive, it has to be attached to a single element. But what if we want to toggle more than one element? In this case we can use `v-if` on a `<template>` element, which serves as an invisible wrapper. The final rendered result will not include the `<template>` element

```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```
