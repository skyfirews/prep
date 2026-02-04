# Node.js (Core) – Interview Notes

> **Target**: Senior Engineers, Backend (JavaScript/Node) | **Level**: Amazon, Flipkart, Razorpay

---

## 1. Overview

**What it is**: Node.js is a JavaScript runtime built on Chrome's V8 engine. It runs JavaScript outside the browser, with non-blocking, event-driven I/O and a single-threaded event loop.

**Why companies use it**: One language (JS) for frontend and backend; huge ecosystem (npm); non-blocking I/O for many concurrent connections; fast for I/O-bound APIs and real-time apps.

**When NOT to use it**: CPU-bound heavy computation (GIL-like; use worker threads or separate service); when you need strong typing without extra tooling (consider TypeScript); when team or ecosystem is stronger in another stack.

---

## 2. Core Concepts (From PDF)

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Node.js | JavaScript runtime on V8 | Run JS on server; event-driven | Why Node for backend | Event loop |
| V8 | JavaScript engine | Compiles JS to machine code | Same as Chrome | |
| Event loop | Handles async operations | Single thread; queue callbacks | How async works | Microtask vs macrotask |
| Call stack | Executes synchronous code | LIFO; one at a time | Blocking | |
| Callback queue | Pending async callbacks | setTimeout, I/O callbacks | Order of execution | |
| Microtask queue | Promises, process.nextTick | Higher priority than callback queue | Promise vs setTimeout | |
| Non-blocking I/O | Don't wait for I/O on main thread | Callback or promise when done | Why Node is fast for I/O | |
| Callbacks | Function passed to async op | err-first convention | Callback hell | |
| Promises | Async value (pending/fulfilled/rejected) | .then/.catch or async/await | Promise chain | |
| async/await | Syntactic sugar for Promises | Pause until promise resolves | Error handling | |
| Core modules | fs, http, path, os, url | Built-in; no install | When to use | |
| require (CommonJS) | Sync module load | require("module") | CommonJS vs ES modules | |
| import/export (ESM) | ES modules | import x from "x" | When to use ESM | |
| Streams | Readable, Writable, Duplex | Chunk-by-chunk data | Backpressure | |
| Buffer | Binary data in Node | Fixed-size raw memory | When to use | |
| process | Current Node process | process.env, process.exit() | Env vars | |
| Cluster | Multi-core | One process per CPU | When to use | |
| Worker Threads | CPU-intensive in separate thread | Bypass event loop for CPU | When to use | |

---

## 3. How It Works (Internals)

**Event loop**  
1. Execute sync code (call stack).  
2. When stack is empty, run microtasks (Promises, process.nextTick).  
3. Run one macrotask (setTimeout, I/O callback) from callback queue.  
4. Repeat 2–3.  
So: sync → microtasks → one macrotask → microtasks → …

**Non-blocking I/O**  
Read file / HTTP request: Node starts I/O and registers callback. Event loop continues. When I/O completes, callback is queued and runs when stack is empty. One thread serves many connections.

**Single-threaded**  
One call stack. Blocking (long sync computation) blocks the whole process. Use worker threads for CPU-heavy work.

**Word diagram**  
```
[Call Stack] ← sync code
      ↓
[Microtask Queue] (Promises, nextTick) – drain fully
      ↓
[Macrotask Queue] (setTimeout, I/O) – one per loop tick
      ↓
repeat
```

---

## 4. Real-World Use Case

**REST API and real-time service**

- **Where Node fits**: API server (Express/Fastify); WebSocket server (real-time); I/O-bound (DB, HTTP, file). Single process handles many connections via event loop.
- **Why this approach**: One language (JS/TS); non-blocking I/O for high concurrency; npm ecosystem; real-time with same stack.
- **Trade-offs**: CPU-bound work blocks event loop (use worker threads or separate service); need discipline for async error handling.

---

## 5. Code Example (Minimal but Powerful)

```javascript
// Node.js – event loop order, async/await, streams
const fs = require("fs");
const http = require("http");

// Microtask runs before macrotask
setTimeout(() => console.log("timeout"), 0);
Promise.resolve().then(() => console.log("promise"));
console.log("sync");
// Output: sync, promise, timeout

// Async/await – always catch
async function fetchUser(id) {
  const res = await fetch(`https://api.example.com/users/${id}`);
  if (!res.ok) throw new Error(res.statusText);
  return res.json();
}
// Wrong: no try/catch → unhandled rejection

// Stream – don't load whole file in memory
const readStream = fs.createReadStream("large.txt");
readStream.on("data", (chunk) => process.stdout.write(chunk));
readStream.on("error", (err) => console.error(err));
// Wrong: fs.readFileSync("large.txt") → high memory for large files
```

---

## 6. Best Practices (Interview-Grade)

| Do | Don't |
|----|--------|
| Use async/await with try/catch | Don't leave promise rejections unhandled |
| Use streams for large data | Don't load huge files into memory |
| Use env vars for config | Don't hardcode secrets or ports |
| Avoid blocking the event loop | Don't do heavy sync computation in main thread |
| Use Worker Threads for CPU-bound | Don't assume threads for I/O (event loop is enough) |
| Use Cluster for multi-core | Don't run single process when you have many cores |

**Security**: Validate input; don't eval user input; use parameterized queries; keep dependencies updated.  
**Performance**: Streams, connection pooling, caching; profile before optimizing.

---

## 7. Common Mistakes (Interview Red Flags)

| Mistake | Why wrong | Correct mental model |
|---------|-----------|------------------------|
| "Node is multi-threaded" | One thread for JS; I/O is async (libuv) | Single-threaded event loop; use workers for CPU |
| "setTimeout(fn, 0) runs immediately" | It runs after current sync code and microtasks | It queues macrotask; microtasks run first |
| "Promises and setTimeout same order" | Microtasks (Promise) before macrotasks (setTimeout) | Know event loop order |
| "We don't need to catch in async" | Unhandled rejection can crash or hang | Always try/catch or .catch in async |
| "Sync is fine for small files" | For large files, memory and blocking | Use streams for large I/O |

---

## 8. Interview Questions & Answers

### A. Basic (Warm-up)

**Q: What is the event loop?**  
**Short answer**: Mechanism that runs JavaScript: executes sync code, then runs microtasks (Promises), then one macrotask (setTimeout, I/O), repeat. Single thread.  
**Follow-up**: "Why single-threaded?"  
**Ideal answer**: Simplicity and no locks for JS code. I/O is delegated to libuv (thread pool or OS); when I/O completes, callback is queued. One thread can serve many I/O-bound connections.

**Q: What is the difference between require and import?**  
**Short answer**: require is CommonJS (sync, default in Node). import is ES modules (async load, static). Use "type": "module" in package.json for ESM.  
**Follow-up**: "When to use which?"  
**Ideal answer**: New projects often use ESM (import). Legacy or many npm packages still use CommonJS. Mix with care (interop rules).

### B. Intermediate (Most common)

**Q: What is the order of execution: setTimeout(fn, 0) vs Promise.then?**  
**Short answer**: Sync code first. Then all microtasks (Promise.then, process.nextTick). Then one macrotask (setTimeout). So Promise runs before setTimeout even with 0 delay.  
**Follow-up**: "What is process.nextTick vs Promise?"  
**Ideal answer**: Both are microtasks. nextTick runs before Promise callbacks in the same phase. Use nextTick for "run before next event loop tick" (e.g. defer to end of current sync).

**Q: Why is Node.js good for I/O but not for CPU-bound?**  
**Short answer**: I/O is non-blocking; while waiting for DB/network, event loop serves other requests. CPU-bound (e.g. heavy computation) blocks the single thread; no other request is handled.  
**Follow-up**: "How do you handle CPU-bound in Node?"  
**Ideal answer**: Worker Threads (separate JS thread), or offload to another service (e.g. queue + worker in another language), or use native addon.

### C. Advanced / Tricky (Senior-level)

**Q: Explain streams and backpressure.**  
**Ideal answer**: Streams process data in chunks (Readable, Writable, Duplex). Backpressure: when consumer is slow, producer should slow down. Writable has .write() returning false when buffer is full; wait for "drain" before writing more. Pipe handles this for you.

**Q: How would you scale a Node.js API?**  
**Ideal answer**: Horizontal: multiple processes (Cluster or PM2) behind load balancer. Each process uses event loop for I/O. Vertical: more CPU only helps if you use Worker Threads for CPU-bound. Scale DB, cache, and use connection pooling. Stateless app for easy scaling.

---

## 9. Quick Revision

- **Event loop**: sync → microtasks (Promise, nextTick) → one macrotask (setTimeout, I/O).
- **Single-threaded** for JS; **non-blocking I/O** via callbacks/promises.
- **async/await** = Promises; always **try/catch** or .catch.
- **Streams** for large data; **Buffer** for binary.
- **Cluster** = multi-core; **Worker Threads** = CPU-bound.
- Don't **block** the event loop.

---

## 10. References

- [Node.js Documentation](https://nodejs.org/docs/)
- [Event Loop Explained (Node.js)](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)
