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
