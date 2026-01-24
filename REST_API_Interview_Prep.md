# REST API Interview Preparation Guide 🚀

This guide provides a comprehensive, interview-focused overview of REST (REpresentational State Transfer). It covers fundamental concepts, best practices, real-world examples, and advanced topics expected from Senior Engineers.

---

## 📖 Brief Overview

### What is REST? (Simple Explanation)
**REST** is an architectural style for designing networked applications. It relies on a stateless, client-server communication protocol—almost always **HTTP**. 

Think of REST as a **language** that allows two computers (a Client and a Server) to talk to each other. The Client asks for a "Resource" (like a user profile or a list of products), and the Server sends back a "Representation" of that resource (usually in JSON format).

### Why REST Matters in Real Projects
In modern software engineering, REST is the backbone of:
- **Microservices**: Allowing independent services to communicate reliably.
- **Mobile & Web Apps**: Providing a unified backend for diverse front-end platforms.
- **Third-Party Integrations**: Most public APIs (Stripe, GitHub, Twilio) are RESTful, making them easy for developers to adopt.

---

## 🧩 Key Concepts

### The 6 Architectural Constraints
To be truly "RESTful," a system must follow these principles:

1.  **Uniform Interface**: A consistent way to interact with the server (using URIs and HTTP methods).
2.  **Client-Server**: The client (UI) and server (data/logic) are separate and can evolve independently.
3.  **Stateless**: Each request from a client must contain all the information needed to understand and process the request. The server does not store "session" state between requests.
4.  **Cacheable**: Responses must define themselves as cacheable or not to improve performance.
5.  **Layered System**: A client cannot tell if it's connected directly to the end server or an intermediary (like a load balancer or proxy).
6.  **Code on Demand (Optional)**: Servers can temporarily extend client functionality by transferring executable code (e.g., JavaScript).

### Common HTTP Status Codes

| Category | Description | Key Examples |
| :--- | :--- | :--- |
| **2xx Success** | Request was successful | `200 OK`, `201 Created`, `204 No Content` |
| **3xx Redirection** | Further action needed | `301 Moved Permanently`, `304 Not Modified` |
| **4xx Client Error** | Problem with the request | `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found` |
| **5xx Server Error** | Problem on the server | `500 Internal Server Error`, `503 Service Unavailable` |

### HTTP Methods (The Verbs)

| Method | Action | Idempotent? | Safe? |
| :--- | :--- | :--- | :--- |
| **GET** | Retrieve a resource | Yes | Yes |
| **POST** | Create a new resource | No | No |
| **PUT** | Replace/Update a resource (entirely) | Yes | No |
| **PATCH** | Update a resource (partially) | No | No |
| **DELETE** | Remove a resource | Yes | No |

> **Interviewer Tip:** *Idempotency* means making the same request multiple times has the same effect as making it once. POST is not idempotent because repeating it creates multiple resources.

---

## 🌍 Real-World Example: E-Commerce System

Imagine a **Product Catalog API** for an online store:

- **Resource**: Products
- **Base URL**: `https://api.store.com/v1`

| Scenario | HTTP Method | Endpoint |
| :--- | :--- | :--- |
| List all products | `GET` | `/products` |
| Get product #42 | `GET` | `/products/42` |
| Add a new product | `POST` | `/products` |
| Update price of #42 | `PATCH` | `/products/42` |
| Delete product #42 | `DELETE` | `/products/42` |

---

## 💻 Code Examples

### 1. HTTP Request & Response (The "Raw" View)

**Request:**
```http
GET /v1/products/42 HTTP/1.1
Host: api.store.com
Accept: application/json
Authorization: Bearer <token>
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 42,
  "name": "Mechanical Keyboard",
  "price": 120.00,
  "currency": "USD",
  "stock_status": "in_stock"
}
```

### 2. Python / FastAPI Implementation

FastAPI is a modern, high-performance framework that makes REST implementation intuitive.

```python
from fastapi import FastAPI, HTTPException, status
from pydantic import BaseModel
from typing import Optional

app = FastAPI()

class Product(BaseModel):
    name: str
    price: float
    description: Optional[str] = None

# Mock Database
db = {1: {"name": "Laptop", "price": 999.99}}

@app.get("/products/{product_id}")
async def get_product(product_id: int):
    if product_id not in db:
        raise HTTPException(status_code=404, detail="Product not found")
    return {"id": product_id, **db[product_id]}

@app.post("/products", status_code=status.HTTP_201_CREATED)
async def create_product(product: Product):
    new_id = len(db) + 1
    db[new_id] = product.dict()
    return {"id": new_id, **db[new_id]}
```

---

## 🚀 Advanced Senior Topics

### 1. API Versioning Strategies
There are three main ways to version a REST API. Each has pros and cons:
- **URI Path (Most Common)**: `https://api.com/v1/users`
    - *Pros*: Easy to see, simple for clients.
    - *Cons*: "Pollutes" the URL; not strictly RESTful (URIs should be permanent).
- **Header Versioning**: `Accept: application/vnd.myapi.v1+json`
    - *Pros*: Clean URLs; more "RESTful."
    - *Cons*: Harder for developers to test in a browser.
- **Query Parameter**: `https://api.com/users?version=1`
    - *Pros*: Simple to implement.
    - *Cons*: Can get messy with other query params.

### 2. Caching & Performance (ETags)
For high-scale systems, caching is critical.
- **ETag (Entity Tag)**: The server sends a hash of the resource.
- **Conditional Requests**: The client sends the ETag back in the `If-None-Match` header.
- **Result**: If the resource hasn't changed, the server returns `304 Not Modified`, saving bandwidth.

### 3. Rate Limiting & Throttling
Protects your system from abuse (DDoS) or accidental loops.
- **Implementation**: Usually done at the API Gateway level (e.g., Kong, AWS API Gateway).
- **Headers**: Return `X-RateLimit-Limit`, `X-RateLimit-Remaining`.
- **Status Code**: `429 Too Many Requests`.

---

## ✅ Best Practices

### Do's 👍
- **Use Nouns, Not Verbs**: `/users` instead of `/getAllUsers`.
- **Use Plural Nouns**: `/products` is standard over `/product`.
- **Versioning**: Always version your API in the URL: `/v1/users`.
- **Logical Nesting**: `/users/12/orders` to get orders for a specific user.
- **Return Correct Status Codes**: Use `201 Created` for POST, `404 Not Found` for missing resources, `401 Unauthorized` for auth issues.
- **Handle Pagination**: Use query params like `/products?page=2&limit=50`.

### Don'ts 👎
- **Don't use GET for sensitive data**: Credentials should never be in the URL.
- **Don't ignore error details**: Return a helpful error body, not just a status code.
- **Don't use CamelCase in URLs**: Kebab-case (`/user-profiles`) or snake_case is preferred.
- **Don't create "God Endpoints"**: Keep resources focused and granular.

---

## ❓ Interview Questions & Answers

**Q1: What is the difference between PUT and PATCH?**
*   **A:** PUT is used to **replace** the entire resource. You must send the full object. PATCH is used for **partial updates**. You only send the fields you want to change.

**Q2: What is HATEOAS?**
*   **A:** It stands for **H**ypermedia **A**s **T**he **E**ngine **O**f **A**pplication **S**tate. It’s a REST constraint where the server provides links in the response, telling the client what other actions are available (e.g., a "Product" response might include a link to "Add to Cart").

**Q3: How do you handle API security in REST?**
*   **A:** Common methods include **JWT (JSON Web Tokens)** for stateless auth, **OAuth2** for delegated authorization, and **API Keys** for simple service-to-service communication. Always use **HTTPS (TLS)** to encrypt data in transit.

**Q4: Explain "Statelessness" and why it is beneficial.**
*   **A:** Statelessness means the server doesn't store any client data between requests. This makes the system highly **scalable** because any server instance can handle any request, simplifying load balancing and recovery.

**Q5: What are "Safe" HTTP methods?**
*   **A:** Safe methods are those that do not modify the state of the server (e.g., GET, HEAD). They are strictly for retrieval.

**Q6: What is "Content Negotiation"?**
*   **A:** It's the mechanism where the client and server agree on the data format (e.g., JSON, XML) using headers like `Accept` and `Content-Type`.

---

## ⚡ Quick Revision

- **REST** = Stateless architectural style using HTTP.
- **Resource** = The entity (User, Order, Post).
- **URI** = The address of the resource.
- **GET** = Read | **POST** = Create | **PUT** = Update/Replace | **DELETE** = Delete.
- **2xx** = Success | **4xx** = Client Error | **5xx** = Server Error.
- **Idempotency** = Multiple identical requests = same result (GET, PUT, DELETE).
- **Stateless** = Server forgets you after every request.

---

## 🔗 References

- [Roy Fielding's Original Dissertation (REST Definition)](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)
- [Microsoft REST API Design Guidelines](https://github.com/microsoft/api-guidelines)
- [RFC 7231 (HTTP/1.1 Semantics)](https://tools.ietf.org/html/rfc7231)
- [FastAPI Documentation](https://fastapi.tiangolo.com/)

---
