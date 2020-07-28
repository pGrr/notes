# TEMPLATE SYNTAX: Interpolation and Directives

* Vue.js uses an HTML-based template syntax that allows you to declaratively bind the rendered DOM to the underlying Vue instance’s data
* Combined with the reactivity system, Vue is able to intelligently figure out the minimal number of components to re-render and apply the minimal amount of DOM manipulations when the app state changes
* [more](https://vuejs.org/v2/guide/syntax.html)

```html
<!-- TEXT (MOUSTACHE SYNTAX: {{ * }} ) -->
<span>Message: {{ msg }}</span>

<!-- IMMUTABILITY (v-once) -->
<span v-once>This will never change: {{ msg }}</span>

<!-- HTML (v-html) -->
<p>Using v-html directive: <span v-html="rawHtml"></span></p>

<!-- ATTRIBUTES (v-bind) -->
<div v-bind:id="dynamicId"></div>
<button v-bind:disabled="isButtonDisabled"> Button</button>

<!-- VALID SYNTAX: EXPRESSIONS -->
{{ number + 1 }}
{{ ok ? 'YES' : 'NO' }}
{{ message.split('').reverse().join('') }}
<div v-bind:id="'list-' + id"></div>
<!-- this is a statement, not an expression: -->
{{ var a = 1 }}
<!-- flow control won't work either, use ternary expressions -->
{{ if (ok) { return message } }}
<!-- Note: Template expressions are sandboxed and only have access to a whitelist of globals such as Math and Date. You should not attempt to access user defined globals in template expressions. -->

<!-- DIRECTIVES arguments and modifiers) -->
<p v-if="seen">Now you see me</p>
<!-- with arguments -->
<a v-bind:href="url"> ... </a>
<a v-on:click="doSomething"> ... </a>
<!-- with arguments and modifiers -->
<form v-on:submit.prevent="onSubmit"> ... </form>
<!-- Dynamic Argument Expression Constraints (see docs)-->
<a v-bind:[attributeName]="url"> ... </a>

<!-- DIRECTIVES SHORTHANDS (v-bind = :, v-on = @) -->
<a :href="url"> ... </a>
<a :[key]="url"> ... </a>
<a @click="doSomething"> ... </a>
<a @[event]="doSomething"> ... </a>
```
