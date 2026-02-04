# FastAPI – Interview Notes

> **Target**: Senior Engineers, Backend (Python) | **Level**: Amazon, Flipkart, Razorpay

---

## 1. Overview

**What it is**: FastAPI is a modern, high-performance Python web framework for building APIs. It uses ASGI, automatic request validation (Pydantic), OpenAPI docs, and async support.

**Why companies use it**: Fast (async, Starlette base); automatic validation and docs; type hints and Pydantic; dependency injection; good for microservices and high-concurrency APIs.

**When NOT to use it**: When you need a full-stack framework with admin and ORM out of the box (consider Django); when team is only familiar with Flask; when you need WSGI-only deployment without ASGI.

---

## 2. Core Concepts (From PDF)

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| FastAPI | High-performance Python API framework | ASGI, Pydantic, OpenAPI | FastAPI vs Flask/Django | |
| ASGI | Asynchronous Server Gateway Interface | Async support; Uvicorn | ASGI vs WSGI | |
| Uvicorn | ASGI server | Run FastAPI in production | Gunicorn + Uvicorn workers | |
| Path operations | @app.get, @app.post | HTTP method + path | Path vs query vs body | |
| Path parameters | /users/{id} | Type conversion (int, etc.) | Automatic validation | |
| Query parameters | ?page=1 | Optional/filter params | Defaults, validation | |
| Request body | Pydantic model | JSON validation | response_model | |
| Pydantic | Data validation via types | Schema + validation | Field constraints | |
| Depends() | Dependency injection | Inject DB, auth, config | Scoped dependencies | |
| yield in dependency | Resource cleanup | Run code after request | When to use | |
| HTTPException | API error response | status_code + detail | Global exception handler | |
| BackgroundTasks | Fire-and-forget tasks | Email, logging | Don't block response | |
| async def | Async endpoint | Concurrency | Blocking in async | |
| Swagger UI / ReDoc | Auto docs | /docs, /redoc | OpenAPI schema | |

---

## 3. How It Works (Internals)

**Request flow**  
Request → Middleware (if any) → Route matching → Dependencies (Depends) → Path operation → Response. Dependencies are resolved in order; if dependency uses yield, code after yield runs after response (cleanup).

**Validation**  
Pydantic models validate request body and path/query params. Invalid input returns 422 with error details. response_model serializes and filters response.

**Async**  
async def endpoints run in event loop; they don't block other requests. Blocking calls (sync DB, CPU) block the loop; use run_in_executor or async DB driver.

**Word diagram**  
```
Request → [Middleware] → [Dependencies] → [Path op] → Response
                              ↓
                    [DB session, auth, etc.]
```

---

## 4. Real-World Use Case

**Internal and partner APIs**

- **Where FastAPI fits**: REST endpoints with validation; auth via Depends (OAuth2, API key); DB via async SQLAlchemy/asyncpg; background tasks for emails/audit; auto docs for partners.
- **Why this approach**: Type safety and validation reduce bugs; async for I/O-bound load; DI makes testing easy; OpenAPI for client generation.
- **Trade-offs**: Async all the way (avoid blocking); need ASGI server (Uvicorn) in production.

---

## 5. Code Example (Minimal but Powerful)

```python
# FastAPI – path/query/body, validation, dependency, error
from fastapi import FastAPI, Depends, HTTPException, Query
from pydantic import BaseModel

app = FastAPI()

class UserCreate(BaseModel):
    name: str
    email: str

class UserResponse(BaseModel):
    id: int
    name: str
    email: str

# Path param (auto int conversion); query with validation
@app.get("/users/{user_id}", response_model=UserResponse)
async def get_user(
    user_id: int,  # Path – 422 if not int
    include_extra: bool = Query(False),
):
    user = await db.get_user(user_id)  # Assume async DB
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    return user

# Body validation via Pydantic
@app.post("/users", status_code=201, response_model=UserResponse)
async def create_user(body: UserCreate):
    # body.name, body.email validated
    user = await db.create_user(body)
    return user

# Dependency injection
def get_db():
    db = Session()
    try:
        yield db
    finally:
        db.close()

@app.get("/users")
async def list_users(db = Depends(get_db)):
    return await db.get_users()
# Wrong: blocking call in async def → blocks event loop.
# Wrong: no response_model → may leak internal fields.
# Complexity: validation O(n); async allows high concurrency.
```

---

## 6. Best Practices (Interview-Grade)

| Do | Don't |
|----|--------|
| Use Pydantic for all request/response | Don't use raw dict without validation |
| Use Depends for DB, auth, config | Don't create DB session in route manually |
| Use async def for I/O-bound routes | Don't do blocking I/O in async without run_in_executor |
| Set response_model to control output | Don't return full ORM objects (leak, N+1) |
| Raise HTTPException for API errors | Don't return 200 with error body for errors |
| Use yield in dependency for cleanup | Don't forget to close sessions/connections |

**Security**: OAuth2PasswordBearer, API key in header; validate and sanitize input; never log secrets.  
**Performance**: Async DB driver; Redis for cache; pagination for lists.

---

## 7. Common Mistakes (Interview Red Flags)

| Mistake | Why wrong | Correct mental model |
|---------|-----------|------------------------|
| "FastAPI and Flask are the same" | FastAPI is ASGI, Pydantic, DI; Flask is WSGI, minimal | Use FastAPI for new APIs; Flask for legacy or simple apps |
| "We can use sync code in async def" | Sync call blocks event loop; no concurrency | Use async libs or run_in_executor for blocking |
| "Dependencies run after the route" | Dependencies run before; yield cleanup runs after | Use yield for cleanup (close DB, etc.) |
| "response_model is optional" | Without it, all fields (including internal) are serialized | Set response_model to control and document output |
| "HTTPException is the only way to error" | For expected API errors yes; for unexpected use exception handler | Use HTTPException for 4xx/5xx; handler for unhandled |

---

## 8. Interview Questions & Answers

### A. Basic (Warm-up)

**Q: What is the difference between path and query parameters in FastAPI?**  
**Short answer**: Path params are part of the URL path (/users/1 → user_id=1). Query params are after ? (/users?page=1). Path for identity, query for optional/filters.  
**Follow-up**: "How does FastAPI validate them?"  
**Ideal answer**: Type hints (e.g. user_id: int) and Query() for query. Invalid values return 422 with detail. Pydantic under the hood.

**Q: What is Depends() used for?**  
**Short answer**: Dependency injection. Inject DB session, auth, config into route. FastAPI resolves dependencies and caches per request by default.  
**Follow-up**: "When would you use yield in a dependency?"  
**Ideal answer**: When you need cleanup after the request (e.g. close DB session, release lock). Code after yield runs after the response is sent.

### B. Intermediate (Most common)

**Q: Why use async def in FastAPI?**  
**Short answer**: Async allows the server to handle other requests while waiting for I/O (DB, HTTP). Higher concurrency without threads.  
**Follow-up**: "What if we call a blocking function inside async def?"  
**Ideal answer**: It blocks the event loop; no other requests are processed during that time. Use async DB driver or run_in_executor for blocking code.

**Q: How does request validation work?**  
**Short answer**: Pydantic models and type hints. Body is validated against the model; path/query against parameter types. Invalid input → 422 with error list.  
**Follow-up**: "How do you add custom validation?"  
**Ideal answer**: Pydantic validators (@validator, @field_validator) or custom types. For complex rules, validator in the model.

### C. Advanced / Tricky (Senior-level)

**Q: How would you structure a large FastAPI app?**  
**Ideal answer**: APIRouter per domain (users, orders); include_routers in main app. Dependencies in a deps module. Pydantic schemas in schemas/. Config via pydantic-settings. Separate router for /api/v1. Test with TestClient and overridden dependencies.

**Q: How do you run FastAPI in production?**  
**Ideal answer**: ASGI server: Uvicorn (or Hypercorn). For multiple workers: Gunicorn with Uvicorn worker class. Behind reverse proxy (Nginx) for TLS and static. Use env for config; don't run with reload in prod.

---

## 9. Quick Revision

- **Path params** = URL path; **query** = ?key=value; **body** = Pydantic model.
- **Depends()** = DI; **yield** in dependency = cleanup after response.
- **async def** for I/O; avoid **blocking** in async.
- **HTTPException** for API errors; **response_model** to shape output.
- **Uvicorn** = ASGI server; **/docs** = Swagger.
- **Validation** via Pydantic; **422** for invalid input.

---

## 10. References

- [FastAPI Documentation](https://fastapi.tiangolo.com/)
- [Pydantic Documentation](https://docs.pydantic.dev/)
