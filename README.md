# Worker Threads

Node.js is inherently single-threaded. It uses the event loop and non-blocking I/O operations to handle concurrency, making it efficient for I/O-bound tasks. However, for CPU-bound tasks (like heavy computations), Node.js's single-threaded nature becomes a bottleneck. The event loop can get blocked, leading to performance issues and a poor user experience.

Worker threads allow you to perform CPU-intensive tasks without blocking the main event loop by offloading these tasks to separate threads.

Imports:

```js
const {
  Worker,
  isMainThread,
  parentPort,
  workerData,
} = require("worker_threads");
```

```js
// main.js
const { Worker } = require("worker_threads");

function runService(workerData) {
  return new Promise((resolve, reject) => {
    const worker = new Worker("./service.js", { workerData });
    worker.on("message", resolve);
    worker.on("error", reject);
    worker.on("exit", (code) => {
      if (code !== 0)
        reject(new Error(`Worker stopped with exit code ${code}`));
    });
  });
}

async function main() {
  const result = await runService("hello");
  console.log(result); // Prints 'hello worker'
}

main().catch((err) => console.error(err));
```

```js
// service.js
const { parentPort, workerData } = require("worker_threads");

parentPort.postMessage(`${workerData} worker`);
```

Only use Worker Threads for CPU-intensive tasks, not for I/O-bound tasks.

# Cluster Module

Node.js runs in a single thread by default, and even though it's highly efficient in handling I/O operations, it doesn't fully utilize the multi-core capabilities of modern processors. This means that, by default, a Node.js application runs on a single CPU core, which can limit its performance and scalability, especially under high traffic.

## Cluster Module to the Rescue

The `cluster` module in Node.js allows you to create multiple child processes (workers), which all share server ports. It enables you to scale your application across multiple cores of your CPU, enhancing performance and the ability to handle more concurrent client requests.

## Key Concepts

- **Master Process**: The initial process that starts when you run your Node.js application. It can be used to fork multiple worker processes.
- **Worker Processes**: These are child processes forked by the master process. They are separate instances of your Node.js application and handle actual client requests.

## Setting Up the Cluster Module

1. **Installation**: No additional installation is required as the `cluster` module is a core Node.js module.

2. **Importing the Module**:
   ```javascript
   const cluster = require("cluster");
   const http = require("http");
   const numCPUs = require("os").cpus().length;
   ```

## Basic Example

### Server Code (`server.js`)

```javascript
const cluster = require("cluster");
const http = require("http");
const numCPUs = require("os").cpus().length;

if (cluster.isMaster) {
  console.log(`Master ${process.pid} is running`);

  // Fork workers.
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on("exit", (worker, code, signal) => {
    console.log(`worker ${worker.process.pid} died`);
  });
} else {
  // Workers can share any TCP connection
  // In this case, it is an HTTP server
  http
    .createServer((req, res) => {
      res.writeHead(200);
      res.end("Hello World\n");
    })
    .listen(8000);

  console.log(`Worker ${process.pid} started`);
}
```

In this example, the master process forks a number of workers equal to the number of CPU cores. Each worker runs an HTTP server listening on the same port (8000).

# Mastering memory management in Node.js

Node.js, being built on the V8 JavaScript engine, inherits its memory management capabilities. While V8 is efficient, it's designed with web browsers in mind, hence it has certain limitations when it comes to server-side applications like those built on Node.js.

The main challenges in memory management are:

1. **Heap Memory Limit**: V8 has default memory limits (about 1.5 GB on 64-bit machines), which might be insufficient for heavy applications.
2. **Memory Leaks**: Poor coding practices can lead to memory leaks, where memory is not released even when it's no longer needed.
3. **Garbage Collection**: JavaScript automatically cleans up unused memory (garbage collection), but this can impact performance if not managed properly.

## Strategies for Memory Management

1. **Monitoring and Profiling**: Regularly monitor memory usage and profile your application to understand how memory is being used.

2. **Efficient Data Structures**: Use memory-efficient data structures and algorithms. For example, Buffer objects for binary data are more efficient than strings.

3. **Memory Leak Detection**: Use tools like `memwatch-next` or `node-memwatch` to detect memory leaks.

4. **Avoid Global Variables**: Limit the use of global variables as they stay in memory for the lifetime of the application.

5. **Manage Scope and Closures**: Be mindful of closures and scope; unintentional references can prevent memory from being freed.

6. **Stream Large Data**: When handling large files or data sets, use streams to process data in chunks rather than loading everything into memory.

## Code Example: Efficient Data Handling

Here’s a basic example of using streams to handle large data efficiently:

```javascript
const fs = require("fs");

const readStream = fs.createReadStream("largefile.txt");
const writeStream = fs.createWriteStream("output.txt");

readStream.on("data", (chunk) => {
  writeStream.write(chunk);
});

readStream.on("end", () => {
  writeStream.end();
});
```

In this example, a large file is read and written in chunks, which prevents the entire file from being loaded into memory.

# Duplex Streams

Duplex streams in Node.js are a type of stream that can be both read from and written to. This is different from readable and writable streams, which only allow either reading or writing, but not both simultaneously. Duplex streams are useful when you need a continuous communication channel, like in the case of a TCP socket where data can be sent and received at the same time.

## Types of Duplex Streams

1. **Duplex**: Can read and write data, but the read and write operations are independent of each other.
2. **Transform**: A special type of duplex stream where the output is computed from the input. They are often used to modify data as it is written and read.

## Transform Streams

Transform streams are a common use case of duplex streams. They allow you to create a stream that transforms data as it is read or written. For example, you could create a transform stream to compress, encrypt, or modify data.

## Code Example: Creating a Transform Stream

Let's create a simple transform stream that converts input text to uppercase.

```javascript
const { Transform } = require("stream");

const uppercaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    // Convert the chunk of data to uppercase
    this.push(chunk.toString().toUpperCase());
    callback();
  },
});

// Using the Transform Stream
process.stdin.pipe(uppercaseTransform).pipe(process.stdout);
```

In this example, `uppercaseTransform` is a transform stream that takes a chunk of data (from `process.stdin`), converts it to uppercase, and then pushes it to the next destination (in this case, `process.stdout`).

## Using Pipe

The `pipe()` method is used to take a readable stream and connect it to a writable stream. In the above example, `process.stdin` is piped to `uppercaseTransform`, and then `uppercaseTransform` is piped to `process.stdout`. This creates a pipeline where data automatically flows from stdin, through the transform, to stdout.

## Real-World Use Case

A common use case for duplex streams, especially transform streams, is in handling network communication or file manipulation. For example, you could use a transform stream to encrypt data before it's written to a file or sent over a network, and then decrypt it when read back.
