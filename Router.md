# Router

# Install

* Via npm or vue-cli or vue-ui

```js
// router.js
import Vue from "vue";
import Router from "vue-router";
// vue components used as views
// (best practice is to place them in components dir)
import Home from "./views/Home.vue";
import About from "./views/About.vue";

Vue.use(Router);
export default new Router({
    routes: [
        {
            path: '/',
            name: 'home',
            component: Home // component
        },
        {
            path: '/about-us',
            name: 'about',
            component: About

        },
    ]
});

// main.js
import Vue from "vue";
import App from "./App.vue";
import router from "./router";
import store from "./store";

new Vue({
    router: router,
    store: store,
    render: h => h(App)
}).$mount("#app");

// App.vue
<template>
    <div id="app">
        <div id="nav">
            <router-link to="{ name: 'home' }">Home</router-link>
            <router-link to="{ name: 'about' }">About</router-link>
        </div>
        <router-view/>
    </div>
</template>
```

# Redirect

```js
routes: [
    {
        path: '/',
        name: 'home',
        component: Home // component
    },
    {
        path: '/about-us',
        name: 'about',
        component: About
        // alias (use the same component but on '/about' url)
        alias: "/about"
    },
    {
        path: '/about',
        // real redirect
        redirect: { name: 'about' }
    }
]
```

# Dynamic route

* `:` in a route means what follows it's a variable
* such variables can be accessed via `$route.params` (state of the routes)
* or can be sent to the component props via `props:true` (best-practice)

```js
// router.js
routes: [
    {
        path: '/user/:username', // dynamic route
        name: 'user',
        component: User,
        props: true,
    }
]

// User.vue
<template>
    <div class="user">
        <h1> Not using props: {{ $route.params.username }}.</h1>
        <h1> Using props: {{ username }}.</h1>
        <router-view/>
    </div>
</template>
```

# Remove the hash from the url

* __hash mode__ is the default mode for vue router: it uses the hash to simulate a full url so that the page won't be reloaded when the URL changes
* `mode: history` - tells the browser to use the browser's `history.pushstate` API to change the url without reloading the page, i.e. tells the server to return the root path, for any url 
* Server configuration is needed (see the docs)
* there is a caveat: 404 errors will not be provided by the server! (check the docs)

```js
export default new Router({
    mode: "history",
    routes: [...],
});
```