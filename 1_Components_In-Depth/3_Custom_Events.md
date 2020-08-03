# EVENT NAMES

* Unlike components and props, event names don’t provide any automatic case transformation. Instead, the name of an emitted event must exactly match the name used to listen to that event. - [more](https://vuejs.org/v2/guide/components-custom-events.html#Event-Names)

# CUSTOMIZING COMPONENT v-model

* By default, `v-model` on a component uses `value` as the prop and `input` as the event, but some input types such as checkboxes and radio buttons may want to use the `value` attribute for a different purpose. Using the `model` option can avoid a conflict in such cases

```js
Vue.component('base-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    checked: Boolean
  },
  template: `
    <input
      type="checkbox"
      v-bind:checked="checked"
      v-on:change="$emit('change', $event.target.checked)"
    >
  `
})
```
```html
<!-- the value of lovingVue will be passed to the checked prop. The lovingVue property will then be updated when <base-checkbox> emits a change event with a new value. -->
<base-checkbox v-model="lovingVue"></base-checkbox>
```

# BINDING NATIVE EVENTS TO COMPONENTS

* There may be times when you want to listen directly to a native event on the root element of a component. In these cases, you can use the `.native` modifier for `v-on`
* but it’s not a good idea when you’re trying to listen on a very specific element, like an `<input>.`
* Vue provides a `$listeners` property containing an object of listeners being used on the component. Using the `$listeners` property, you can forward all event listeners on the component to a specific child element with `v-on="$listeners"`
* This way you can make, e.g. the `<base-input>` component, a fully transparent wrapper, meaning it can be used exactly like a normal element, e.g. `<input>`: all the same attributes and listeners will work, without the `.native` modifier
* [more](https://vuejs.org/v2/guide/components-custom-events.html#Binding-Native-Events-to-Components)

# .sync MODIFIER (two-way props binding pattern)

* In some cases, we may need “two-way binding” for a prop. Unfortunately, true two-way binding can create maintenance issues, because child components can mutate the parent without the source of that mutation being obvious in both the parent and the child.
* That’s why instead, we recommend emitting events in the pattern of `update:myPropName`

```js
this.$emit('update:title', newTitle)
```
```html
<text-document
  v-bind:title="doc.title"
  v-on:update:title="doc.title = $event"
></text-document>
```

* Vue offer a shorthand for this pattern with the `.sync` modifier: - [more](https://vuejs.org/v2/guide/components-custom-events.html#sync-Modifier)

```html
<text-document v-bind:title.sync="doc.title"></text-document>
```