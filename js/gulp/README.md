# GULP

* Gulp.js is a task runner mostly used for web-development, that let you easily automate tasks and perform operation on files, e.g.:
    * transpiling
    * minification
    * images optimizazion
    * ...etc

# INSTALL

```bash
npm install gulp-cli -g
npm init
npm install --save-dev gulp
gulp --version
```

# GULPFILE

* A gulpfile is a file in your project directory titled `gulpfile.js` (or capitalized as `Gulpfile.js`, like Makefile), that automatically loads when you run the `gulp` command. 
    * You can alternatively split up tasks into different files and put them into a `gulpfile.js` directory (project root directory): then you define an `index.js` file that will be used by gulp as the gulpfile, where you can import the other files
    * You can write a gulpfile using a language that requires transpilation, like TypeScript or Babel, by changing the extension on your gulpfile.js to indicate the language and install the matching transpiler module - [more](https://gulpjs.com/docs/en/getting-started/javascript-and-gulpfiles/#transpilation)
* Within this file, you'll often see gulp APIs, like `src()`, `dest()`, `series()`, or `parallel()` but any vanilla JavaScript or Node modules can be used. 
* __Any exported functions will be registered into gulp's task system__.
* [more](https://gulpjs.com/docs/en/getting-started/javascript-and-gulpfiles)

```js
// gulpfile.js

function defaultTask(cb) {
  // place code for your default task here
  cb(); // callback
}

exports.default = defaultTask
```

```bash
gulp # run default task
```

# TASKS

## Tasks must be asynchronous js function

* __Each gulp task must be an asynchronous JavaScript function__ - a function that either:
    * __accepts an `error-first callback`__
    * __returns either a:__ 
        * `stream`
        * `promise`
        * `event emitter`
        * `child process`
        * `observable`

### Task that accepts an Error first callbacks (most common)

* If nothing is returned from your task, you must use the error-first callback ([more](https://nodejs.org/api/errors.html#errors_error_first_callbacks)) to signal completion. The callback will be passed to your task as the only argument - e.g. named `cb()`

```js
function callbackTask(cb) {
  // `cb()` should be called by some async work
  cb();
}

exports.default = callbackTask;
```

* To indicate to gulp that an error occurred in a task using an error-first callback, call it with an `Error` as the only argument.

```js
function callbackError(cb) {
  // `cb()` should be called by some async work
  cb(new Error('kaboom'));
}

exports.default = callbackError;
```

* However, you'll often pass this callback to another API instead of calling it yourself.

```js
const fs = require('fs');

function passingCallback(cb) {
  fs.access('gulpfile.js', cb);
}

exports.default = passingCallback;
```

### Task returning a stream

* See [streams](https://nodejs.org/api/stream.html#stream_stream)

```js
const { src, dest } = require('gulp');

function streamTask() {
  return src('*.js')
    .pipe(dest('output'));
}

exports.default = streamTask;
```

### Task returning a promise (or using async-await)

* See [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises)

```js
function promiseTask() {
  return Promise.resolve('the value is ignored');
}

exports.default = promiseTask;
```

You can define your task as an `async` function, which wraps your task in a promise. This allows you to work with promises synchronously using `await` and use other synchronous code.

```js
const fs = require('fs');

async function asyncAwaitTask() {
  const { version } = JSON.parse(fs.readFileSync('package.json', 'utf8'));
  console.log(version);
  await Promise.resolve('some result');
}

exports.default = asyncAwaitTask;
```

### Task returning an event emitter

* See [event emitters](https://nodejs.org/api/events.html#events_events)

```js
const { EventEmitter } = require('events');

function eventEmitterTask() {
  const emitter = new EventEmitter();
  // Emit has to happen async otherwise gulp isn't listening yet
  setTimeout(() => emitter.emit('finish'), 250);
  return emitter;
}

exports.default = eventEmitterTask;
```

### Task returning a child process

* See [child processes](https://nodejs.org/api/child_process.html#child_process_child_process)

```js
const { exec } = require('child_process');

function childProcessTask() {
  return exec('date');
}

exports.default = childProcessTask;
```

### Task returning an observable

* See [observables](https://github.com/tc39/proposal-observable/blob/master/README.md)

```js
const { Observable } = require('rxjs');

function observableTask() {
  return Observable.of(1, 2, 3);
}

exports.default = observableTask;
```

## Public and private tasks (export)

* A task can be
    * __public (i.e. exported in gulpfile and thus runnable from the cli `gulp` command)__
    * __private (i.e. not exported in gulpfile and thus made to be used internally, usually used as part of `series()` or `parallel()` composition)__

```js
const { series } = require('gulp');

// The `clean` function is not exported so it can be considered a private task.
// It can still be used within the `series()` composition.
function clean(cb) {
  // body omitted
  cb();
}

// The `build` function is exported so it is public and can be run with the `gulp` command.
// It can also be used within the `series()` composition.
function build(cb) {
  // body omitted
  cb();
}

exports.build = build;
exports.default = series(clean, build);
```

## Task composition

* You can compose tasks using `series()` and `parallel()`
    * which accept any number of tasks or composed operations (i.e. they can be nested to any depth)

```js
// gulpfile.js
const { series, parallel } = require('gulp');

function clean(cb) {
  // body omitted
  cb();
}

function css(cb) {
  // body omitted
  cb();
}

function javascript(cb) {
  // body omitted
  cb();
}

exports.build = series(clean, parallel(css, javascript));
```

```bash
gulp --tasks
gulp build
```

Complex composition is possible by nesting tasks, `parallel()` and `series()`:

```js
// gulpfile.js
// ...
exports.build = series(
  clean,
  parallel(
    cssTranspile,
    series(jsTranspile, jsBundle)
  ),
  parallel(cssMinify, jsMinify),
  publish
);
```

Tasks are composed immediately when either `series()` or `parallel()` is called. This allows variation in the composition instead of conditional behavior inside individual tasks.

```js
// gulpfile.js

// ...
if (process.env.NODE_ENV === 'production') {
  exports.build = series(transpile, minify);
} else {
  exports.build = series(transpile, livereload);
}
```

# GLOBS (i.e. wildcards)

* `globs` are a way to refer to files using wildcards
* `/` is the path separator for gulp's globs, regardless of the operating system (thus avoid using Node's `path` methods, like `path.join`, to create globs).
    * The portion between two path separator is called a __segment__ (`./<SEGMENT>/`)
* Two or more globs that (un)intentionally match the same file are considered overlapping. When overlapping globs are used within a single `src()`, gulp does its best to remove the duplicates, but doesn't attempt to deduplicate across separate `src()` calls.

```js
'*.js' 
// index.js, but not scripts/index.js or scripts/nested/index.js

'scripts/**/*.js'
// scripts/index.js, scripts/nested/index.js, and scripts/nested/twice/index.js

['scripts/**/*.js', '!scripts/vendor/**', 'scripts/vendor/react.js']
// array globs are matched in array order:
// all js files in scripts and its subfolder 
// a part from all files in vendor subfolder
// but do add vendor/react.js 
```

* `*` - Matches 0 or more characters in a single path portion
* `?` - Matches 1 character
* `[...]` - Matches a range of characters, similar to a RegExp range. If the first character of the range is `!` or `^` then it matches `*` any character not in the range.
* `!(pattern|pattern|pattern)` - Matches anything that does not match any of the patterns provided.
* `?(pattern|pattern|pattern)` - Matches zero or one occurrence of the patterns provided.
* `+(pattern|pattern|pattern)` - Matches one or more occurrences of the patterns provided.
* `*(a|b|c)` - Matches zero or more occurrences of the patterns provided
* `@(pattern|pat*|pat?erN)` - Matches exactly one of the patterns provided
* `**` - If a "globstar" is alone in a path portion, then it matches zero or more directories and subdirectories searching for `*` matches. It does not crawl symlinked directories.
* __Dot files__: If a file or directory path portion has a `.` as the first character, then it will not match any glob pattern unless that pattern's corresponding path part also has a `.` as its first character. For example, the pattern `a/.*/c` would match the file at `a/.b/c`. However the pattern `a/*/c` would not, because `*` does not start with a dot character.

# FILE STREAMS (src, dest, pipe)

* `src(<GLOBs>)` - expects a single glob string or an array of globs to determine the source files and produces a [readable nodejs stream]((https://nodejs.org/api/stream.html#stream_types_of_streams)) (or error if no match was found), which should be returned from the task (after all transformations). 
    * can be configured to work in different ways: `buffer` (default, loads file into memory), `stream` (used to deal with large files), `empty` (used to deal with files with no content but just metadata) - [more](https://gulpjs.com/docs/en/getting-started/working-with-files#modes-streaming-buffered-and-empty)
* `.pipe(<TRANSFORMSTREAM>)` - chains [Transform or Writable nodejs streams](https://nodejs.org/api/stream.html#stream_types_of_streams), i.e. the stream on which it's called will be written or transformed and then returned
* `dest(<DIRECTORY>)` - is given an output directory string and returns a [writable nodejs stream]((https://nodejs.org/api/stream.html#stream_types_of_streams)). When passed to `pipe()` the stream on which the latter is called will be written in the given folder.
    * `symlink()` operates the same, but creates links instead of files
* See [Nodejs streams](https://nodejs.org/api/stream.html#stream_types_of_streams)

```js
const { src, dest } = require('gulp');
const babel = require('gulp-babel');
const uglify = require('gulp-uglify');
const rename = require('gulp-rename');

exports.default = function() {
  return src('src/*.js')
    .pipe(babel())
    .pipe(src('vendor/*.js'))
    .pipe(dest('output/'))
    .pipe(uglify())
    .pipe(rename({ extname: '.min.js' }))
    .pipe(dest('output/'));
}
```

# PLUGINS

* Gulp plugins are [Node Transform Streams](https://nodejs.org/api/stream.html#stream_types_of_streams) that encapsulate common behavior to transform files in a pipeline - often placed between `src()` and `dest()` using the `.pipe()` method. They can change the filename, metadata, or contents of every file that passes through the stream.
* Gulp plugins, which are available as npm modules, can be searched in [gulp's plugin search page](https://gulp.js.com/plugins)
* Plugins should always transform files. Use a (non-plugin) Node module or library for any other operations - [more](https://gulpjs.com/docs/en/getting-started/using-plugins#do-you-need-a-plugin)

```js
const { src, dest } = require('gulp');
const uglify = require('gulp-uglify');
const rename = require('gulp-rename');

exports.default = function() {
  return src('src/*.js')
    // The gulp-uglify plugin won't update the filename
    .pipe(uglify())
    // So use gulp-rename to change the extension
    .pipe(rename({ extname: '.min.js' }))
    .pipe(dest('output/'));
}
```

## Conditional plugins

Since plugin operations shouldn't be file-type-aware, you may need a plugin like gulp-if to transform subsets of files.

```js
const { src, dest } = require('gulp');
const gulpif = require('gulp-if');
const uglify = require('gulp-uglify');

function isJavaScript(file) {
  // Check if file extension is '.js'
  return file.extname === '.js';
}

exports.default = function() {
  // Include JavaScript and CSS files in a single pipeline
  return src(['src/*.js', 'src/*.css'])
    // Only apply gulp-uglify plugin to JavaScript files
    .pipe(gulpif(isJavaScript, uglify()))
    .pipe(dest('output/'));
}
```

## Inline plugins

Inline plugins are one-off Transform Streams you define inside your gulpfile by writing the desired behavior

```js
const { src, dest } = require('gulp');
const uglify = require('uglify-js');
const through2 = require('through2');

exports.default = function() {
  return src('src/*.js')
    // Instead of using gulp-uglify, you can create an inline plugin
    .pipe(through2.obj(function(file, _, cb) {
      if (file.isBuffer()) {
        const code = uglify.minify(file.contents.toString())
        file.contents = Buffer.from(code.code)
      }
      cb(null, file);
    }))
    .pipe(dest('output/'));
}
```

# WATCH API

* The `watch()` API connects globs to tasks using a file system watcher. It watches for changes to files that match the globs and executes the task when a change occurs
    * If the task doesn't signal [Async Completion](https://gulpjs.com/docs/en/getting-started/async-completion), it will never be run a second time.
* By default, the watcher executes tasks whenever a file is created, changed, or deleted. If you need to use different events, you can use the `events` option - [more](https://gulpjs.com/docs/en/getting-started/watching-files#watched-events)
* This API provides built-in delay and queueing based on most-common-use defaults. - [more](https://gulpjs.com/docs/en/getting-started/watching-files)

```js
const { watch, series } = require('gulp');

// ...

exports.default = function() {
  // You can use a single task
  watch('src/*.css', css);
  // Or a composed task
  watch('src/*.js', series(clean, javascript));
};
```

