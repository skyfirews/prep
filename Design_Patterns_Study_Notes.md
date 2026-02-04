# Design Patterns – Comprehensive Study Notes

> **Purpose**: Exam prep & interview revision | **Style**: Beginner-friendly, technically correct

---

## What Are Design Patterns?

**Design patterns** are reusable solutions to common problems in software design. They give you a shared vocabulary and proven structure so code stays **reusable**, **flexible**, and **maintainable**.

### Three Categories

| Category | Focus |
|----------|--------|
| **Creational** | How objects are created |
| **Structural** | How objects are composed/related |
| **Behavioral** | How objects interact and communicate |

---

## 1. Singleton Pattern

### Layman explanation
There is **only one instance** of something in the whole system—like a single master key for a building. Everyone uses that same key; no duplicates.

### Problem it solves
You need exactly one instance of a class (e.g. one config, one connection pool, one print spooler). Creating multiple instances would waste resources or cause inconsistent state.

### When to use
- Configuration / settings object shared app-wide  
- Database connection pool manager  
- Logging service  
- Cache manager (single shared cache)

### When NOT to use
- When you need multiple instances or test with mocks (Singleton makes testing harder)  
- When “single instance” is only per process/thread (use scoped services instead)  
- When you use it “because it’s easy”—prefer dependency injection for shared services

### Real-world (non-technical) example
**Building manager + one main key**: Only one key to the main entrance. Everyone uses that key; duplicates would cause security and consistency issues.

### Real-time software example
**App config in a microservice**: One service reads config (DB URL, API keys) at startup. All handlers use that same config object so settings are consistent and loaded once.

### Python code example

```python
class AppConfig:
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialized = False
        return cls._instance

    def __init__(self):
        if self._initialized:
            return
        self._initialized = True
        self.debug = False
        self.api_key = ""

# Usage
config1 = AppConfig()
config2 = AppConfig()
assert config1 is config2  # Same instance
```

### Key advantages
- Single point of access; no duplicate state  
- Lazy creation possible (create on first use)  
- Familiar pattern; easy to explain

### Common mistakes / pitfalls
- **Thread safety**: Plain Singleton is not thread-safe; use locks or module-level instance in Python  
- **Testing**: Hard to replace with a mock; prefer injecting the “single” dependency  
- **Overuse**: Don’t make everything a Singleton; use for true global single instance only

---

## 2. Factory Method Pattern

### Layman explanation
You ask a **factory** for “a pizza” or “a shoe”; the factory decides the exact type and gives you the right product. You don’t build it yourself.

### Problem it solves
You have a family of similar types (e.g. different payment handlers, notification channels). You don’t want callers to depend on concrete classes or repeat creation logic everywhere.

### When to use
- Multiple similar types; choice depends on config, user input, or environment  
- You want to add new types without changing client code  
- Creation logic is non-trivial (validation, wiring dependencies)

### When NOT to use
- Only one concrete type and no plan to add more  
- Creation is a one-liner with no logic  
- You need runtime parameters that don’t fit a single “product type” (consider Builder or Abstract Factory)

### Real-world (non-technical) example
**Pizza shop**: You order “pepperoni” or “margherita.” The kitchen (factory) makes that type; you don’t go into the kitchen or know the recipe.

### Real-time software example
**Payment gateway**: User chooses “card” / “UPI” / “wallet.” A factory returns the right payment handler. New methods (e.g. BNPL) = new class + factory branch; API and controllers stay unchanged.

### Python code example

```python
from abc import ABC, abstractmethod

class Notification(ABC):
    @abstractmethod
    def send(self, message: str) -> None:
        pass

class EmailNotification(Notification):
    def send(self, message: str) -> None:
        print(f"Email: {message}")

class SMSNotification(Notification):
    def send(self, message: str) -> None:
        print(f"SMS: {message}")

def create_notification(channel: str) -> Notification:
    if channel == "email":
        return EmailNotification()
    if channel == "sms":
        return SMSNotification()
    raise ValueError(f"Unknown channel: {channel}")

# Usage
notifier = create_notification("email")
notifier.send("Order confirmed")
```

### Key advantages
- Decouples client from concrete classes  
- New types added in one place (factory)  
- Easier testing (inject a factory or stub products)

### Common mistakes / pitfalls
- **God factory**: One giant factory with many `if/elif`; split by domain or use registry  
- **Leaking concrete types**: Client should depend on the abstract type (e.g. `Notification`), not `EmailNotification`  
- **Creation in wrong layer**: Factory should sit where “policy” is (e.g. app layer), not deep inside libraries

---

## 3. Adapter Pattern

### Layman explanation
Two things don’t fit together (e.g. European plug vs US socket). An **adapter** sits in between and makes one side look like what the other expects.

### Problem it solves
You have an existing component (or external API) with interface A, but your code expects interface B. Rewriting the component is costly or impossible; you need them to work together.

### When to use
- Integrating a third-party or legacy library with a different interface  
- Making one API match another (e.g. internal vs external contract)  
- Wrapping external services so the rest of the app sees a single, consistent interface

### When NOT to use
- You control both sides; change one interface instead of adding an adapter  
- The “adaptation” is really business logic; put it in a service, not in the adapter  
- You’re only renaming methods; a thin wrapper might be enough, but don’t call it Adapter if there’s no real interface mismatch

### Real-world (non-technical) example
**Travel adapter**: European charger (230V, two pins) vs US outlet (120V, different shape). The adapter changes shape and optionally voltage so the charger works.

### Real-time software example
**Legacy billing API**: Old system returns XML and uses `get_invoice(id)`. New app expects JSON and `fetch_invoice(invoice_id)`. An adapter wraps the old client, calls `get_invoice`, and maps XML → JSON and `id` → `invoice_id`.

### Python code example

```python
# Legacy service (incompatible interface)
class LegacyPaymentService:
    def pay_now(self, amount_dollars: float, user_id: str) -> bool:
        return True  # simplified

# Interface our app expects
class PaymentGateway(ABC):
    @abstractmethod
    def charge(self, amount_cents: int, customer_id: str) -> bool:
        pass

class LegacyPaymentAdapter(PaymentGateway):
    def __init__(self, legacy: LegacyPaymentService):
        self._legacy = legacy

    def charge(self, amount_cents: int, customer_id: str) -> bool:
        amount_dollars = amount_cents / 100.0
        return self._legacy.pay_now(amount_dollars, customer_id)

# Usage
legacy = LegacyPaymentService()
gateway = LegacyPaymentAdapter(legacy)
gateway.charge(1999, "user_123")  # $19.99 in cents
```

### Key advantages
- Reuse existing code without changing it  
- Single place to map between two interfaces  
- Keeps “translation” logic out of business code

### Common mistakes / pitfalls
- **Adapter doing too much**: Only translate interface and delegate; no business rules or heavy logic  
- **Many adapters for one type**: Can indicate a messy or unstable target interface  
- **Forgetting error mapping**: Map legacy exceptions to domain errors the rest of the app expects

---

## 4. Observer Pattern

### Layman explanation
One source (e.g. news channel) and many subscribers. When something happens, the source **notifies** all subscribers. Subscribers don’t poll; they get updates when they occur.

### Problem it solves
When one object changes state, others need to react (e.g. UI updates when model changes, caches invalidated when data changes). You don’t want the “subject” to know every dependent by name; you want loose coupling.

### When to use
- Event-driven UIs (model → views)  
- Pub/sub style updates (price changes, order status)  
- Decoupling producers from consumers (microservices, in-process events)

### When NOT to use
- Only one observer and tight coupling is acceptable  
- Order of reaction is critical and complex (consider a small pipeline or state machine)  
- High-frequency events with many observers (consider batching, async, or a message queue instead of synchronous notify)

### Real-world (non-technical) example
**YouTube subscriptions**: You subscribe to a channel. When a new video is published, you get a notification. You don’t refresh the channel page constantly.

### Real-time software example
**E-commerce order status**: Order service updates status to “Shipped.” Observers: send email, update analytics, refresh inventory dashboard. Order service only “emits event”; observers are registered elsewhere (e.g. event bus or in-process list).

### Python code example

```python
from abc import ABC, abstractmethod

class OrderSubject:
    def __init__(self):
        self._observers = []
        self._status = "pending"

    def attach(self, observer: "OrderObserver") -> None:
        self._observers.append(observer)

    def set_status(self, status: str) -> None:
        self._status = status
        for obs in self._observers:
            obs.on_status_change(self._status)

class OrderObserver(ABC):
    @abstractmethod
    def on_status_change(self, status: str) -> None:
        pass

class EmailNotifier(OrderObserver):
    def on_status_change(self, status: str) -> None:
        print(f"[Email] Order status: {status}")

class AnalyticsTracker(OrderObserver):
    def on_status_change(self, status: str) -> None:
        print(f"[Analytics] Recorded: {status}")

# Usage
order = OrderSubject()
order.attach(EmailNotifier())
order.attach(AnalyticsTracker())
order.set_status("shipped")  # Both observers notified
```

### Key advantages
- Subject doesn’t know concrete observers; easy to add/remove  
- Broadcasts one change to many listeners  
- Fits event-driven and reactive designs

### Common mistakes / pitfalls
- **Memory leaks**: Unregister observers when they’re no longer needed (e.g. view closed)  
- **Long work in observer**: Notify should be fast; offload heavy work to a queue or thread  
- **No clear contract**: Define what “event” carries (e.g. status string vs full event object) so observers don’t depend on internals

---

## 5. Decorator Pattern

### Layman explanation
You start with a **base thing** (e.g. plain coffee). You **wrap** it with extras (milk, sugar) one by one. The base stays the same; each wrapper adds one kind of behavior or data.

### Problem it solves
You want to add responsibilities to an object at **runtime** (e.g. logging, retry, compression) without changing its class or creating a huge inheritance tree.

### When to use
- Add optional behavior (caching, logging, retry) around a core component  
- Compose behavior in different combinations (A + B, A + C, A + B + C)  
- When subclassing would explode (many combinations)

### When NOT to use
- Only one fixed “wrapper” and no composition; a simple wrapper function or subclass may be enough  
- You need to remove or reorder behavior at runtime in complex ways; consider a pipeline or chain instead  
- When it obscures the real type (e.g. type checkers see “Decorator” not “Service”); document or use interfaces

### Real-world (non-technical) example
**Coffee + toppings**: Base = black coffee. Add milk (same cup, now “with milk”). Add sugar (same cup, now “with milk and sugar”). Core is still “coffee”; each step adds one thing.

### Real-time software example
**HTTP client**: Core client does the request. Wrap with `LoggingDecorator` (log request/response), then `RetryDecorator` (retry on 5xx), then `CacheDecorator` (cache GETs). Same interface; behavior added in layers.

### Python code example

```python
from abc import ABC, abstractmethod

class DataSource(ABC):
    @abstractmethod
    def read(self) -> str:
        pass

class FileDataSource(DataSource):
    def __init__(self, path: str):
        self._path = path

    def read(self) -> str:
        return "raw data from file"

class CompressionDecorator(DataSource):
    def __init__(self, inner: DataSource):
        self._inner = inner

    def read(self) -> str:
        return "decompressed(" + self._inner.read() + ")"

class EncryptionDecorator(DataSource):
    def __init__(self, inner: DataSource):
        self._inner = inner

    def read(self) -> str:
        return "decrypted(" + self._inner.read() + ")"

# Usage: file -> decrypt -> decompress
source = FileDataSource("data.bin")
source = EncryptionDecorator(source)
source = CompressionDecorator(source)
print(source.read())  # decrypted(decompressed(raw data from file))
```

### Key advantages
- Add/remove behavior without changing existing classes  
- Compose in any order; follows open/closed principle  
- Single responsibility per decorator

### Common mistakes / pitfalls
- **Many small decorators**: Can make stack traces and debugging noisy; group related behavior when it makes sense  
- **Wrong order**: Order matters (e.g. decrypt then decompress); document or use a builder that enforces order  
- **State in decorators**: Prefer stateless decorators; if state is needed, be explicit and thread-safe

---

## 6. Proxy Pattern

### Layman explanation
A **proxy** is a stand-in that sits in front of the real object. It can check permissions, cache results, or delay talking to the real object until needed—like a receptionist who filters calls to the boss.

### Problem it solves
You need to **control access** to an object (lazy load, access check, audit) or **reduce cost** (caching, batching) without the client knowing. The client uses the same interface as the real object.

### When to use
- Lazy initialization (create heavy object only when first used)  
- Access control (permissions, rate limiting)  
- Caching, logging, or auditing around the real object  
- Remote or expensive resource (proxy talks to the real thing over network)

### When NOT to use
- No need for control or indirection; use the real object directly  
- You only need one extra step (e.g. log then call); a simple wrapper might be enough  
- When “proxy” is actually a full-fledged service (e.g. API gateway); call it by its real name to avoid confusion

### Real-world (non-technical) example
**Security guard at a building**: Visitors don’t walk straight to the CEO. The guard checks ID, purpose, and appointment; only then allows access. The guard is the proxy for “access to the building.”

### Real-time software example
**Image loader in an app**: List view shows thumbnails. A proxy implements the same “get image” interface: first return a placeholder and trigger load; when the real image is loaded, replace the placeholder. Heavy download and decoding stay behind the proxy.

### Python code example

```python
from abc import ABC, abstractmethod

class ImageLoader(ABC):
    @abstractmethod
    def load(self, path: str) -> str:
        pass

class RealImageLoader(ImageLoader):
    def load(self, path: str) -> str:
        print(f"Loading from disk: {path}")
        return f"pixel_data_{path}"

class ImageLoaderProxy(ImageLoader):
    def __init__(self):
        self._loader = None
        self._cache = {}

    def load(self, path: str) -> str:
        if path in self._cache:
            print(f"Cache hit: {path}")
            return self._cache[path]
        if self._loader is None:
            self._loader = RealImageLoader()
        data = self._loader.load(path)
        self._cache[path] = data
        return data

# Usage
proxy = ImageLoaderProxy()
proxy.load("photo.jpg")  # Loads from disk
proxy.load("photo.jpg")  # From cache
```

### Key advantages
- Same interface as real object; client unchanged  
- Lazy init, caching, and access control in one place  
- Can swap real implementation (e.g. mock in tests)

### Common mistakes / pitfalls
- **Proxy too smart**: Proxy should delegate; move business logic to the real object or a service  
- **Ignoring lifecycle**: Cached or lazy-built objects may need cleanup (e.g. file handles, connections)  
- **Thread safety**: Shared cache or lazy init must be safe under concurrency

---

## 7. Facade Pattern

### Layman explanation
A **facade** is one simple front (e.g. a waiter) behind which many subsystems exist (kitchen, cashier, inventory). You use one simple interface; complexity is hidden inside.

### Problem it solves
A set of classes or APIs is hard to use correctly (order of calls, defaults, error handling). You want a **single, simple interface** that does the right thing for common cases and hides the rest.

### When to use
- Simplify a complex library or legacy module  
- Provide a “default way” to use several subsystems together  
- Reduce coupling between clients and internal modules  
- Build a clear API layer (e.g. “orchestration” facade over many services)

### When NOT to use
- Clients need fine-grained control over subsystems; don’t hide necessary knobs behind a facade  
- There’s only one subsystem; a thin wrapper is enough  
- The “facade” is doing business logic; then it’s a service or use case, not just a structural facade

### Real-world (non-technical) example
**Restaurant waiter**: You order and pay through the waiter. You don’t talk to the kitchen, cashier, or storeroom. The waiter is the single interface to the whole system.

### Real-time software example
**Onboarding API**: “Create user” might require: validate input, create user in DB, send welcome email, create default workspace, enqueue analytics. A single `UserOnboardingFacade.create_user(email, name)` does these steps in the right order; callers don’t touch DB, email, or queue directly.

### Python code example

```python
class InventoryService:
    def reserve(self, product_id: str, qty: int) -> bool:
        print(f"Reserved {qty} of {product_id}")
        return True

class PaymentService:
    def charge(self, amount_cents: int, token: str) -> bool:
        print(f"Charged {amount_cents} cents")
        return True

class ShippingService:
    def schedule(self, order_id: str, address: str) -> None:
        print(f"Scheduled shipping for {order_id} to {address}")

class OrderFacade:
    def __init__(self):
        self._inventory = InventoryService()
        self._payment = PaymentService()
        self._shipping = ShippingService()

    def place_order(self, product_id: str, qty: int, payment_token: str, address: str) -> str:
        if not self._inventory.reserve(product_id, qty):
            raise ValueError("Out of stock")
        order_id = "ord_123"
        if not self._payment.charge(999 * qty, payment_token):
            raise ValueError("Payment failed")
        self._shipping.schedule(order_id, address)
        return order_id

# Usage: one call instead of three services
facade = OrderFacade()
order_id = facade.place_order("prod_1", 2, "tok_xxx", "123 Main St")
```

### Key advantages
- Simpler API for clients; fewer mistakes  
- Changes inside subsystems don’t force client changes  
- Clear “entry point” for a feature or subsystem

### Common mistakes / pitfalls
- **Facade becomes a god object**: If it grows too much, split by use case or domain  
- **Hiding what shouldn’t be hidden**: Expose options or hooks when callers need control  
- **Duplicating logic**: Facade should orchestrate, not reimplement; delegate to subsystems

---

## 8. Strategy Pattern

### Layman explanation
You have **several ways** to do the same thing (e.g. reach a destination: walk, drive, bus). You **choose one strategy** based on the situation and use it; the “navigator” doesn’t care how the route is computed.

### Problem it solves
You have multiple algorithms or behaviors for the same goal (e.g. sort by name vs by date, compress with gzip vs zip). You want to **swap them at runtime** without branching everywhere and without changing the code that uses them.

### When to use
- Multiple interchangeable algorithms (validation, pricing, routing)  
- You want to avoid large `if/elif` or switch on “type”  
- Runtime selection (user preference, A/B test, config)  
- Easy to add new strategies without touching existing code

### When NOT to use
- Only one fixed algorithm; no need for indirection  
- Strategies need lots of context or shared state; consider a pipeline or a small state object instead  
- Selection is compile-time and never changes; a simple function or module may be enough

### Real-world (non-technical) example
**Navigation app**: You choose “shortest distance,” “fastest time,” or “avoid tolls.” The app uses that strategy to compute the route. Same “navigate” action; different way of computing it.

### Real-time software example
**Pricing engine**: Same “compute price” entry point; strategies: “default,” “promo,” “B2B contract,” “subscription discount.” New pricing rule = new strategy; rest of checkout unchanged.

### Python code example

```python
from abc import ABC, abstractmethod

class PricingStrategy(ABC):
    @abstractmethod
    def compute(self, base_price: float, qty: int) -> float:
        pass

class DefaultPricing(PricingStrategy):
    def compute(self, base_price: float, qty: int) -> float:
        return base_price * qty

class BulkDiscountPricing(PricingStrategy):
    def compute(self, base_price: float, qty: int) -> float:
        total = base_price * qty
        return total * 0.9 if qty >= 10 else total

class Checkout:
    def __init__(self, strategy: PricingStrategy):
        self._strategy = strategy

    def set_strategy(self, strategy: PricingStrategy) -> None:
        self._strategy = strategy

    def total(self, base_price: float, qty: int) -> float:
        return self._strategy.compute(base_price, qty)

# Usage
checkout = Checkout(DefaultPricing())
print(checkout.total(10.0, 5))   # 50.0
checkout.set_strategy(BulkDiscountPricing())
print(checkout.total(10.0, 12))  # 108.0 (10% off)
```

### Key advantages
- Easy to add new behaviors (new strategy class)  
- Removes conditionals from client code  
- Test each strategy in isolation

### Common mistakes / pitfalls
- **Strategy needs too much context**: Pass a small context object; avoid strategies that pull from globals or many sources  
- **Too many strategies**: If you have dozens, consider grouping or a different model (e.g. rules engine)  
- **Copy-paste between strategies**: Shared logic should live in a base class or helper, not duplicated

---

## 9. Composite Pattern

### Layman explanation
**Files and folders**: A folder can contain files and other folders. You “list” or “delete” a folder the same way you do for a file—treat single items and groups **uniformly**.

### Problem it solves
You have a **tree** of objects (part–subpart, menu–submenu, shape–group). You want to treat leaves and branches with the same interface so client code can work recursively without special cases.

### When to use
- Hierarchical structures (menus, file systems, org charts, UI containers)  
- Operations that apply to “one or many” the same way (render, total size, validate)  
- Building trees where nodes can be either leaf or container

### When NOT to use
- No real hierarchy; flat list is enough  
- Leaves and branches behave very differently; forcing one interface can be awkward  
- Tree is tiny and fixed; a couple of nested loops might be clearer

### Real-world (non-technical) example
**Folder structure**: A folder has “size” (sum of contents) and “delete” (remove itself and contents). A file has “size” and “delete.” You treat both as “items” that support the same operations.

### Real-time software example
**Permission tree**: “Role” can be a single permission or a group of roles. “Can user do X?” is asked on the root; it recurses into children. Same interface for leaf permission and group.

### Python code example

```python
from abc import ABC, abstractmethod

class Component(ABC):
    @abstractmethod
    def size(self) -> int:
        pass

    @abstractmethod
    def list_paths(self) -> list[str]:
        pass

class File(Component):
    def __init__(self, name: str, size_bytes: int):
        self._name = name
        self._size_bytes = size_bytes

    def size(self) -> int:
        return self._size_bytes

    def list_paths(self) -> list[str]:
        return [self._name]

class Folder(Component):
    def __init__(self, name: str):
        self._name = name
        self._children: list[Component] = []

    def add(self, child: Component) -> None:
        self._children.append(child)

    def size(self) -> int:
        return sum(c.size() for c in self._children)

    def list_paths(self) -> list[str]:
        result = []
        for c in self._children:
            for p in c.list_paths():
                result.append(f"{self._name}/{p}")
        return result

# Usage
root = Folder("root")
root.add(File("a.txt", 100))
sub = Folder("sub")
sub.add(File("b.txt", 200))
root.add(sub)
print(root.size())        # 300
print(root.list_paths())   # ['root/a.txt', 'root/sub/b.txt']
```

### Key advantages
- Same interface for leaf and composite; client code is simple  
- New node types (e.g. symlink) fit in without changing client  
- Natural for tree operations (traverse, aggregate)

### Common mistakes / pitfalls
- **Parent references**: If children need to know parent, add carefully to avoid cycles and ownership confusion  
- **Mutable structure while iterating**: Add/remove children in a way that doesn’t break iteration (e.g. copy list or use safe iterators)  
- **Overloading “component”**: Keep the interface small and tree-oriented; don’t turn it into a generic “node” with unrelated methods

---

## 10. Chain of Responsibility Pattern

### Layman explanation
A **request** is passed along a **chain** of handlers. Each handler either handles it or passes to the next. Like support: L1 → L2 → L3 until someone resolves it.

### Problem it solves
You have multiple handlers that might process a request (e.g. auth, validation, logging, business logic). You don’t want the sender to know which handler will act; you want to add or reorder handlers without changing senders or other handlers.

### When to use
- Request processing pipelines (middleware, validation, auth)  
- Multiple handlers, any of which might “handle” the request  
- You want to add or remove steps without changing existing code  
- “First handler that can do it” semantics

### When NOT to use
- Every request must go through a fixed sequence; a pipeline or facade might be clearer  
- Only one handler ever applies; no need for a chain  
- Handlers need to communicate heavily; consider a pipeline with explicit context object instead

### Real-world (non-technical) example
**Support ticket**: L1 looks at the ticket; if they can fix it, they do. Otherwise they pass to L2. L2 same; then L3. The ticket moves down the chain until someone handles it.

### Real-time software example
**HTTP middleware**: Request goes through: CORS → auth → rate limit → body parser → route handler. Each middleware can respond (e.g. 401) or call “next.” Order is configured once; handlers don’t reference each other.

### Python code example

```python
from abc import ABC, abstractmethod
from dataclasses import dataclass

@dataclass
class Request:
    path: str
    user: str | None = None
    body: str = ""

class Handler(ABC):
    def __init__(self):
        self._next: Handler | None = None

    def set_next(self, handler: "Handler") -> "Handler":
        self._next = handler
        return handler

    def handle(self, request: Request) -> str | None:
        result = self._process(request)
        if result is not None:
            return result
        if self._next:
            return self._next.handle(request)
        return None

    @abstractmethod
    def _process(self, request: Request) -> str | None:
        pass

class AuthHandler(Handler):
    def _process(self, request: Request) -> str | None:
        if request.user is None and request.path.startswith("/admin"):
            return "401 Unauthorized"
        return None

class LoggingHandler(Handler):
    def _process(self, request: Request) -> str | None:
        print(f"Request: {request.path}")
        return None

# Usage
auth = AuthHandler()
logging = LoggingHandler()
auth.set_next(logging)

r1 = Request("/admin/settings", user=None)
print(auth.handle(r1))  # 401 Unauthorized

r2 = Request("/admin/settings", user="alice")
print(auth.handle(r2))   # None (passed to logging, no response)
```

### Key advantages
- Decouples sender from concrete handlers  
- Add/remove/reorder handlers without changing others  
- Single responsibility per handler

### Common mistakes / pitfalls
- **Forgetting to call next**: Every handler must either return a result or call the next handler, or requests “drop.”  
- **No guarantee any handler runs**: Document or enforce that at least one handler is terminal (e.g. 404 handler).  
- **Shared mutable state**: Prefer passing a request/context object; avoid global state so the chain is testable and predictable.

---

## Summary Table: All Patterns at a Glance

| Pattern | Category | Core Purpose | Real-World Analogy |
|---------|----------|--------------|--------------------|
| **Singleton** | Creational | Exactly one instance app-wide; global access point | One master key for the building |
| **Factory Method** | Creational | Create objects without specifying concrete class; centralize creation | Pizza shop: order type, get that pizza |
| **Adapter** | Structural | Make one interface look like another so two systems work together | Travel plug adapter (EU → US) |
| **Observer** | Behavioral | One subject notifies many observers when state changes | YouTube: channel notifies subscribers |
| **Decorator** | Structural | Add behavior to an object at runtime by wrapping it | Coffee + milk + sugar (same cup, extra layers) |
| **Proxy** | Structural | Stand-in that controls or optimizes access to the real object | Receptionist / security guard in front of the boss |
| **Facade** | Structural | Simple interface to a set of subsystems | Waiter: one interface to kitchen, cashier, etc. |
| **Strategy** | Behavioral | Swap algorithm/behavior at runtime; same goal, different ways | Navigator: shortest vs fastest vs avoid tolls |
| **Composite** | Structural | Treat single items and groups with the same interface (tree) | Files and folders: same ops on both |
| **Chain of Responsibility** | Behavioral | Pass request along a chain until a handler processes it | Support: L1 → L2 → L3 until resolved |

---

**Quick category recap**

- **Creational**: Singleton, Factory Method (and Abstract Factory, Builder, Prototype in full syllabi).  
- **Structural**: Adapter, Decorator, Proxy, Facade, Composite (and Bridge, Flyweight in full syllabi).  
- **Behavioral**: Observer, Strategy, Chain of Responsibility (and Command, Iterator, State, etc. in full syllabi).

Use this doc for revision and interviews: one section per pattern, with problem, when to use/not use, analogies, software examples, code, advantages, and pitfalls.
