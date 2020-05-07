# Install

* via npm or cdn

# The View instance

* The vue instance is "the heart" of the application
  * it binds to the element `el`
  * on that element, any referenced `data` variable (e.g. `{{ product }}`, which references `app.data.product`) is changed reactively

```js
//js
var app = new Vue({
    el: '#app',
    data: {
        product: 'Socks',
    }
});
// html
```
```html
<h1 id="app">{{ product }}</h1>
```

# v-bind

* you can bind an html element attribute to a vue's data variable with `v-bind`
  * `<img v-bind:src="image">` equals to `<img src="{{ image }}">`
* short syntax is just `:`:
  * `<img :src="image">`
* Using v-bind you can make any property a variable (href, title, class, style, etc)

# v-x directives

## v-if v-elseif v-else

```html
<!-- add or remove to the dom conditionally one or the other basing on inventory variable -->
<p v-if="inventory > 10">In stock</p>
<p v-else-if="inventory <= 10 && inventory > 0">Almost sold out!</p>
<p v-else>Not available</p>
```

## v-show

```html
<!-- v-show toggle hidden property basing on the condition value -->
<p v-show="inStock">In stock.</p>
```

## v-for 

* When using v-for it is recommended to give each rendered element its own unique key, and thus helping vue to diffs the list in order to reuse and reorder existing elements

```html
<!-- Loops over the elements variable (create multiple li) -->
<ul>
    <li v-for="element in elements" :key="element.id"> {{ element.name }} </li>
</ul>
```