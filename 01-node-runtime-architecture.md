# Phase 1 — Node.js Runtime & Architecture

## 1. What Node.js is

Node.js is a JavaScript runtime built around Google's V8 JavaScript engine, with additional runtime APIs and an event-driven architecture.

Important distinction:

- JavaScript = language.
- V8 = JavaScript engine.
- Node.js = runtime that embeds V8 and provides server/runtime capabilities.
- Express = web framework running on Node.

Mental model:

```text
JavaScript
   ↓
V8
   ↓
Node.js runtime
   ├── event loop
   ├── Node APIs
   ├── libuv
   ├── networking
   ├── filesystem
   └── process/runtime facilities
```

## 2. Why Node is useful for backend systems

Node is particularly strong for workloads with lots of I/O:
- HTTP requests
- database calls
- filesystem operations
- network calls
- queues and messaging

The key idea is that Node can avoid blocking the JavaScript thread while I/O is in progress.

Node is **not** automatically ideal for CPU-heavy work. Long-running CPU work on the main JavaScript thread can block other requests.

## 3. Single-threaded JavaScript execution

The JavaScript execution environment has one main thread for executing JavaScript at a time.

This does NOT mean Node can never use other threads.

Node/libuv and the OS can use other mechanisms for I/O and certain operations, and Node provides worker threads for CPU-heavy JavaScript work.

Interview-safe wording:

> Node executes JavaScript on a main thread, while asynchronous I/O can be handled through the runtime/OS and libuv mechanisms. CPU-heavy JavaScript on the main thread can still block the event loop.

## 4. V8

V8:
- parses/compiles JavaScript
- executes JavaScript
- manages JavaScript memory
- performs garbage collection

Node adds runtime capabilities around V8.

## 5. libuv

libuv is a major part of Node's asynchronous infrastructure.

It provides the event loop and abstractions around asynchronous operations.

A simplified mental model:

```text
Your JavaScript
      ↓
Node APIs
      ↓
libuv / OS facilities
      ↓
I/O completes
      ↓
callback becomes eligible
      ↓
event loop
      ↓
JavaScript runs
```

Do not oversimplify by saying "libuv does everything." Some operations are handled directly by the OS/kernel, while some use libuv's thread pool.

## 6. Node process

A Node application runs as a process.

Useful concepts:
- `process.pid`
- `process.env`
- `process.argv`
- `process.cwd()`
- signals
- exit codes

Environment variables are commonly used for configuration:

```js
const port = process.env.PORT || 3000;
```

Secrets should not be hard-coded in source control.

## 7. CommonJS vs ESM

CommonJS:

```js
const fs = require("fs");
module.exports = something;
```

ES modules:

```js
import fs from "node:fs";
export default something;
```

Modern Node supports both, depending on project configuration.

## 8. Important interview distinction

Do not say:

> Node is single-threaded, so it cannot handle many requests.

Better:

> JavaScript execution occurs on the main thread, but Node uses asynchronous I/O and runtime facilities so a single process can efficiently handle many concurrent I/O-bound operations. CPU-heavy JavaScript can still block that main thread.

## 9. When Node struggles

Example:

```js
app.post("/calculate", (req, res) => {
    veryExpensiveCalculation();
    res.json({ ok: true });
});
```

If `veryExpensiveCalculation()` takes several seconds synchronously, other JavaScript callbacks cannot run during that period.

Symptoms:
- latency spikes
- requests queue up
- event loop lag increases
- throughput falls

Solutions depend on workload:
- optimize the algorithm
- move CPU-heavy work to worker threads
- use separate processes/services
- queue background work
- scale horizontally

## 10. Worker threads vs child processes

Worker threads:
- run JavaScript in separate threads
- useful for CPU-intensive JavaScript
- can communicate with the parent
- can share some memory through SharedArrayBuffer where deliberately designed

Child processes:
- separate OS processes
- stronger isolation
- separate memory space
- useful for running separate programs or process-level isolation

Rule of thumb:

```text
CPU-heavy JavaScript → worker threads
Separate process/program/isolation → child process
```

## Interview traps

### "Node is single-threaded."
Incomplete.

### "Node is non-blocking."
Only when you avoid blocking operations on the main JavaScript thread. Synchronous CPU work still blocks.

### "Async means another JavaScript thread."
No. Async programming is about scheduling/completion; JavaScript execution is still performed on the main thread unless you explicitly use workers/processes.
