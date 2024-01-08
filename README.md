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
