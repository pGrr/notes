# SLOT CONTENT

* Just like with HTML elements, it’s often useful to be able to pass content to a component - [more](https://vuejs.org/v2/guide/components.html#Content-Distribution-with-Slots)
* we just add the `<slot></slot>` where we want it to go – and that’s it. We’re done! - [more (full guide on slots)](https://vuejs.org/v2/guide/components-slots.html)
* If we want to provide a default value we just place it inside: `<slot>Default value</value>` - [more](https://vuejs.org/v2/guide/components-slots.html#Fallback-Content)
* the slot tag will be replaced with the content passed inside a component, or its default value
* If template did not contain a `<slot>` element, any content provided between its opening and closing tag would be discarded
* Slots can contain any template code, including html or other components
* [more](https://vuejs.org/v2/guide/components-slots.html#Slot-Content)

```html
<navigation-link url="/profile">
  <!-- Use a component to add an icon -->
  <font-awesome-icon name="user"></font-awesome-icon>
  Your Profile
</navigation-link>
```
```html
<!-- navigation-link template -->
<a
  v-bind:href="url"
  class="nav-link"
>
  <slot>Default value</slot>
</a>
```

## COMPILATION SCOPE

* Everything in the parent template is compiled in parent scope; everything in the child template is compiled in the child scope. - [more](https://vuejs.org/v2/guide/components-slots.html#Compilation-Scope)
* There is a way to achieve that, though: scoped slots (or slot props) -> see below

```html
<navigation-link url="/profile">
  Logged in as {{ user.name }} 
  <!-- user.name will be rendered, as it is part of the current (parent) scope -->

  Clicking here will send you to: {{ url }}
  <!--
  The `url` will be undefined, because this content is passed _to_ <navigation-link>, rather than defined _inside_ the <navigation-link> component.
  -->
</navigation-link>
```

# SCOPED SLOTS (Slot props)

* Sometimes, it’s useful for __slot content to have access to data only available in the child component__
* This is achieved through scoped slots (slot props):
    * adding `v-bind` to the `slot` tag, to bind a attribute to that slot (this attribute is called slot-prop)
    * now in the parent scope we can use `v-slot` with a value to define a name for the slot props we’ve been provided
* [more](https://vuejs.org/v2/guide/components-slots.html#Scoped-Slots)

```html
<!-- child template -->
<span>
  <slot v-bind:user="user">
    {{ user.lastName }}
  </slot>
</span>
```
```html
<!-- parent template -->
<current-user>
  <template v-slot:default="slotProps">
    {{ slotProps.user.firstName }}
  </template>
</current-user>
```

## Abbreviated Syntax for Lone Default Slots

* when only the default slot is provided content, an abbreviated syntax can be used: the component’s tags can be used as the slot’s template and the default name can be omitted:

```html
<current-user v-slot="slotProps">
  {{ slotProps.user.firstName }}
</current-user>
```

## Destructuring Slot Props

* Internally, scoped slots work by wrapping your slot content in a function passed a single argument. That means the value of v-slot can actually accept any valid JavaScript expression that can appear in the argument position of a function definition. This opens to many possibilities:

```html
<!-- Object destructuring (ES2015) -->
<current-user v-slot="{ user }">
  {{ user.firstName }}
</current-user>
```
```html
<!-- Renaming -->
<current-user v-slot="{ user: person }">
  {{ person.firstName }}
</current-user>
```
```html
<!-- Fallback -->
<current-user v-slot="{ user = { firstName: 'Guest' } }">
  {{ user.firstName }}
</current-user>
```

# NAMED SLOTS (multiple slots)

* the `<slot>` element has a special attribute, `name`, which can be used to define additional slots
    * if not specified, the default name is `default`
* To provide content to named slots, we can use the `v-slot` directive on a `<template>`, providing the name of the slot as `v-slot`‘s argument
    * Any content not wrapped in a `<template>` using `v-slot` is assumed to be for the `default` slot
    * `v-slot` can only be used in a `template`

```html
<div class="container">
  <header>
    <slot name="header"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```
```html
<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```

## Dynamic slot names

* Dynamic directive arguments also work on v-slot, allowing the definition of dynamic slot names - [more](https://vuejs.org/v2/guide/components-slots.html#Dynamic-Slot-Names)

```html
<base-layout>
  <template v-slot:[dynamicSlotName]>
    ...
  </template>
</base-layout>
```

## Named slots shorthand syntax

* Similar to v-on and v-bind, v-slot also has a shorthand, replacing everything before the argument (v-slot:) with the special symbol `#`. For example, `v-slot:header` can be rewritten as `#header` - [more](https://vuejs.org/v2/guide/components-slots.html#Named-Slots-Shorthand)

```html
<base-layout>
  <template #header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template #footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```

```html
<current-user #default="{ user }">
  {{ user.firstName }}
</current-user>
```

# SLOT USAGE EXAMPLES

[link](https://vuejs.org/v2/guide/components-slots.html#Other-Examples)

# DEPRECATED SYNTAX < 2.6

[link](https://vuejs.org/v2/guide/components-slots.html#Deprecated-Syntax)

