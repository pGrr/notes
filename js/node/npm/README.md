# NPM - NODE PACKAGE MANAGER

# INSTALL

```bash
# docker run -it paologarroni/nodejs

FROM ubuntu

RUN apt-get update -y
RUN apt-get upgrade -y
RUN apt-get install curl -y
RUN curl -sL https://deb.nodesource.com/setup_14.x | bash -
RUN apt-get install -y nodejs

WORKDIR /
CMD bash
```

# NPM INIT

```bash
npm init # set up a project in current directory
```

* `package.json` - map of the project: author, version, dependencies, tasks... etc
    * `name` - must be unique in npm repository
    * `main` - js file that will be launched (e.g. `index.js`)
    * ...etc

# NPM INSTALL

* __Local install__ will install the package in a `node_modules` directory inside the current directory
* __Global install__ will install the package system-wide, e.g. in `/usr/local/lib/node_modules` (may change depending on the os), thus the package will be usable in any project

```bash
# INSTALL DEPENDENCIES

# locally (dev and production)
npm install <PACKAGE> [<PACKAGE>...] 
npm install <PACKAGE>[@<VERSION>] # specific version
npm install <PACKAGE>@latest # latest

# development only (not included in production builds)
npm install <PACKAGE>[@<VERSION>] --save-dev
# --no-save = do not add dependency in package.json
# --save-prod = production only
# --save-optional = mark as optional

# globally
npm install -g <PACKAGE>[@<VERSION>]
```

# NPM OUTDATED AND NPM UPDATE

```bash
# CHECK FOR OUTDATED PACKAGES
npm outdated # locally
npm outdated -g # globally

# UPDATE PACKAGE
npm update [<PACKAGE>]
npm install [<PACKAGE>]
```

# NPM UNINSTALL

```bash
npm uninstall <PACKAGE>
```

# SEMANTIC VERSIONING

* `<MAJOR>.<MINOR>.<PATCH>`
* Wildcards
    * no wildcard = `1.5.6` = exact version only
    * caret = `^1.5.6` = all minor and patches ok (`1.x.x`)
    * tilde = `~1.5.6` = all patches ok (`~1.5.x`)

## package.lock.json

* Having developers in a team building and testing the software using different package versions (because of semantic versioning with wildcards) make the software unreliable and hard to debug (e.g. a specific version of a package is breaking the software, but not all developers are using that version)
* `package.lock.json` contains the actual version currently used for this project, which should be the same for all the team: it's goal is to ensure that the same version of each package is installed for the project each time (regardless of semantic version wildcards)
* When we `npm install` a package
    * if the project contains a `package.lock.json` file, it reads from there the exact version to be installed
    * else it installs the latest version of the package compatible with the one specified in `package.json` file

# NPM CACHE VERIFY AND CLEAN

* npm keeps a cache of your install modules so that it doesn't have to get them every time. But that, sometimes, can lead to unexpected results. And worse, sometimes it doesn't work properly.
* in most cases, when you try to install the module that should be working properly and it doesn't, or when a version of a module that doesn't work, just try clearing your cache.
* __clearing npm cache should always be part of packages troubleshooting__

```bash
# check cache (run a report that verifies the cache, but it's not fool-proof)
npm cache verify 

# force cache cleaning
npm cache clean --force
```

# NPM AUDIT

* npm audit will check the dependencies of your project and make sure they are safe to use and warns if there is any issue with a package.
    * Ranks the warnings as "High" (to be addressed ASAP!), "Moderate", "Low"
    * What kind of vulnerability is that
    * what part of the package is causing it
    * in what version has it been patched
        * SEMVEER - means the patch could break the code (and your software too)
* It runs automatically on `npm install` and can also be invoked separately

```bash
npm audit # check dependencies issues

npm audit fix # fix all issues

# ...or (better):
npm install <PACKAGE> # for all packages causing problems
# (this way if a package update breaks your code then you can track who is that)
```

# NPM SCRIPTS

* npm comes with scripting features
* scripts will be defined in `package.json` file by `scripts`
* There are many pre-defined properties (e.g. `publish`, `start`, `version`, `pre-test`, `build`, etc.) that can be used to run our scripts during execution of a specific npm command ([See reference](https://docs.npmjs.com/misc/scripts))
* You can also create your own and launch it with `npm run <NAME>`

```json
// package.json
// ...
"scripts": {
    // default script "test" will be executed on "npm test" command
    "test": "echo \"Error: no test specified\" && exit 1", 
    // custom "myscript" will be executed on "npm run myscript"
    "myscript": "echo Hello world!"
}
// ...
```

# NPX

* sometimes installing CLI tools from npm packages that you use only once in order to create new projects (e.g. create-react-app) will result in package pollution in your global directories (which are rarely checked by developers)
* The main purpose of creating `npx` was to resolve these issues: it will temporarily install a package and use it to run a specified command, but without installing it nor globally nor locally. Use cases are:
    * cli-tools used only to create new projects (e.g. `create-react-app`, `angular-cli`, etc)
    * run tests (e.g. with `mocha`)
    * create custom scripts in `package.json` file that needs a package to accomplish a single task, but you don't want that package to be installed
* [more](https://www.npmjs.com/package/npx)

```bash
# NPX SYNTAX
npx --help
npx [options] <command>[@version] [command-arg]...
npx [options] [-p|--package <pkg>]... <command> [command-arg]...
npx [options] -c '<command-string>'
npx --shell-auto-fallback [shell]
```

```bash
# e.g. temporarily install angular-cli, just to create a new project
npx -p @angular/cli ng new myapp
# e.g. run mocha tests without installing mocha
npx mocha
# e.g. draw a cow that says "hello" with cowsay, without installing cowsay
npx cowsay "hello!"
```

```json
// package.json
// ...
"scripts": {
    // create a new angular app, without installing angular-cli
    "ng-new-app": "npx -p @angular/cli ng new myapp"
}
// ...
```

# NPM ALTERNATIVES (e.g. YARN)

* [__YARN__](https://yarnpkg.com/)
    * was introduced by the Facebook team. It is pretty much almost the same as npm, but usually it's faster. 
    * __It requires you to install the packager__
        * ...but this is where `npx` comes useful: __you can do `npx yarn` and it would actually run Yarn and do the install in your system without having Yarn installed__
        * The second option is called ni. It can be found at this lin
* [__NI__](https://github.com/imkimchi/ni)
    * essentially the same as npm, but with a different approach (again, you can try it without installing it by using `npx`)
* ...etc

