# VUE CLI

* convenient packed cli interface to configure a vue project
  * select libraries and plugs them into the project
  * configure webpack
  * HMR - Hot Modules Replacement (live update whenever project changes)

## Install and upgrade

```bash
# INSTALL
npm install -g @vue/cli @vue/cli-service-global
# or
yarn global add @vue/cli @vue/cli-service-global

# SEE VERSION
vue --version

# UPGRADE VUE-CLI
npm update -g @vue/cli
# OR
yarn global upgrade --latest @vue/cli

# UPGRADE DEPENDENCIES
vue upgrade [options] [plugin-name]
```

## Instant prototyping

```bash
# Serve the current directory vue app (infers the entry file)
vue serve

# Serve a .js or .vue file in development mode with zero config
vue serve <COMPONENT>

# Build the current directory vue app (infers the entry file)
vue build

# Build a .js or .vue file in development mode with zero config
vue build <COMPONENT>
```

## Create project

* will create a folder containing our configured app
* you can select you own libraries (e.g. babel, linter, vuex, router) or use the default configuration
* Presets saved during vue create are stored in a configuration file in your user home directory (~/.vuerc)
* [see config reference](https://cli.vuejs.org/config/) for complex custom builds

```bash
vue create project <NAME> # creates a folder <NAME> containing our configured app

cd <NAME> 

npm run build # will build our app 
npm run serve # will build our app and serve it on localhost

# internally uses vue-cli-service (see package.json):
npx vue-cli-service help [command] # help
vue-cli-service build # will build our app 
vue-cli-service serve # will build our app and serve it on localhost
vue-cli-service inspect # inspect the webpack config inside a Vue CLI project
vue-cli-service lint # lint and fix source files
```

## Vue UI

```bash
vue ui
```

* does the same as Vue CLI but with an in-browser GUI
  * it can handle many projects, and create new, for each project:
    * same options as the cli
    * once installed we have sections: plugins, configuration, tasks (e.g. npn serve), build, lint, etc

## Folder structure

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

## Installing Plugins in an Existing Project

* Each CLI plugin ships with a generator (which creates files) and a runtime plugin (which tweaks the core webpack config and injects commands)
* The command resolves @vue/eslint to the full package name @vue/cli-plugin-eslint, installs it from npm, and invokes its generator.
* vue add is specifically designed for installing and invoking Vue CLI plugins. It is not meant as a replacement for normal npm packages. 

```bash
vue add eslint
# these are equivalent to the previous usage
vue add cli-plugin-eslint
```

## Browser list

You will notice a browserslist field in `package.json` (or a separate `.browserslistrc` file). [See browser compatibility ranges](https://github.com/ai/browserslist).

## Modern mode (for production builds)

```bash
vue-cli-service build --modern
```

Vue CLI will produce two versions of your app: one modern bundle targeting modern browsers that support ES modules, and one legacy bundle targeting older browsers that do not. The generated HTML file will automatically contain the code to use the correct one based on browser version. 

## Deployment

* `npm run build` - builds and put files into `dist` directory
  * everything will be packed into 2 js files, one for vendor code and one for our app, which will be loaded in index.html
  * [see deployment examples](https://cli.vuejs.org/guide/deployment.html#general-guidelines)

# VS-CODE OPTIMIZATION

* `Vetur` - syntax highlighting and more (cs vode will recommend it)
  * `scaffold + ENTER` - creates scaffold for a component
  * __Emmet shortcodes__ 
    * `h1 + ENTER` - h1 element
    * `div>ul>li` - create complex structure
* `ESlint` and `Prettier` install
  * `.eslintrc.js` - config eslint
    * in `extends`, add `'plugin':prettier/recommended'`

...See more on [https://www.vuemastery.com/courses/real-world-vue-js/optimizing-your-editor](https://www.vuemastery.com/courses/real-world-vue-js/optimizing-your-editor)

