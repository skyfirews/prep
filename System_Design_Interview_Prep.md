# System Design – Interview Notes

> **Target**: Senior Engineers, Technical Architects | **Level**: Amazon, Flipkart, Razorpay

---

## 1. Overview

**What it is**: System design is the process of defining architecture, components, and trade-offs for a system that meets functional and non-functional requirements (scale, latency, availability).

**Why companies care**: Interviews test how you handle ambiguity, trade-offs, and real-world constraints. Production systems need clear requirements, scalable design, and operability.

**When NOT to over-design**: For small scope or MVP, avoid unnecessary complexity. Design for expected scale and iterate; don't assume Google-scale on day one.

---

## 2. Core Concepts (From PDF)

### Requirements

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Functional requirements | What the system should do | Features, user actions | Core features | |
| Non-functional requirements | How the system should behave | Scale, latency, availability, consistency | SLAs, trade-offs | |
| Scale | Load and data size | QPS, users, storage | How to scale | |
| Latency | Response time | p50, p99 | Where latency comes from | |
| Availability | Uptime | 99.9%, etc. | How to achieve | |

### Architecture

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Monolith | Single deployable unit | One codebase, one deploy | Pros and cons | When to break up |
| Microservices | Independent services | Deploy and scale per service | When to use | Communication |
| Event-driven | Async via events | Message queue, pub/sub | Decoupling | Consistency |
| API Gateway | Single entry for clients | Auth, rate limit, routing | Why use | |
| Load Balancer | Distribute traffic | Horizontal scaling | ALB vs NLB | |
| Auto Scaling | Dynamic capacity | Add/remove instances | Cost vs availability | |

### Data & Consistency

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| SQL (RDBMS) | Relational, ACID | Strong consistency, transactions | When to use | |
| NoSQL | Non-relational | Scale, flexible schema | CAP, when to use | |
| Caching | Fast read layer | Redis, in-memory | Invalidation | |
| CDN | Edge caching | Static assets, global | When to use | |
| Strong consistency | All see latest write | Costly in distributed system | When required | |
| Eventual consistency | All see latest eventually | Stale reads possible | When acceptable | |
| CAP | Consistency, Availability, Partition tolerance | Pick two in partition | How to interpret | |

### Security & Observability

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Authentication | Who are you | JWT, OAuth2 | Stateless auth | |
| Authorization | What can you do | RBAC | How to implement | |
| Logging | Centralized logs | Debug, audit | Structure, levels | |
| Monitoring | Metrics and alerts | SLIs, SLOs | What to monitor | |
| Deployment | CI/CD, Blue-Green, Canary | Zero downtime | Rollback | |
| Disaster recovery | Backup, failover | RTO, RPO | Multi-region | |

---

## 3. How It Works (Internals)

**High-level flow**  
Clients (web/mobile) → Load Balancer → API Gateway (auth, rate limit) → Services (stateless) → Data stores (DB, cache, queue). Cache and CDN reduce load and latency.

**Scaling**  
Vertical: bigger machine. Horizontal: more instances behind LB; stateless app; DB scale via replicas (read) or sharding (write). Cache and async (queues) decouple and absorb load.

**Consistency**  
Strong: synchronous replication or single leader; higher latency. Eventual: async replication; stale reads; lower latency and higher availability. Choose by business (e.g. payment = strong; feed = eventual).

**Word diagram – simple API system**  
```
[Clients] → [CDN?] → [LB] → [API Gateway] → [App Servers] → [DB]
                              ↓                    ↓
                         [Auth, Rate Limit]    [Cache]
```

---

## 4. Real-World Use Case

**E-commerce platform**

- **Functional**: Browse catalog, search, cart, checkout, orders, user account.
- **Non-functional**: High read (catalog), spike (sales); payment strong consistency; catalog eventual OK; availability 99.9%.
- **Where design fits**: API Gateway (auth, rate limit); catalog service + cache (Redis) + CDN for static; order service + SQL for orders and payment; queue for async (email, inventory); search (Elasticsearch or similar); DB replicas for read scaling; backup and failover for DR.
- **Trade-offs**: Monolith vs microservices (start simple, split when needed); strong vs eventual consistency by use case; cost vs redundancy.

---

## 5. Code Example (Minimal but Powerful)

```python
# Not code-heavy; system design is architecture.
# Example: idempotent API with rate limit (concept)

# Idempotency: same request twice = same effect once
# Client sends Idempotency-Key: uuid
# Server: if key seen, return cached response; else process and cache response for key

# Rate limit: token bucket or sliding window per user/IP
# Store in Redis: key = user_id, value = count, TTL = window
# If count > limit → 429 Too Many Requests
```

---

## 6. Best Practices (Interview-Grade)

| Do | Don't |
|----|--------|
| Clarify requirements (scale, consistency, latency) | Don't assume requirements |
| Start simple (one service, one DB) then scale | Don't over-engineer upfront |
| Draw components and data flow | Don't jump to details without picture |
| Discuss trade-offs (CAP, cost, complexity) | Don't ignore trade-offs |
| Consider failure (single point of failure, DR) | Don't assume nothing fails |
| Mention monitoring, logging, deployment | Don't forget operability |

**Security**: Auth at gateway; least privilege; encrypt in transit and at rest.  
**Scalability**: Stateless app; horizontal scaling; cache and CDN; async where possible.

---

## 7. Common Mistakes (Interview Red Flags)

| Mistake | Why wrong | Correct mental model |
|---------|-----------|------------------------|
| "We need microservices from day one" | Complexity and ops cost; monolith is often enough | Start monolith; split when team or scale demands |
| "NoSQL is always more scalable" | Depends on access pattern; SQL scales with replicas/sharding | Choose by consistency and access pattern |
| "Eventual consistency everywhere" | Payment and inventory need strong consistency | Strong where business requires; eventual elsewhere |
| "We don't need a load balancer" | Single server = single point of failure | LB + multiple app instances for HA |
| "Design for 1M QPS from start" | Over-engineering; iterate with real numbers | Design for stated scale; mention how to scale further |

---

## 8. Interview Questions & Answers

### A. Basic (Warm-up)

**Q: What is the difference between functional and non-functional requirements?**  
**Short answer**: Functional = what the system does (features). Non-functional = how it behaves (scale, latency, availability, security).  
**Follow-up**: "Give examples of non-functional requirements."  
**Ideal answer**: Latency p99 < 200ms, availability 99.9%, 10K QPS, data durability 99.999999999%, compliance (GDPR, PCI).

**Q: What is a load balancer?**  
**Short answer**: Distributes incoming traffic across multiple servers. Enables horizontal scaling and high availability; one server down, others serve.  
**Follow-up**: "What is the difference between L4 and L7 load balancer?"  
**Ideal answer**: L4 (e.g. NLB) works on TCP/UDP; L7 (e.g. ALB) on HTTP (path, host, headers). L7 can do path-based routing and TLS termination.

### B. Intermediate (Most common)

**Q: When would you use SQL vs NoSQL?**  
**Short answer**: SQL when you need strong consistency, transactions, complex queries, and relationships. NoSQL when you need horizontal scale, flexible schema, or specific access patterns (key-value, document).  
**Follow-up**: "What about CAP theorem?"  
**Ideal answer**: In partition: CP (consistent but may be unavailable) vs AP (available but may be inconsistent). SQL often CP; NoSQL often AP or tunable. Choose by business need.

**Q: How do you achieve high availability?**  
**Short answer**: No single point of failure: multiple app instances behind LB; DB with replicas and failover; multi-AZ or multi-region; health checks and auto-recovery; deployment with zero downtime (Blue-Green, Canary).  
**Follow-up**: "What is the difference between multi-AZ and multi-region?"  
**Ideal answer**: Multi-AZ = same region, sync or fast failover; multi-region = DR and global latency; async replication; higher cost and complexity.

### C. Advanced / Tricky (Senior-level)

**Q: Design a URL shortener.**  
**Ideal answer**: Requirements: shorten, redirect, scale (high read). Components: API (create short URL, redirect); storage (short code → long URL); redirect (302 to long URL). Short code: base62 of id or hash. DB: key-value or SQL with index. Cache (Redis) for hot URLs. Optional: analytics in async pipeline. Scale: shard by short code or use distributed ID.

**Q: Design a chat system (e.g. 1:1 and groups).**  
**Ideal answer**: Clients; API for send message, get history; real-time via WebSocket or long polling. Messages stored in DB (per room or global with room_id); cache recent messages. Real-time: WebSocket server or pub/sub (e.g. Redis); each room = channel; online presence via heartbeat and Redis. Scale: multiple WS servers; pub/sub for cross-server delivery; DB shard by room or time.

---

## 9. Quick Revision

- **Functional** = what; **non-functional** = how (scale, latency, availability).
- **Monolith** = one unit; **microservices** = many services; split when needed.
- **LB** = distribute traffic; **API Gateway** = auth, rate limit, route.
- **SQL** = strong consistency, transactions; **NoSQL** = scale, flexibility.
- **Cache** = fast read; **CDN** = edge static; **queue** = async.
- **Strong** vs **eventual** consistency by use case.
- **HA** = no single point of failure; **DR** = backup, failover.

---

## 10. References

- [Designing Data-Intensive Applications (book)](https://dataintensive.net/)
- [System Design Primer (GitHub)](https://github.com/donnemartin/system-design-primer)
- [High Scalability blog](http://highscalability.com/)
