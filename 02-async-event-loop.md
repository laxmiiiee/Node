# Phase 2 — Asynchronous Node.js & Event Loop

## 1. The core problem

Consider:

```js
const data = await database.query(...);
```

You do not want the entire server to stop executing JavaScript while the database is working.

Node's architecture allows the operation to be initiated and the JavaScript thread to continue handling other eligible work.

## 2. Synchronous vs asynchronous

Synchronous:

```js
const data = fs.readFileSync("large.txt");
console.log("after");
```

The JavaScript thread waits.

Asynchronous:

```js
fs.readFile("large.txt", (err, data) => {
    console.log(data);
});

console.log("after");
```

The callback runs later after the operation completes.

## 3. Event loop mental model

Simplified:

```text
                 ┌───────────────┐
                 │ Call Stack    │
                 └───────┬───────┘
                         │
                         ▼
                 Event Loop / runtime
                         │
          ┌──────────────┼──────────────┐
          ▼              ▼              ▼
       timers          I/O           callbacks
          │              │              │
          └──────────────┴──────────────┘
                         │
                         ▼
                  Call Stack again
```

The exact Node event-loop phases are more nuanced.

## 4. Microtasks

Promise callbacks and `queueMicrotask()` use the microtask queue.

```js
Promise.resolve().then(() => {
    console.log("promise");
});
```

Node also has `process.nextTick()`, which has special scheduling behavior and should be used carefully.

Important interview point:

> Microtasks can run before the event loop proceeds to later work, and excessive microtask scheduling can delay other work.

## 5. `process.nextTick()`

```js
process.nextTick(() => {
    console.log("next tick");
});
```

It is not simply another timer.

`process.nextTick()` callbacks are processed very early after the current operation, before the event loop continues normally.

Abusing it can starve the event loop.

## 6. Promise flow

```js
async function getUser() {
    const user = await db.findUser();
    return user;
}
```

`await` does not mean "block the Node process."

It pauses the current async function until the promise settles, allowing the runtime to continue handling other work.

## 7. Promise states

A Promise can be:
- pending
- fulfilled
- rejected

```text
pending
  ├──→ fulfilled
  └──→ rejected
```

It cannot move back to pending or change after settlement.

## 8. `async` functions

An `async` function always returns a Promise.

```js
async function f() {
    return 42;
}
```

Conceptually:

```js
Promise.resolve(42)
```

## 9. Errors

Synchronous throw inside an async function becomes a rejected Promise:

```js
async function f() {
    throw new Error("bad");
}
```

Handle with:

```js
try {
    await f();
} catch (err) {
    // handle
}
```

## 10. Promise concurrency

Sequential:

```js
const a = await getA();
const b = await getB();
```

If independent, this may be slower than necessary.

Concurrent:

```js
const [a, b] = await Promise.all([
    getA(),
    getB()
]);
```

Use concurrency when operations are independent and the system can safely handle the load.

## 11. `Promise.all`

`Promise.all()`:
- starts/observes all provided promises
- fulfills when all fulfill
- rejects when one rejects

It does not magically cancel already-running underlying operations when one promise rejects.

## 12. CPU-bound work

If you have:

```js
for (let i = 0; i < 10_000_000_000; i++) {
    // expensive work
}
```

that JavaScript blocks the event loop.

Even though the code is "inside Node," it is still synchronous CPU work.

## 13. Interview scenario

Question:

> CPU usage is 95%, database metrics are normal, and API latency is increasing. What do you investigate?

Strong answer:

1. Check event-loop lag.
2. Profile CPU.
3. Look for synchronous CPU-heavy code.
4. Check JSON serialization/parsing or expensive transformations.
5. Check regex/pathological algorithms.
6. Move CPU-heavy work to worker threads/background jobs if appropriate.
7. Scale horizontally only after understanding the bottleneck.

## 14. Event-loop blocking

Bad examples:
- huge synchronous loops
- `JSON.parse()` of extremely large input
- `JSON.stringify()` of extremely large objects
- synchronous filesystem operations in request paths
- catastrophic regular expressions
- expensive crypto/computation on the main thread

## 15. Mental model to remember

```text
I/O-bound
→ Node is strong

CPU-bound on main JS thread
→ can block everyone sharing that event loop
```
