# Caching – Interview Notes

> **Target**: Senior Engineers, Backend/API | **Level**: Amazon, Flipkart, Razorpay

---

## 1. Overview

**What it is**: Caching is storing frequently used data in fast storage (usually RAM) so later reads are served without hitting the primary source (DB, API). Reduces latency and load on the backend.

**Why companies use it**: Lower latency for users, higher throughput (fewer DB/API calls), cost savings (less DB load and smaller DB tier). Essential for high-traffic systems.

**When NOT to use it**: Strong consistency requirements where stale data is unacceptable; very low read volume; data that changes every request; when cache invalidation complexity outweighs benefit.

---

## 2. Core Concepts (From PDF)

| Concept | Definition | Simple Explanation | What Interviewers Test | Common Follow-up |
|---------|------------|--------------------|-------------------------|------------------|
| Caching | Storing frequently used data in fast storage | Copy hot data to RAM; serve from there | Why cache, trade-offs | When not to cache |
| Cache Hit | Request served from cache | Data found in cache; no DB call | Hit ratio importance | How to measure hit ratio |
| Cache Miss | Data not in cache | Fallback to DB; optionally populate cache | Miss penalty | Cache stampede |
| Cache Eviction | Removing data from cache | Free space for new entries | Eviction policies | LRU vs LFU |
| Read-Through | Cache layer fetches from DB on miss | App asks cache; cache loads DB | Transparency | Who owns DB fetch |
| Write-Through | Write to cache and DB together | Both updated; strong consistency | Consistency vs performance | When to use |
| Write-Behind | Write to cache; async write to DB | High write throughput; risk of loss | Durability trade-off | When safe to use |
| Cache-Aside | App manages cache and DB | App checks cache; on miss loads DB and fills cache | Most common pattern | Why app-managed |
| TTL | Time To Live | Entry expires after time | Stale data vs freshness | Sliding vs fixed TTL |
| LRU | Least Recently Used | Evict least recently accessed | Default in many caches | LRU implementation |
| LFU | Least Frequently Used | Evict least often accessed | Long-term hot data | LFU vs LRU |
| Strong Consistency | Cache always matches source | Every read sees latest write | When required | Cost of strong consistency |
| Eventual Consistency | Cache syncs later | Stale reads possible for a while | Distributed systems | Invalidation strategy |
| Cache Stampede | Many requests miss same key at once | Thundering herd to DB | Mitigation: locking, coalescing | Single-flight pattern |
| ETag / If-None-Match | HTTP conditional request | 304 Not Modified if unchanged | REST caching | When to use ETag |

---

## 3. How It Works (Internals)

**Cache-Aside flow**  
1. App receives request for key K.  
2. App checks cache for K.  
3. If hit: return cached value.  
4. If miss: read from DB, write to cache (with TTL), return value.  
5. On write: app updates DB, then invalidates or updates cache (delete K or set new value).

**Read-Through flow**  
Cache component implements "get(K)". On miss, cache itself loads from DB (via callback or internal connector), stores in cache, returns. App only talks to cache.

**Eviction**  
When cache is full, eviction policy (e.g. LRU) picks a victim; that entry is removed. New entry is inserted. LRU: doubly linked list + hash map for O(1) get/put and eviction.

**Consistency**  
Write-through: every write goes to DB and cache → strong consistency, higher latency. Cache-aside with invalidation: write DB, delete cache key → next read misses and repopulates; eventual consistency until next read.

**Word diagram**  
```
Client → [App] → [Cache?] → [DB]
              hit: return
              miss: read DB → fill cache → return
```

---

## 4. Real-World Use Case

**E-commerce product catalog API**

- **Where caching fits**: Product details (read-heavy); API response cache (GET /products/:id); CDN for images; Redis for session and recently viewed.
- **Why this approach**: Catalog changes infrequently; TTL (e.g. 5 min) or event-based invalidation on product update; cache-aside in app so product service owns DB and cache keys.
- **Trade-offs**: Stale price/stock for TTL window vs invalidation complexity; cache memory vs DB load; cache stampede on hot key (use single-flight or lock).

---

## 5. Code Example (Minimal but Powerful)

```python
# Cache-Aside with Redis (Python)
import redis
from typing import Optional

r = redis.Redis(host="localhost", port=6379, decode_responses=True)

def get_user(user_id: int) -> Optional[dict]:
    key = f"user:{user_id}"
    # 1. Try cache first
    cached = r.get(key)
    if cached:
        return json.loads(cached)  # Cache hit – no DB call

    # 2. Cache miss – fetch from DB (pseudo)
    user = db.fetch_user(user_id)  # Replace with real DB call
    if not user:
        return None

    # 3. Populate cache with TTL (e.g. 300s)
    r.setex(key, 300, json.dumps(user))  # setex = set + expire
    return user
# Wrong: forgetting TTL → stale forever. Wrong: caching None → cache poisoning.
# Complexity: get O(1), setex O(1); space O(n) for n cached users.
```

**Cache stampede mitigation (single-flight)**  
Use a lock or "promise" so only one request loads from DB; others wait and then read from cache.

```python
# Simple lock per key (pseudo)
with redis_lock(f"lock:user:{user_id}", ttl=5):
    return get_user(user_id)  # Only one caller hits DB
```

---

## 6. Best Practices (Interview-Grade)

| Do | Don't |
|----|--------|
| Set TTL on every cached value | Don't cache without expiry (stale forever) |
| Invalidate or update cache on write | Don't write DB only and leave cache stale |
| Use cache-aside for full control | Don't cache blindly without invalidation strategy |
| Monitor hit ratio and latency | Don't ignore cache effectiveness |
| Use consistent key naming (e.g. `entity:id`) | Don't use ambiguous or colliding keys |
| Consider cache stampede for hot keys | Don't assume low concurrency |

**Security**: Don't cache secrets, tokens, or raw PII without encryption. Validate data before caching to avoid cache poisoning.  
**Scalability**: Distributed cache (e.g. Redis cluster) for multi-instance apps; partition keys to avoid hot spots.

---

## 7. Common Mistakes (Interview Red Flags)

| Mistake | Why wrong | Correct mental model |
|---------|-----------|------------------------|
| "Cache everything" | Memory and staleness; some data must be fresh | Cache read-heavy, slowly changing data; have invalidation plan |
| "We don't need TTL" | Cache never expires; bugs and stale data | Always set TTL; use event invalidation for critical freshness |
| "Write-through is always best" | Higher latency; not always needed | Use when strong consistency is required; else cache-aside + invalidate |
| "Cache and DB always consistent" | With TTL or async invalidation they are not | Design for eventual consistency; document staleness |
| "Ignore cache stampede" | One hot key can take down DB | Use locking, single-flight, or probabilistic early expiry |

---

## 8. Interview Questions & Answers

### A. Basic (Warm-up)

**Q: What is the difference between cache hit and cache miss?**  
**Short answer**: Hit = data found in cache, served from cache. Miss = data not in cache; load from source (e.g. DB) and optionally store in cache.  
**Follow-up**: "Why is hit ratio important?"  
**Ideal answer**: High hit ratio means most requests are served from cache → lower latency and less load on DB. Low hit ratio means cache adds little value or key design/TTL is wrong.

**Q: What is TTL?**  
**Short answer**: Time To Live – how long a cache entry is valid; after TTL it is considered expired and evicted or refetched.  
**Follow-up**: "What is sliding expiry?"  
**Ideal answer**: TTL is reset on each access. Good for "active" data (e.g. session); avoids expiring while user is active. Fixed TTL is simpler and better when you want a fixed freshness window.

### B. Intermediate (Most common)

**Q: Explain Cache-Aside vs Read-Through.**  
**Short answer**: Cache-Aside: application checks cache and DB; on miss, app loads from DB and populates cache. Read-Through: cache layer encapsulates DB; app only asks cache; cache loads DB on miss internally.  
**Follow-up**: "When would you choose one over the other?"  
**Ideal answer**: Cache-Aside when app needs full control (e.g. complex keys, multiple sources). Read-Through when cache is a transparent layer and you want simpler app code.

**Q: How do you avoid cache stampede?**  
**Short answer**: When a key expires, many requests can miss and hit DB at once. Mitigations: (1) lock so only one request loads and others wait; (2) probabilistic early expiry (e.g. refresh before TTL); (3) background refresh.  
**Follow-up**: "What is the downside of locking?"  
**Ideal answer**: Lock contention and latency; if the loader is slow, many requests wait. Use short lock TTL and fallback to stale data or retry.

### C. Advanced / Tricky (Senior-level)

**Q: How do you keep cache and database consistent on writes?**  
**Short answer**: Options: (1) Write-through (write both; strong consistency, higher latency). (2) Write DB then invalidate cache (delete key); next read repopulates (eventual consistency). (3) Write DB then update cache with new value (eventual until update completes).  
**Follow-up**: "What if cache update fails after DB write?"  
**Ideal answer**: Cache holds stale data until TTL or next invalidation. Mitigate with retries, or invalidate (delete) instead of update so next read refetches correct value from DB.

**Q: Design caching for a multi-tenant API where each tenant has different rate limits and permissions.**  
**Ideal answer**: Cache key includes tenant_id (e.g. `ratelimit:tenant_123`, `user:tenant_123:user_456`). Never cache without tenant scope to avoid leaking data. Invalidate per tenant on permission/rate limit change. Consider separate TTL or namespaces per tenant if needed.

---

## 9. Quick Revision

- **Cache hit** = from cache; **miss** = from DB, then fill cache.
- **Cache-Aside**: app manages cache + DB; **Read-Through**: cache loads DB on miss.
- **Write-Through**: write cache + DB; **Write-Behind**: write cache, async DB (risk of loss).
- **TTL** = expiry time; **sliding** = reset on access.
- **LRU** = evict least recently used; **LFU** = least frequently used.
- **Cache stampede** = many misses at once → lock or single-flight.
- **Strong consistency** = cache matches source; **eventual** = stale for a while.
- Don't cache secrets; always set **TTL**; **invalidate on write**.

---

## 10. References

- [Redis Documentation](https://redis.io/docs/)
- [Caching Best Practices (AWS)](https://aws.amazon.com/caching/best-practices/)
