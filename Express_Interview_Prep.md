# Express.js – Interview Notes

> **Target**: Senior Engineers, Backend (Node.js) | **Level**: Amazon, Flipkart, Razorpay

---

## 1. Overview

**What it is**: Express.js is a minimal, unopinionated web framework for Node.js. It provides routing, middleware, and request–response handling for building REST APIs and web applications.

**Why companies use it**: Fast to build APIs; huge ecosystem (middleware, ORMs); runs on Node.js (single language for frontend and backend); non-blocking I/O for many concurrent connections.

**When NOT to use it**: When you need strong opinions (consider Nest.js, Fastify); when you need built-in GraphQL/WS (consider Apollo, Socket.io on top); when you prefer Python/Go for backend (use FastAPI, Go frameworks).

---

## 2. Core Concepts (From PDF)

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Express app | express() instance | Entry point; attach routes and middleware | app.listen, app.use | |
| Routing | app.get, app.post, app.put, app.delete | Map HTTP method + path to handler | Route params vs query | |
| Route parameters | /users/:id | Dynamic segment in URL | req.params | |
| Query parameters | ?page=1 | Key-value in URL | req.query | |
| Request body | req.body | JSON or form data | express.json() | |
| Middleware | Functions before final handler | Execute in order; next() passes control | Order matters | Error middleware |
| App-level middleware | app.use(fn) | Runs for all routes (or under path) | When to use | |
| Router-level middleware | router.use(fn) | Runs for router only | Modular routes | |
| Error-handling middleware | (err, req, res, next) | Four args; handles errors | Must be last | |
| express.json() | Parse JSON body | req.body populated | Required for POST JSON | |
| express.Router() | Mini app for routes | Mount at path; modular structure | app.use("/api", router) | |
| Async in routes | async/await + try/catch | Or pass to next(err) | Don't forget catch | |
| CORS | Cross-Origin Resource Sharing | Allow frontend from other origin | cors middleware | |
| Static files | express.static("public") | Serve files from folder | When to use | |

---

## 3. How It Works (Internals)

**Request flow**  
Request → Middleware 1 → Middleware 2 → … → Route handler → Response. Each middleware calls next() to pass to the next, or sends response and stops. If any middleware calls next(err), Express skips to error-handling middleware (four-arg function).

**Middleware order**  
Order of app.use() and route definitions matters. First matching route wins. Error middleware should be last (after all routes).

**Async errors**  
Async route handlers that throw or reject don't automatically go to error middleware. Wrap in try/catch and call next(err), or use a wrapper that catches and forwards.

**Word diagram**  
```
Request → [json] → [cors] → [auth?] → [route handler] → Response
                ↓
         [error middleware] ← next(err)
```

---

## 4. Real-World Use Case

**REST API for mobile and web**

- **Where Express fits**: API layer (routes, validation, auth); middleware for logging, rate limit, CORS; integration with DB (Sequelize, Mongoose) and cache (Redis).
- **Why this approach**: One codebase for API; Node.js good for I/O-bound; Express flexible and well-known.
- **Trade-offs**: Unopinionated so structure is up to you; need to add validation, auth, error handling yourself (or via middleware).

---

## 5. Code Example (Minimal but Powerful)

```javascript
// Express – routing, params, body, error handling
const express = require("express");
const app = express();

app.use(express.json()); // Required for req.body

// Route params: /users/42 → req.params.id === "42"
app.get("/users/:id", async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) return res.status(404).json({ error: "Not found" });
    res.json(user);
  } catch (err) {
    next(err); // Pass to error middleware
  }
});

// Query: /users?page=1&limit=10
app.get("/users", (req, res) => {
  const { page = 1, limit = 10 } = req.query;
  // Use page, limit for pagination
  res.json({ data: [] });
});

// Body: POST /users
app.post("/users", (req, res, next) => {
  const { name, email } = req.body; // From express.json()
  // Validate, then save. Don't trust req.body without validation.
  res.status(201).json({ id: 1, name, email });
});

// Error middleware – must be last, 4 args
app.use((err, req, res, next) => {
  console.error(err);
  res.status(err.status || 500).json({ error: err.message || "Internal error" });
});

app.listen(3000);
// Wrong: async route without try/catch → unhandled rejection, no error response.
// Wrong: no express.json() → req.body undefined for JSON POST.
```

---

## 6. Best Practices (Interview-Grade)

| Do | Don't |
|----|--------|
| Use try/catch or wrapper for async routes | Don't leave async errors unhandled |
| Validate and sanitize req.body, req.query | Don't trust client input |
| Use Router for modular routes | Don't put all routes in one file |
| Set security headers (e.g. Helmet) | Don't skip CORS and rate limiting in prod |
| Use environment variables for config | Don't hardcode ports or secrets |
| Centralize error handling | Don't scatter res.status(500) in every route |

**Security**: Helmet, CORS, rate limit; validate input; parameterized queries for DB.  
**Performance**: Compression, caching for GET; avoid blocking event loop.

---

## 7. Common Mistakes (Interview Red Flags)

| Mistake | Why wrong | Correct mental model |
|---------|-----------|------------------------|
| "Async errors are caught by Express" | They are not; unhandled rejection crashes or hangs | Always try/catch and next(err) in async routes |
| "req.params and req.query are the same" | params = path segments (/users/:id); query = ?key=value | Use params for resource id, query for filters/pagination |
| "Error middleware can be anywhere" | Express only uses four-arg middleware as error handler; order matters | Put error middleware after all routes |
| "We don't need express.json() for POST" | Without it, req.body is undefined for JSON | Use express.json() for JSON APIs |
| "Middleware order doesn't matter" | First matching route/middleware runs; order defines behavior | Put auth before protected routes; error handler last |

---

## 8. Interview Questions & Answers

### A. Basic (Warm-up)

**Q: What is middleware in Express?**  
**Short answer**: A function that receives req, res, next. It runs before the route handler; it can modify req/res, end the response, or call next() to pass to the next middleware/handler.  
**Follow-up**: "What is the difference between app.use and app.get?"  
**Ideal answer**: app.use runs for all methods (or all under a path) unless you add a method check. app.get only runs for GET. Both accept middleware; app.get is for a specific method and path.

**Q: How do you get route parameters vs query parameters?**  
**Short answer**: Route params from URL path: /users/:id → req.params.id. Query params from ?key=value → req.query.  
**Follow-up**: "When would you use each?"  
**Ideal answer**: Params for resource identity (e.g. user id). Query for filters, pagination, optional params (e.g. ?page=1&sort=name).

### B. Intermediate (Most common)

**Q: How do you handle errors in async route handlers?**  
**Short answer**: Wrap handler in try/catch and call next(err) in catch. Or use a wrapper that catches promise rejection and calls next(err). Express does not catch async errors automatically.  
**Follow-up**: "What if we don't call next(err)?"  
**Ideal answer**: Unhandled rejection; no response sent; client may timeout. Process may crash depending on Node version. Always forward errors to error middleware.

**Q: Explain the middleware flow.**  
**Short answer**: Request hits middleware in order. Each calls next() to continue or sends response to end. If next(err) is called, Express skips to the first four-argument error middleware.  
**Follow-up**: "Can error middleware be before routes?"  
**Ideal answer**: It can be defined before routes, but it only runs when next(err) is called. Defining it after routes ensures all route errors are passed to it. Convention is to put it last.

### C. Advanced / Tricky (Senior-level)

**Q: How would you structure a large Express API?**  
**Ideal answer**: By feature or domain: routes in routers (e.g. userRouter, orderRouter); mount at /api/users, /api/orders. Middleware in separate files (auth, validation, error). Config via env. Use a wrapper for async route handlers so every route forwards errors to central error middleware.

**Q: What happens if you block the event loop in an Express handler?**  
**Ideal answer**: Node.js is single-threaded. Blocking (e.g. long sync computation, sync file read) blocks the entire process; no other requests are handled. Use async I/O, worker threads for CPU-heavy work, or offload to a queue.

---

## 9. Quick Revision

- **Middleware**: (req, res, next); order matters; next() or send response.
- **Error middleware**: (err, req, res, next); must be last.
- **req.params** = path segments; **req.query** = query string; **req.body** = body (need express.json()).
- **Async routes**: use try/catch and next(err) or wrapper.
- **Router** for modular routes; mount with app.use("/path", router).
- **Security**: Helmet, CORS, rate limit, validate input.

---

## 10. References

- [Express.js Official Documentation](https://expressjs.com/)
- [Express Best Practices (Node.js)](https://expressjs.com/en/advanced/best-practice-performance.html)
