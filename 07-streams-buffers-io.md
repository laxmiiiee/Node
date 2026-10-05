# Phase 7 — Streams, Buffers & I/O

## 1. Why streams matter

Node is designed around asynchronous I/O.

Streams let you process data incrementally rather than requiring the entire payload in memory.

Useful for:
- files
- HTTP bodies
- uploads/downloads
- compression
- network data

## 2. Buffer

A Buffer represents raw binary data.

```js
const buffer = Buffer.from("hello");
```

Buffers are useful when working with:
- files
- network packets
- binary protocols
- encoded data

## 3. Readable stream

A readable stream produces data.

Examples:
- file read stream
- HTTP request body
- process stdin

## 4. Writable stream

A writable stream consumes data.

Examples:
- file write stream
- HTTP response
- process stdout

## 5. Duplex stream

Both readable and writable.

Example:
- network socket

## 6. Transform stream

A stream that transforms data while it passes through.

Examples:
- gzip compression
- parsing/transformation

## 7. Why not read huge files at once?

Bad:

```js
const data = fs.readFileSync("huge-file.csv");
```

Potential problem:
- high memory usage
- event-loop blocking if synchronous
- poor scalability

Better:

```js
const stream = fs.createReadStream("huge-file.csv");
```

Then process chunks.

## 8. Backpressure

Backpressure happens when the producer generates data faster than the consumer can handle it.

Without backpressure:

```text
Producer >>>>>>>>>>> Consumer
       memory grows
```

With backpressure:

```text
Producer → controlled flow → Consumer
```

Node streams provide mechanisms to handle this.

## 9. `pipe`

Conceptually:

```js
readable.pipe(writable);
```

This connects the streams and manages flow.

Example:

```js
fs.createReadStream("input.txt")
  .pipe(fs.createWriteStream("output.txt"));
```

## 10. HTTP and streams

Request bodies and responses are stream-based.

This is why understanding streams helps explain:
- uploads
- downloads
- large payloads
- streaming responses

## 11. Buffer vs stream

Buffer:
> data already held in memory.

Stream:
> a mechanism for processing data incrementally over time.

## 12. Interview answer

Q: Why use streams?

> Streams let Node process data incrementally, which can reduce memory usage and allow efficient handling of large files or network data. They also provide backpressure mechanisms so fast producers don't overwhelm slower consumers.
