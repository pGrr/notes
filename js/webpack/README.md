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
# SINGLE EXECUTION
webpack # if installed globally
./node_modules/.bin/webpack # if installed locally

# WATCH MODE (execute on changes)
webpack -w 
./node_modules/.bin/webpack -w

```

Typically `webpack` command is set up as npm's `build` script in `package.json` file:

```json
"scripts": {
    "build": "./node_modules/.bin/webpack -w"
}
```

# WEBPACK CONFIG FILE

* `webpack.config.js` (root project folder) - is the main webpack configuration file from which the `webpack` command will load all configuration options 
    * you can use a different location and different filename, but then you have to specify it in the webpack command: `npx webpack --config my.custom.webpack.config.js` 
* [more](https://webpack.js.org/configuration/)

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

# LOADERS

Loaders allow you to pre-process source files as you import or “load” them:

* Help load files and images
* Perform any transformation on files
* Transpile dialects (jsx, typescript, sass/less, etc into js and css)
* Load inline images as data URLs
* import CSS files directly from your JavaScript modules
* ...etc

Thus, loaders are kind of like “tasks” in other build tools and provide a powerful way to handle front-end build steps. This allows you to bundle any static resource way beyond JavaScript.

You can use [existing ones](https://webpack.js.org/loaders/) or easily write your own loaders using Node.js. - [more](https://webpack.js.org/concepts/loaders/)

## Example: babel, react, style-loader, css-loader, url-loader

E.g. let's suppose to use loaders to:

* use `babel` and `react`, thus performing es6 and jsx transpiling to js
* import css, sass/less as a module directly from js, thus only bundling the styles that the app uses, and pre-process them to achieve cross-browser vanilla css
* load images by inlining the image bundle pointed by a url and then returning it from `require`, thus reducing the number of http requests

```bash
# install loaders and the related core packages
npm install babel-loader @babel/core --save-dev
npm install react react-dom --save-dev
npm install style-loader css-loader --save-dev
npm install url-loader --save-dev

# install loaders options presets
npm install @babel/preset-env --save-dev
npm install @babel/preset-react --save-dev
```

```js
// webpack.config.js
const path = require("path");

module.exports = {
    entry: "./src/index.js",
    output: {
        filename: "main.js",
        path: path.resolve(__dirname, "dist")
    },
    devServer: {
        contentBase: path.join(__dirname, "dist"),
        port: 9000
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                exclude: /(node_modules)/,
                use: {
                    loader: 'babel-loader',
                    options: {
                        presets: ['@babel/preset-env', '@babel/preset-react']
                    }
                }
            },
            {
                test: /\.css$/,
                use: [
                    {loader: 'style-loader'},
                    {loader: 'css-loader'}
                ]
            },
            {
                test: /\.(png|jpg)$/,
                use: [
                    {loader: 'url-loader'}
                ]
            }
        ]
    }
}
```

`.babelrc` config file (in root folder):

```json
{
    "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```

```js
// index.js

import React from 'react'
import { render } from 'react-dom'
import './style.css'

const Greeting = () => {
    return (
        <div>
            <h1>Hello from React</h1>
            <div id="image"></div>
        </div>
    )
}

render(
    <Greeting />,
    document.getElementById('target')
)
```

```css
/* 
greeting.css 
...css will be bundled (i.e. pre-processed and put as static assed in dist folder)
...the image given as url will be inlined (i.e. made binary and returned from require in js)
*/
h1 {
    font-family: Arial;
    color: blueviolet;
}

#image {
    background: url('ski-day.jpg');
    height: 500px;
    width: 500px;
}
```

# CODE SPLITTING

* Code splitting allows you to break up your code into different bundles by setting multiple entry points
    * For example, an application that has an About-us and a Contact-us page, we can tell Webpack to create bundles for each of these, and then only the resources necessary for the page will be loaded. This reduces the size of the request and if somebody visits the site and only goes to the About page, they won't load the Contact us information. Additionally, we can use Webpack to create bundles for shared code and for vendor code.

```js
// webpack.config.js

const path = require("path");

module.exports = {
    entry: {
        about: "./src/about.js", // => about.bundle.js
        contact: "./src/contact.js" // => contact.bundle.js
    },
    output: {
        filename: "[name].bundle.js",
        path: path.resolve(__dirname, "dist")
    },
    // ...
}
```

# PLUGINS

* The `plugins` option in `webpack.config.js` is used to customize the webpack build process in a variety of ways. webpack comes with a variety built-in plugins available under `webpack.[plugin-name]`
* See [Plugins page](https://webpack.js.org/plugins) for a list of plugins and documentation but note that there are a lot more out in the community.

## Automatic html file generation with HtmlWebpackPlugin

* Instead of manually writing an html file that links to your bundle js files, you can automatically generate those with `HtmlWebpackPlugin`
    * If you have multiple webpack entry points, they will all be included with `<script>` tags in the generated HTML.
* [more](https://webpack.js.org/plugins/html-webpack-plugin/)

```bash
npm install html-webpack-plugin --save-dev
```

```js
// webpack.config.js

const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

module.exports = {
    // ...
    plugins: [new HtmlWebpackPlugin()],
    // ...
}
```

# OPTIMIZATION

* Since version 4 webpack runs optimizations for you depending on the chosen mode, still all optimizations are available for manual configuration and overrides.
* These are set using `webpack.config.js` in the `optimization` option, e.g.:
    * `optimization.minimize` - boolean (true by default)
    * `optimization.minimizer` - plugin used for minification
    * `optimization.splitChunks` - `all`, `initial`, `async`, etc 
    * `optimization.concatenateModules` - boolean (true by default)
* Many optimizations are possible: [see more](https://webpack.js.org/configuration/optimization/)

## Chunk splitting with splitChunks

* With bundle splitting, you can push the vendor dependencies to a bundle of their own and benefit from client level caching. - [more](https://webpack.js.org/guides/code-splitting/)
* `splitChunks` - it's a webpack plugin, which is going to look for any repeated code between my two bundles and it's going to create a vendor bundle for that
    * To give you a quick example, instead of having main.js (100 kB), you could end up with main.js (10 kB) and vendor.js (90 kB). Now changes made to the application are cheap for the clients that have already used the application earlier.
    * e.g. all react components have react, so webpack splitChunks is going to create a bundle just for that vendor code
    * bundle files of components will now contain only component's code whereas react will be in a separate file
    * in the html page of a component, you link both the bundle files, the one of the component and the vendor one. But since they are now separate files and the vendor one is repeated between multiple html pages, it will be sent only the first one and then will be cached client-side
    
```js
// webpack.config.js

const path = require("path");

module.exports = {
    // ...
    optimization: {
        splitChunks: {
            chunks: 'all' // initial, async, etc
        }
    },
    // ...
}
```

## Minification with uglify.js

```bash
npm install uglifyjs-webpack-plugin --save-dev
```

```js
// webpack.config.js

const path = require("path");
const UglifyJsPlugin = require("uglifyjs-webpack-plugin");

module.exports = {
    // ...
    optimization: {
        minimizer: [new UglifyJsPlugin()]
    }
    // ...
}
```

# WEBPACK-DEV-SERVER

* Development server that will make your project available your project (i.e. the static bundle assets) in localhost at the specified port and live-reload them when they change
    * It uses `node.js`, `express.js` to realize the server, which uses the `webpack-dev` middleware to serve a webpack bundle and uses and `socket.io` to listen for any changes

```bash
# Install module
npm install webpack-dev-server --save-dev
```

```js
// webpack.config.js

const path = require('path');

module.exports = {
    // ...
    devServer: {
        contentBase: path.join(__dirname, "dist"),
        port: 9000
    }
    // ...
}
```

...in `package.json` file:

```json
{
    "scripts": {
        "build": "./node_modules/.bin/webpack -w",
        "start:dev": "./node_modules/.bin/webpack-dev-server"
    }
}
```

...and then start it with:

```bash
npm run start:dev
```
