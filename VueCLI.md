# Vue CLI

* convenient packed cli interface to configure a vue project
  * select libraries and plugs them into the project
  * configures webpack
  * HMR - Hot Modules Replacement (live update whenever project changes)

# Install

```bash
npm install -g @vue/cli @vue/cli-service-global
# or
yarn global add @vue/cli @vue/cli-service-global
```

# Create project

* `vue create project-name`
  * you can select what libraries you wish in your app, e.g. babel, linter, vuex, router
  * will create a `project-name` folder containing our app
  * `npm run serve` - will build our app and serve it on localhost

# Vue UI

* does the same as Vue CLI but with a GUI
* `vue ui` - will open the interface on the browser
  * it can handle many projects, and create new, for each project:
    * same options as the cli
    * once installed we have sections: plugins, configuration, tasks (e.g. npn serve), build, lint, 

# Folder structure

* `node_modules` - dependencies
* `public` - assets that you don't want to be process by webpack
* `src` - code that you want to be processed by webpack
  * `assets` - images, fonts, ecc
  * `components` - vue components
  * `views` - files for each view ("page") of our app
  * `App.vue` - root component (all other components are nested within)
  * `main.js` - render the app and mounts it to the DOM
  * `router.js` - vue router file
  * `store.js` - vuex file
  * `.gitignore`, `babel.config.js`, `package.json`, ...etc

# Deployment

* `npm run build` - builds and put files into `dist` directory
  * everything will be packed into 2 js files, one for vendor code and one for our app, which will be loaded in index.html

# VS-code optimization

* `Vetur` - syntax highlighting and more (cs vode will recommend it)
  * `scaffold + ENTER` - creates scaffold for a component
  * __Emmet shortcodes__ 
    * `h1 + ENTER` - h1 element
    * `div>ul>li` - create complex structure
* `ESlint` and `Prettier` install
  * `.eslintrc.js` - config eslint
    * in `extends`, add `'plugin':prettier/recommended'`

...See more on [https://www.vuemastery.com/courses/real-world-vue-js/optimizing-your-editor](https://www.vuemastery.com/courses/real-world-vue-js/optimizing-your-editor)