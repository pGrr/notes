# WEBPACK 4

* Webpack simplyfies the management of static assets (js, css, images, etc) in web projects:
    * __dependency management (dependency graph)__: configuration and management of dependencies between resources, simplifying the dependency management, e.g. between js files instead of having to load them one by one in a specific order with possible errors because of wrong order, global variables overriding, functions being called before others, etc)
    * __transformations__: configuration and automation of code compiling (e.g. css preprocessors, babel/typescript, etc) 
    * __code splitting and concatenation__: We're going to require assets like images, CSS files, and Java script files and their loaded when they're needed for the page. We can also split our app into different files and just load the code that the page requires. Then when the user goes to a new page, they don't download the already downloaded code again. The code that must be used together is concatenated to reduce the number of http requests for performance (e.g. css and/or js concatenation to produce a single css file and a single js file)

# INSTALL

```bash
# installation
npm install webpack
npm install webpack-cli

# run webpack without installing
npx webpack
```

# USAGE

* `webpack` command by default will look in current directory for a `index.js` file as source file and will produce a `main.js` file as output result

```bash
webpack # if installed globally
./node_modules/.bin/webpack # if installed locally
```

Typically `webpack` command is set up as npm's `build` script in `package.json` file:

```json
"scripts": {
    "build": "./node_modules/.bin/webpack"
}
```

# WEBPACK CONFIG FILE

* `webpack.config.js` (root project folder) - is the main webpack configuration file from which the `webpack` command will load all configuration options 
    * you can use a different location and different filename, but then you have to specify it in the webpack command: `npx webpack --config my.custom.webpack.config.js` 

```js
// webpack.config.js

// node module to route created files into the proper directory
const path = require("path");

module.exports = {
    entry: "./src/index.js",
    output: {
        fileName: "main.js",
        path: path.resolve(__dirname, "dist")
    }
}
```