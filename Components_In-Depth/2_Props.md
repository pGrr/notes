## PROPS CASING

* when you’re using in-DOM templates, camelCased prop names need to use their kebab-cased (hyphen-delimited) equivalents. if you’re using string templates, this limitation does not apply. - [more](https://vuejs.org/v2/guide/components-props.html#Prop-Casing-camelCase-vs-kebab-case)

# PROPS TYPING

* props can be declared as a list of strings, this way they can be of any type. But you can type-hint them if you want them to be of a specific type. This not only documents your component, but will also warn users in the browser’s JavaScript console if they pass the wrong type. 

```js
// props list
props: ['title', 'likes', 'isPublished', 'commentIds', 'author']

// ...vs type-hinted props object:
props: {
  title: String,
  likes: Number,
  isPublished: Boolean,
  commentIds: Array,
  author: Object,
  callback: Function,
  contactsPromise: Promise // or any other constructor
}
```

## Prop validation and type checks

* Components can specify requirements for their props, such as the types you’ve already seen. If a requirement isn’t met, Vue will warn you in the browser’s JavaScript console. This is especially useful when developing a component that’s intended to be used by others.
* props are validated before a component instance is created
* [more](https://vuejs.org/v2/guide/components-props.html#Prop-Validation)

```js
Vue.component('my-component', {
  props: {
    // Basic type check (`null` and `undefined` values will pass any type validation)
    propA: Number,
    // Multiple possible types
    propB: [String, Number],
    // Required string
    propC: {
      type: String,
      required: true
    },
    // Number with a default value
    propD: {
      type: Number,
      default: 100
    },
    // Object with a default value
    propE: {
      type: Object,
      // Object or array defaults must be returned from
      // a factory function
      default: function () {
        return { message: 'hello' }
      }
    },
    // Custom validator function
    propF: {
      validator: function (value) {
        // The value must match one of these strings
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }

    // Constructor function (pre-defined object)
    // validate that the value of the author prop was created with new Person
    propG: Person,
  }
})

function Person (firstName, lastName) {
  this.firstName = firstName
  this.lastName = lastName
}
```


# PASSING STATIC OR DYNAMIC PROPS (props + v-bind)

* [more](https://vuejs.org/v2/guide/components-props.html#Passing-Static-or-Dynamic-Props)

```html
<!-- prop: interpreted as string -->
<blog-post title="My journey with Vue"></blog-post>

<!-- v-bind -->

<!-- if the passed value is a data variable, that variable will be dinamically bound to the prop -->
<blog-post v-bind:title="post.title"></blog-post>
<blog-post v-bind:likes="post.likes"></blog-post>
<blog-post v-bind:is-published="post.isPublished"></blog-post>

<!-- ...else it will be interpreted as data-scoped js expression -->
<blog-post
  v-bind:title="post.title + ' by ' + post.author.name"
></blog-post>

<!-- ...so too pass a non-string as a prop, even if we are passing a static value we need to add v-bind to tell Vue that this is a js expression (e.g. number, boolean, array, object, etc) rather than a string -->
<blog-post v-bind:likes="42"></blog-post>
<blog-post v-bind:comment-ids="[234, 266, 273]"></blog-post>
<blog-post
  v-bind:author="{
    name: 'Veronica',
    company: 'Veridian Dynamics'
  }"
></blog-post>
```

## Passing the Properties of an Object

If you want to pass all the properties of an object as props, you can use `v-bind` without an argument (`v-bind` instead of `v-bind:prop-name`). For example, given a post object:

```js
// given a post object:
post: {
  id: 1,
  title: 'My Journey with Vue'
}
```
```html
<!-- ...the following template: -->
<blog-post v-bind="post"></blog-post>

<!-- ...will be equivalent to: -->
<blog-post
  v-bind:id="post.id"
  v-bind:title="post.title"
></blog-post>
```

# ONE WAY DATA FLOW

* All props form a one-way-down binding between the child property and the parent one: when the parent property updates, it will flow down to the child, but not the other way around. This prevents child components from accidentally mutating the parent’s state, which can make your app’s data flow harder to understand.
* Every time the parent component is updated, all props in the child component will be refreshed with the latest value.
* This means __you should not attempt to mutate a prop inside a child component__. If you do, Vue will warn you in the console.
    * E.g. objects and arrays in JavaScript are passed by reference, if the prop is an array or object, mutating the object or array inside the child component will affect parent state!
* __To avoid this, just copy the prop value and use it in a internal data property__
    * Need to pass in an initial value the child component wants to use as a local data property? Then use an additional data property, initialized with the prop value
    * Need to pass a raw value that needs to be transformed? Then it’s best to define a computed property using the prop’s value.
* [more](https://vuejs.org/v2/guide/components-props.html#One-Way-Data-Flow)

# NON-PROPS ATTRIBUTES

* A non-prop attribute is an attribute that is passed to a component, but does not have a corresponding prop defined: the non-prop attribute will automatically be added to the root element of the component - [more](https://vuejs.org/v2/guide/components-props.html#Non-Prop-Attributes)
* For most attributes, the value provided to the component will replace the value set by the component. So for example, passing `type="text"` will replace `type="date"` and probably break it! Fortunately, __the class and style attributes are a little smarter, so values are merged__ - [more](https://vuejs.org/v2/guide/components-props.html#Replacing-Merging-with-Existing-Attributes)
* If you do not want the root element of a component to inherit attributes, you can disable it - [more](https://vuejs.org/v2/guide/components-props.html#Disabling-Attribute-Inheritance)

