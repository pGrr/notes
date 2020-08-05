# WEBPACK 4

* Webpack simplyfies the management of static assets (js, css, images, etc) in web projects:
    * __dependency management (dependency graph)__: configuration and management of dependencies between resources, simplifying the dependency management, e.g. between js files instead of having to load them one by one in a specific order with possible errors because of wrong order, global variables overriding, functions being called before others, etc)
    * __transformations__: configuration and automation of code compiling (e.g. css preprocessors, babel/typescript, etc) 
    * __code splitting and concatenation__: We're going to require assets like images, CSS files, and Java script files and their loaded when they're needed for the page. We can also split our app into different files and just load the code that the page requires. Then when the user goes to a new page, they don't download the already downloaded code again. The code that must be used together is concatenated to reduce the number of http requests for performance (e.g. css and/or js concatenation to produce a single css file and a single js file)

# INSTALL

```bash
npm install webpack[@version]
```