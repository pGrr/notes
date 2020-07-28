# COMPUTED PROPERTIES AND WATCHERS

## Computed properties

* In-template expressions are very convenient, but they are meant for simple operations. Putting too much logic in your templates can make them bloated and hard to maintain. for any complex logic, you should use a computed property
* `methods` can be used to achieve the same result, but unlike methods a computed property will only re-evaluate when some of its reactive dependencies have changed (caching) - [more](https://vuejs.org/v2/guide/computed.html#Computed-Caching-vs-Methods)
* Also, `watch` (see watchers, below) can be used to achieve the same result, but it is more general and makes code repetitive. Computed properties should be preferred when you have some data that needs to change based on some other data - [more](https://vuejs.org/v2/guide/computed.html#Computed-vs-Watched-Property)
* [more](https://vuejs.org/v2/guide/computed.html)

```html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>
```
```js
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // a computed getter
    reversedMessage: function () {
      // `this` points to the vm instance
      return this.message.split('').reverse().join('')
    }
  }
})
```

* The function provided will be used as the getter function for the property. Computed properties are by default getter-only, but you can also provide a setter when you need it - [more](https://vuejs.org/v2/guide/computed.html#Computed-Setter)

```js
// Computed property with getters and setters
computed: {
  fullName: {
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
  }
}
```

## Watchers

* While computed properties are more appropriate in most cases, there are times when a more generic way to react to data changes is necessary. That’s why Vue provides a  through the `watch` option
* This is most useful when you want to perform asynchronous or expensive operations in response to changing data
* using the watch option allows us to perform an asynchronous operation (accessing an API), limit how often we perform that operation, and set intermediary states until we get a final answer. None of that would be possible with a computed property.
* [more](https://vuejs.org/v2/guide/computed.html#Watchers)
