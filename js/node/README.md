# NODE.JS

## Apache vs Node.js

* __Apache = multi-threaded, synchronous (blocking) I/O__
    * For each request a single thread is created and handles the request in a synchronous blocking way (e.g. waiting for an I/O filesystem request): each thread is blocked while waiting for the response to be ready. Then the thread finishes.
    * A lot of requests = a lot of blocking threads simultaneously = less scalability 
        * Each request is served immediately but will require more resources and the bottleneck will be on the filesystem side anyway
    * E.g. like a restaurant where for every client (request) a single waiter (thread) is hired (created). The waiter goes to the kitchen and orders the food (e.g. file-system request), and stands there waiting until the food is ready. That same client won't be able to order a glass of water until the food is ready. When the food is ready, the waiter serves it to the client (response), and then the glass of water is requested (and so on). When there are no more requests by that client, that waiter is fired (thread ends). A lot of clients would mean hiring a lot of waiters (creating many threads, resource-consuming), which is good because each client (request) is served immediately, but all waiters will be waiting for a long time until their food is ready (a system may have too many threads if the requests rate is higher than the time needed to respond, i.e. to access the filesystem, so it doesn't scale efficiently).
* __Nodejs = Single-threaded, async (Non-blocking), event-driven I/O__
    * Events (requests/responses/etc) are raised and recorded in an event-queue and handled (by executing a task) in order they were raised, by a single thread, in a non-blocking async way 
    * So a single thread is shared by all tasks (events: requests, responses, etc), but it can multitask and it never blocks (e.g. for I/O).
    * E.g. like a restaurant where there is only one waiter (single-threaded). The waiter place the order for the food in the kitchen (e.g. filesystem request), but then he immediately comes back to serve another client (request) without waiting for the food to be ready. When the food is ready the chef rings a bell (event). When the waiter goes back to the kitchen he places the second order and then brings the food to the client of the first order (event callback). Then he takes another order, and so on. When there are a lot of requests the single waiter is very busy, but he can multitask. Also, you can easily scale it, by duplicating the instance (and this is precisely how node.js apps are hosted in the cloud)

## When is Node.js good and when it's not

* So Node.js is a good for:
    * Web applications that have to handle a lot of I/O requests and/or have to be able to scale well
* ...and is not-so-good for:
    * A CPU-Heavy Application (node.js is fast when handling a lot of I/O requests, not very efficient for actual computations)
    * A Simple CRUD (or HTML) Application without tons of requests(might be only "slightly" better)
    * For security-critical or Relational Database-Backed Server-Side App (because node.js simply is not mature enough yet and might be unreliable or dangerous for security issues or rdbms-related module bugs)

# INSTALL

[See docs](https://nodejs.dev/)

```bash
node -v
```

# USAGE

```js
// myFile.js
console.log("Hello World!");
```

```bash
node myFile
```

# THE GLOBAL OBJECT

* everything on the `global` object is available to us globally, i.e. the global object contains all of the objects, values, and methods that we can use in a Node.js file without having to import anything, e.g. `__dirname`, `__filename`, `console`, `module`, `exports`, `require()`, etc
* [more](https://nodejs.org/api/globals.html)

```js
// console
console.log("Hello World!"); // ...is the same as:
global.console.log("Hello World!")

// directory name
console.log(__dirname);

// filename 
console.log(__filename); 

// process
console.log(process.pid);
console.log(process.argv);
console.log(process.versions.node);

// ...etc
```

## require() - commonjs module pattern

* You can import
    * core modules, which are shipped with node.js but not included in the global object so they must be imported
    * node npm packages
    * your own js files

```js
// path module is shipped with node.js but must be imported
const path = require("path");
console.log(`The file name is ${path.basename(__filename)}`);
// ...or with destructuring:
const { basename } = require("path");
console.log(`The file name is ${basename(__filename)}`);

// util modules is shipped with node.js but must be imported
const util = require("util");
util.log(path.basename(__filename)); // log with automatic timedate
// ...or with destructuring:
const { log } = require("util");
log(path.basename(__filename));

// v8 modules is shipped with node.js but must be imported
const v8 = require("v8");
util.log(v8.getHeapStatistics());
// ...or with destructuring:
const { getHeapStatistics } = require("util");
getHeapStatistics();
```

## process

* The `process` object is part of the global object (thus available globally), and provides:
    * Information about the process (pid, node version, etc)
    * Access to environment variables and informations
    * Access to CLI arguments
    * Communicate with the terminal or parent processes through stdin/stdout/stderr
    * Exit the current process
    * ...etc

```js
// pid
console.log(process.pid); // pid
console.log(process.argv); // arg
console.log(process.versions.node);
```

### process.argv - CLI arguments

```js
// firstSecond.js
// Arguments by order
const [, , first, second] = process.argv;
console.log(`First arg is ${first}, second is ${second}`);
```
```bash
node firstSecond firstArg secondArg
```

```js
// greeting.js (arguments with flags)

const grab = flag => {
    let indexAfterFlag = process.argv.indexOf(flag) + 1;
    return process.argv[indexAfterFlag];
}

const greeting = grab("--greeting");
const user = grab("--user");

console.log(`${greeting}, ${user}!`);
```
```bash
node greeting --user ned --greeting "Hidely Hoe"
```

### process stdin / stdout / stderr

```js
// process.stdout is a Writable stream that writes to stdout:
process.stdout.write("Hello ");
process.stdout.write("World \n\n\n");

// process.stderr is a Writable stream that writes to stderr:
process.stderr.write("Error!!");

// process.stdin is a Readable stream that reads from stdin:
process.stdin.on("data", data => {
    // do something with data
    // ...and do something to stop processing input when a condition is met
});
```

#### Question/answer example

```js
const questions = [
  "What is your name?",
  "What would you rather be doing?",
  "What is your preferred programming language?"
];

const ask = (i = 0) => {
  process.stdout.write(`\n\n\n ${questions[i]}`);
  process.stdout.write(` > `);
};

ask();

const answers = [];
process.stdin.on("data", data => {
  answers.push(data.toString().trim());

  if (answers.length < questions.length) {
    ask(answers.length);
  } else {
    process.exit();
  }
});

process.on("exit", () => {
  const [name, activity, lang] = answers;
  console.log(`
  
Thank you for your anwsers.

Go ${activity} ${name} you can write ${lang} code later!!!

  
  `);
});
```

## Timing: setTimeout(), clearTimeout(), setInterval(), clearInterval()

* They are async functions and work the same way they do in the browser - [more](https://www.w3schools.com/js/js_timing.asp)

```js
const waitTime = 5000;
const waitInterval = 500;
let currentTime = 0;

const incTime = () => {
  currentTime += waitInterval;
  const p = Math.floor((currentTime / waitTime) * 100);
  process.stdout.clearLine();
  process.stdout.cursorTo(0);
  process.stdout.write(`waiting ... ${p}%`);
};

console.log(`setting a ${waitTime / 1000} second delay`);

const timerFinished = () => {
  clearInterval(interval);
  process.stdout.clearLine();
  process.stdout.cursorTo(0);
  console.log("done");
};

const interval = setInterval(incTime, waitInterval);
setTimeout(timerFinished, waitTime);
```

# CORE MODULES

## path

```js
// path module is shipped with node.js but must be imported
const path = require("path");
console.log(`The file name is ${path.basename(__filename)}`);
// ...or with destructuring:
const { basename } = require("path");
console.log(`The file name is ${basename(__filename)}`);
```

## util

```js
// util modules is shipped with node.js but must be imported
const util = require("util");
util.log(path.basename(__filename)); // log with automatic timedate
// ...or with destructuring:
const { log } = require("util");
log(path.basename(__filename));
```

## v8

```js
// v8 modules is shipped with node.js but must be imported
const v8 = require("v8");
util.log(v8.getHeapStatistics());
// ...or with destructuring:
const { getHeapStatistics } = require("util");
getHeapStatistics();
```

## readline

* simplyfies promting the user and collecting answers (asynchronously)

```js
const readline = require("readline");

const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout
});

rl.question("How do you like node?", answer => {
    console.log(`Your answer: ${answer}`);
});
```

