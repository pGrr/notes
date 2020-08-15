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

* everything on the `global` object is available to us globally, i.e. the global object contains all of the objects, values, and methods that we can use in a Node.js file without having to import anything, e.g. `__dirname`, `__filename`, `process`, `console`, `module.exports`, `require()`, etc
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

## COMMONJS MODULE PATTERN - require(), module.exports

### require()

* You can import
    * core modules, which are shipped with node.js but not included in the global object so they must be imported
    * node npm packages
    * json files (which will be automatically parsed into a js object)
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

### module.exports

* What is assigned to `module.exports` (which is part of the global object) will be returned by `require("myModule")`
* Everything else will be scoped to the js file (i.e. the module) and unaccessible from the outside: this allows you to have both "private" and "public" fields or methods (e.g. alike to a Java class)

```js
// counter.js
let count = 0;

const inc = () => ++count;
const dec = () => --count;
const getCount = () => count;

module.exports = {
  inc,
  dec,
  getCount
};
```

```js
// usecounter.js
const counter = require("counter");
counter.inc();
counter.dec();
conter.getCount();

// ...or using destructuring
{ inc, getCount } = require("counter");
inc();
getCount();
```

## PROCESSES - process

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
    // ...and do something to stop processing input 
    // when a condition is met, e.g. write to stdout or process.exit()
    // (readline might be a better choice to get user input)
});
```

#### Question/answer example with stdin

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

## TIMING - setTimeout(), clearTimeout(), setInterval(), clearInterval()

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

* Core modules are modules included with node.js but which are not part of the global object (so they must be imported), which provide a lot of useful functionalities.

## USER PROMPTING - readline

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

### Question/answers example with readline

```js
const readline = require("readline");

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

const questions = [
  "What is your name? ",
  "Where do you live? ",
  "What are you going to do with node js? "
];

const collectAnswers = (questions, done) => {
  const answers = [];
  const [firstQuestion] = questions;

  const questionAnswered = answer => {
    answers.push(answer);
    if (answers.length < questions.length) {
      rl.question(questions[answers.length], questionAnswered);
    } else {
      done(answers);
    }
  };

  rl.question(firstQuestion, questionAnswered);
};

collectAnswers(questions, answers => {
  console.log("Thank you for your answers. ");
  console.log(answers);
  process.exit();
});
```

## PATH UTILITIES - path

```js
// path module is shipped with node.js but must be imported
const path = require("path");
console.log(`The file name is ${path.basename(__filename)}`);
// ...or with destructuring:
const { basename } = require("path");
console.log(`The file name is ${basename(__filename)}`);
```

## UTILITIES AND LOGS - util

```js
// util modules is shipped with node.js but must be imported
const util = require("util");
util.log(path.basename(__filename)); // log with automatic timedate
// ...or with destructuring:
const { log } = require("util");
log(path.basename(__filename));
```

## V8 UTILITIES - v8

```js
// v8 modules is shipped with node.js but must be imported
const v8 = require("v8");
util.log(v8.getHeapStatistics());
// ...or with destructuring:
const { getHeapStatistics } = require("util");
getHeapStatistics();
```

## EVENTS - events

* __The event emitter is Node.js implementation of the publish-subscribe pattern__: `emitter = events.EventEmitter()` gives us a mechanism for emitting custom events (with a name and associated data), and wiring up listeners and handlers for those events using `emitter.on(name, callback)`, where the callback is passed the associated data.

```js
const events = require("events");

const emitter = new events.EventEmitter();

emitter.on("customEvent", (message, user) => {
  console.log(`${user}: ${message}`);
});

process.stdin.on("data", data => {
  const input = data.toString().trim();
  if (input === "exit") {
    emitter.emit("customEvent", "Goodbye!", "process");
    process.exit();
  }
  emitter.emit("customEvent", input, "terminal");
});
```

## FILESYSTEM - fs

* The `fs` module can be used to list files in directories, create new files in directories, stream files, watch files, modify file permissions, just about anything you would need to work with files and directories.
  * most of the `fs.*` functions are available both asynchronous (default) or synchronous (usually with a `Sync` appended to the function name)

### List directory files - fs.readdir

```js
const fs = require("fs");

fs.readdir("./assets", (err, files) => {
  if (err) {
    throw err;
  }
  console.log("complete");
  console.log(files);
});

console.log("started reading files");
```

### Read files - fs.readFile

```js
const fs = require("fs");

// read text files
fs.readFile("./Readme.md", "UTF-8", (err, text) => {
  console.log(text);
});

// read binary files
fs.readFile("./assets/alex.jpg", (err, img) => {
  if (err) {
    console.log(`An error has occured: ${err.message}`);
    process.exit();
  }
  console.log("file contents read");
  console.log(img);
});
```

### Write and append files - fs.writeFile

* You can use `fs.writeFile(file, data[, options], callback)` to write to a file, where in the options you can specify the writing flag (e.g. `w`, `a`, etc - [more](https://nodejs.org/api/fs.html#fs_file_system_flags)) - [more](https://nodejs.org/api/fs.html#fs_fs_writefile_file_data_options_callback)
  * `fs.appendFile` is the same, just with the `a` flag as default
* Only use this for occasional appends. Never use it to append frequently (e.g. for a log), as it opens a file handle for each piece of data you add to your file, after a while you get a `EMFILE` error. In these cases, it's much better to [reuse the file handler](https://stackoverflow.com/questions/3459476/how-to-append-to-a-file-in-node/43370201#43370201)

```js
// fs.writeFile
const fs = require("fs");

const md = `
# This is a new file

We can write text to a file with fs.writeFile
`;

fs.writeFile("./assets/notes.md", 
  {
    encoding: "utf-8",
    flag: "w",
  }, 
  md.trim(), 
  err => {
    if (err) {
      throw err;
    }
    console.log("file saved");
  }
);
```

```js
// fs.appendFile
const fs = require("fs");
const colorData = require("./assets/colors.json");

colorData.colorList.forEach(c => {
  fs.appendFile("./storage-files/colors.md", `${c.color}: ${c.hex} \n`, err => {
    if (err) {
      throw err;
    }
  });
});
```

### Create directory - fs.mkdir

```js
const fs = require("fs");

if (fs.existsSync("storage-files")) {
  console.log("Already there");
} else {
  fs.mkdir("storage-files", err => {
    if (err) {
      throw err;
    }
    console.log("directory created");
  });
}
```

### Rename and Remove files - fs.rename, fs.unlink

```js
const fs = require("fs");

fs.renameSync("./assets/colors.json", "./assets/colorData.json");

fs.rename("./assets/notes.md", "./storage-files/notes.md", err => {
  if (err) {
    throw err;
  }
});

setTimeout(() => {
  fs.unlinkSync("./assets/alex.jpg");
}, 4000);
```

### Rename and Remove directories - fs.rename , fs.rmdir

```js
const fs = require("fs");

// rename directory
fs.rename("./old-storage", "./storage", err => {
  if (err) {
    throw err;
  }
});

// rmdir will throw an error if the directory is not empty
fs.readdirSync("./storage").forEach(fileName => {
  fs.unlinkSync(`./storage/${fileName}`);
});

// remove directory
fs.rmdir("./storage", err => {
  if (err) {
    throw err;
  }
  console.log("./storage directory removed");
});
```

## CHILD PROCESSES - child_process

* Node.js comes with a child process module, which allows you to execute external processes in your environment. In other words, you node JS app can run and communicate with other applications within the environment that is hosted.
* The `child_process` module provides a lot of functionalities to create and manage child processes: `exec`, `fork`, `
* The most common methods to create a child process are `exec` and `spawn`
  * `exec` 
    * returns the whole buffer output from the child process.
    * spawns a shell and runs a command within that shell, passing the stdout and stderr to a callback function when complete.
    * By default the buffer size is set at 200k. If the child process returns anything more than that, you program will crash with the error message `Error: maxBuffer exceeded`. You can fix that problem by setting a bigger buffer size in the exec options. But you should not do it because exec is not meant for processes that return HUGE buffers to Node.
    * is "synchronously asynchronous", meaning although the exec is asynchronous, it waits for the child process to end and tries to return all the buffered data at once. If the buffer size of exec is not set big enough, it fails with a "maxBuffer exceeded" error.
    * is best used to run processes (do something, spit the result) that are synchronous and return result statuses or small amounts of data
  * `spawn` is for asynchronous processes
    * returns an object with stdout and stderr streams. You can tap on the stdout stream to read data that the child process sends back to Node
    * is "asynchronously asynchronous", meaning it starts sending back data from the child process in a stream as soon as the child process starts executing.
    * is best used for asynchronous processes, long running processes, processes that remain waiting for input, anything that remains open, or whenever you expect the child process to return a large amount of data, e.g. image processing, reading binary data etc.
* [more](https://nodejs.org/api/child_process.html)

```js
// exec
const cp = require("child_process");

cp.exec("ls", (err, data, stderr) => {
    if (err) {
      throw err;
      // or console.log(stderr)
    }
    console.log(data);
});
```

```js
// spawn
const cp = require("child_process");

const questionApp = cp.spawn("node", ["questions.js"]);

questionApp.stdin.write("Alex \n");
questionApp.stdin.write("Tahoe \n");
questionApp.stdin.write("Change the world \n");

questionApp.stdout.on("data", data => {
  console.log(`from the question app: ${data}`);
});

questionApp.on("close", () => {
  console.log(`questionApp process exited`);
});
```

# STREAMS

* The stream API interface allow to read and write data
  * to files (included stdin/stdout/stderr)
  * to communicate with the internet
  * to communicate with other processes
* The data can be both text or binary data
* Data is read/written bit-by-bit/chunk-by-chunk (event-driven), thus streams are useful for big files, network communications, etc
* There are four fundamental stream types within Node.js:
  * __Readable__: streams from which data can be read (for example, fs.createReadStream()).
  * __Writable__: streams to which data can be written (for example, fs.createWriteStream()).
  * __Duplex__: streams that are both Readable and Writable (for example, net.Socket).
  * __Transform__: Duplex streams that can modify or transform the data as it is written and read (for example, zlib.createDeflate())
* Readable/Writable streams can be piped together with `readableSrc.pipe(writableDest)`
* [Streams 101](https://www.freecodecamp.org/news/node-js-streams-everything-you-need-to-know-c9141306be93/#:~:text=There%20are%20four%20fundamental%20stream,of%20that%20is%20the%20fs.)
* [more](https://nodejs.org/api/stream.html)

## Readable streams

* Readable streams read data chunk by chunk (by listening to events, e.g. `data`, `end`) instead of reading everything all at once and loading it into a buffer, thus will use less memory and is perfect for big files

```js
// process.stdin is a Readable stream
// (we read the data by listening to "data" events)
console.log("type something...");
process.stdin.on("data", data => {
  console.log(`I read ${data.length - 1} characters of text`);
});
```

```js
// fs.createReadStream() creates a readable stream
// (we read the data by listening to "data" events)
const fs = require("fs");

const readStream = fs.createReadStream("./assets/lorum-ipsum.md", "UTF-8");

let fileTxt = "";

console.log("type something...");
readStream.on("data", data => {
  console.log(`I read ${data.length - 1} characters of text`);
  fileTxt += data;
});

readStream.once("data", data => {
  console.log(data);
});

readStream.on("end", () => {
  console.log("finished reading file");
  console.log(`In total I read ${fileTxt.length - 1} characters of text`);
});
```

## Writable streams

```js
const fs = require("fs");

const writeStream = fs.createWriteStream("./assets/myFile.txt", "UTF-8");

writeStream.write("hello");
writeStream.write(" world \n");
```

```js
// fs.createWriteStream creates a Writable stream to a file
const fs = require("fs");

const writeStream = fs.createWriteStream("./assets/myFile.txt", "UTF-8");

writeStream.write("hello");
writeStream.write(" world \n");
```

## Piping streams - stream.pipe

* streams can be piped together with `readableSrc.pipe(writableDest)`

```js
// fs.createWriteStream creates a Writable stream to a file
const fs = require("fs");

const writeStream = fs.createWriteStream("./assets/myFile.txt", "UTF-8");
const readStream = fs.createReadStream("./assets/lorum-ipsum.md", "UTF-8");

readStream.pipe(writeStream);
```



