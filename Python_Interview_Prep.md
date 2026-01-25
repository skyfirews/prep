# Python – Interview Notes

> **Target Audience**: Senior Engineers (10+ years), Technical Architects  
> **Interview Level**: Amazon, Flipkart, Razorpay (Product & Service-based companies)

---

## 1. Overview

### What is Python?
Python is a high-level, interpreted, dynamically-typed programming language with automatic memory management. It emphasizes code readability through significant whitespace and a philosophy of "batteries included."

**Non-textbook explanation**: Python trades raw performance for developer productivity. It's like a Swiss Army knife—versatile, easy to use, but not the fastest tool for every job.

### Why Companies Use Python in Real Systems
- **Rapid Development**: 3-5x faster development cycles compared to compiled languages
- **Rich Ecosystem**: Django/Flask for web, Pandas for data, TensorFlow for ML
- **Developer Productivity**: Less boilerplate = fewer bugs = faster time-to-market
- **Microservices**: Perfect for API services where I/O-bound operations dominate
- **Data Engineering**: Industry standard for ETL pipelines (Airflow, Spark)

**Real Example**: Instagram's backend is 100% Python (Django). They handle 500M+ users with Python because most operations are I/O-bound (database queries, API calls), not CPU-bound.

### Where Python is a Bad Choice
- **High-Frequency Trading**: Latency matters in microseconds. Use C++/Rust.
- **Mobile Apps**: No native mobile support. Use Swift/Kotlin.
- **Real-time Systems**: GIL prevents true parallelism. Use Go/Erlang.
- **Memory-Constrained Devices**: Python's overhead is too high. Use C.

**Production Failure**: A fintech startup tried to build their payment gateway in pure Python. During Black Friday, the GIL bottleneck caused 2-second response times. They migrated critical paths to Go.

**Senior Engineer Perspective**: Python is a tool, not a religion. Use it where it excels (I/O-bound, rapid iteration) and avoid it where it doesn't (CPU-bound, low-latency).

---

## 2. Core Concepts

### 2.1 Variables & Memory Model

| Concept | Definition | Interview Focus |
|---------|-----------|-----------------|
| **Variable** | A name bound to an object in memory | References vs values, mutability |
| **Reference** | A pointer to an object's memory address | How Python passes arguments |
| **Mutable** | Object can be changed in-place | Lists, dicts, sets |
| **Immutable** | Object cannot be changed | int, str, tuple, frozenset |

**Simple Explanation**: In Python, variables are **labels**, not boxes. `x = 5` means "label x points to the integer object 5." If you do `y = x`, both labels point to the same object.

**What Interviewers Test**: 
- "What happens when you pass a list to a function?" (Answer: Reference is passed, list can be modified)
- "Why does `a += 1` work differently for int vs list?" (Answer: `int` is immutable, creates new object; `list` is mutable, modifies in-place)

**Follow-up Trap**: "Is Python pass-by-value or pass-by-reference?"  
**Correct Answer**: Neither. It's **pass-by-object-reference**. The reference (memory address) is passed by value.

**How It Works Internally**:
1. Python creates objects on the **heap**
2. Variables store **references** (memory addresses) on the **stack**
3. `id()` returns the memory address
4. `is` checks if two variables point to the same object

```python
a = [1, 2, 3]
b = a          # b points to same object as a
b.append(4)    # Modifies the shared object
print(a)       # [1, 2, 3, 4] - both changed!

# Immutable example
x = 5
y = x          # Both point to same int object
y += 1         # Creates NEW int object (6), y points to it
print(x)       # 5 - unchanged
```

**Production Failure**: A developer passed a large list to a function, modified it, and expected the original to be unchanged. This caused a data corruption bug in production because the function modified the shared reference.

**Senior Engineer Perspective**: Always assume mutable objects are shared. Use `copy.deepcopy()` when you need independent copies. Document function side-effects clearly.

---

### 2.2 GIL (Global Interpreter Lock)

**Definition**: A mutex that allows only one thread to execute Python bytecode at a time.

**Simple Explanation**: Python's GIL is like a single bathroom key. Even if you have 4 threads (4 people), only one can use the bathroom (execute Python code) at a time. This prevents race conditions but limits true parallelism.

**What Interviewers Test**:
- "Why doesn't Python use multiple cores for CPU-bound tasks?"
- "How do you work around the GIL?"

**Follow-up Trap**: "Can you remove the GIL?"  
**Correct Answer**: Technically yes (Jython, IronPython), but CPython's GIL exists because Python's memory management isn't thread-safe. Removing it would break C extensions and slow down single-threaded code by 2-3x.

**How It Works**:
1. Thread acquires GIL → executes bytecode → releases GIL
2. GIL is released every 100 bytecode instructions (or on I/O operations)
3. Only one thread executes Python code, but I/O operations (file, network) release GIL
4. This is why Python threads work well for I/O-bound tasks but not CPU-bound

**Real-World Use Case**: A web scraper needs to fetch 1000 URLs. Using threads (with GIL) is fine because threads release GIL during `requests.get()`. But for image processing (CPU-bound), use `multiprocessing` instead.

**Code Example**:
```python
# BAD: CPU-bound with threads (GIL bottleneck)
import threading

def cpu_task():
    total = 0
    for i in range(10000000):
        total += i

threads = [threading.Thread(target=cpu_task) for _ in range(4)]
# All threads compete for GIL - no speedup!

# GOOD: CPU-bound with multiprocessing (bypasses GIL)
from multiprocessing import Process

processes = [Process(target=cpu_task) for _ in range(4)]
# Each process has its own GIL - true parallelism!
```

**Production Failure**: A data science team used `threading` for parallel NumPy computations. Performance didn't improve because NumPy releases GIL, but their custom Python loops didn't. They switched to `multiprocessing` and got 4x speedup.

**Senior Engineer Perspective**: GIL is a feature, not a bug. Use threads for I/O-bound, multiprocessing for CPU-bound, asyncio for high-concurrency I/O.

---

### 2.3 Memory Management & Garbage Collection

**Definition**: Python automatically manages memory through reference counting and a cyclic garbage collector.

**Simple Explanation**: Python keeps track of how many variables point to each object. When count reaches zero, memory is freed immediately. For circular references (A → B → A), a garbage collector runs periodically.

**What Interviewers Test**:
- "How does Python's garbage collection work?"
- "What's the difference between `__del__` and garbage collection?"

**Follow-up Trap**: "When is an object garbage collected?"  
**Correct Answer**: When reference count = 0 (immediate), or when GC detects it's unreachable in a cycle (periodic).

**How It Works**:
1. **Reference Counting**: Each object has a counter. When a variable references it, count++. When variable goes out of scope, count--. At 0, object is deleted.
2. **Cyclic GC**: For circular references, Python uses a generational GC (3 generations). Objects that survive GC move to older generation.
3. **Memory Layout**: Objects stored on heap, variables (references) on stack.

**Real-World Use Case**: A caching service stores user sessions in memory. Without proper cleanup, circular references between Session → User → Session cause memory leaks. Using `weakref` breaks the cycle.

**Code Example**:
```python
import sys
import gc

# Reference counting
a = [1, 2, 3]
print(sys.getrefcount(a))  # 2 (a + temporary in getrefcount)

b = a
print(sys.getrefcount(a))  # 3 (a, b, temporary)

del b
print(sys.getrefcount(a))  # 2 (back to a + temporary)

# Circular reference (needs GC)
class Node:
    def __init__(self, value):
        self.value = value
        self.next = None

a = Node(1)
b = Node(2)
a.next = b
b.next = a  # Circular reference!

del a, b  # Reference count never reaches 0
gc.collect()  # GC breaks the cycle
```

**Production Failure**: A long-running service had a memory leak. Objects weren't being freed because of circular references between models. The service crashed after 3 days. Solution: Used `weakref` and explicit `del` statements.

**Senior Engineer Perspective**: Don't rely on GC for cleanup. Use context managers (`with` statements) for resources. Profile memory with `memory_profiler` in production.

---

### 2.4 Decorators

**Definition**: Functions that modify or extend the behavior of other functions without changing their code.

**Simple Explanation**: Decorators are like gift wrappers. You wrap a function (the gift) with a decorator (the wrapper) to add functionality (logging, caching, auth) without modifying the original function.

**What Interviewers Test**:
- "Write a decorator that measures execution time"
- "How do decorators work under the hood?"

**Follow-up Trap**: "What's the difference between `@decorator` and `@decorator()`?"  
**Correct Answer**: `@decorator` applies the decorator function directly. `@decorator()` calls the decorator function first (which returns another function), then applies that.

**How It Works**:
1. Decorator is a **higher-order function** (takes function, returns function)
2. `@decorator` is syntactic sugar for `func = decorator(func)`
3. Decorator receives the original function, wraps it, returns the wrapper
4. When decorated function is called, wrapper executes, then original function

**Real-World Use Case**: An API service needs to log all function calls, measure latency, and handle retries. Decorators handle cross-cutting concerns without cluttering business logic.

**Code Example**:
```python
import time
from functools import wraps

# Simple decorator
def timer(func):
    @wraps(func)  # Preserves function metadata
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.time() - start:.2f}s")
        return result
    return wrapper

# Decorator with arguments
def retry(max_attempts=3):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_attempts - 1:
                        raise
                    print(f"Attempt {attempt + 1} failed: {e}")
            return None
        return wrapper
    return decorator

@timer
@retry(max_attempts=3)
def fetch_user_data(user_id):
    # Business logic
    return {"id": user_id, "name": "Alice"}

# Usage
fetch_user_data(123)
```

**Production Failure**: A developer forgot `@wraps(func)` in a decorator. Stack traces showed `wrapper` instead of the actual function name, making debugging impossible in production.

**Senior Engineer Perspective**: Always use `@wraps` to preserve metadata. Use `functools.lru_cache` for memoization. Keep decorators simple—complex logic belongs in the function.

---

### 2.5 Generators & Iterators

**Definition**: Generators are functions that yield values lazily, creating iterator objects. Iterators are objects that implement `__iter__` and `__next__`.

**Simple Explanation**: A generator is like a lazy list. Instead of creating all values upfront (memory-intensive), it produces values on-demand when you iterate. Think of it as a factory that makes items one at a time, not all at once.

**What Interviewers Test**:
- "What's the difference between a generator and a list?"
- "Write a generator that yields Fibonacci numbers"

**Follow-up Trap**: "Can you iterate over a generator twice?"  
**Correct Answer**: No. Generators are **one-shot iterators**. Once exhausted, they're empty. Convert to list if you need to iterate multiple times.

**How It Works**:
1. Generator function contains `yield` instead of `return`
2. Calling generator function returns a **generator object** (doesn't execute yet)
3. Each `next()` call executes until `yield`, returns value, pauses
4. State is preserved between calls (local variables, execution position)
5. When function ends, `StopIteration` is raised automatically

**Real-World Use Case**: Processing a 10GB log file. Loading entire file into memory would crash the server. Generator reads line-by-line, processing one chunk at a time.

**Code Example**:
```python
# Generator function
def fibonacci():
    a, b = 0, 1
    while True:
        yield a
        a, b = b, a + b

# Usage
gen = fibonacci()
print(next(gen))  # 0
print(next(gen))  # 1
print(next(gen))  # 1
print(next(gen))  # 2

# Generator expression (list comprehension for generators)
squares = (x**2 for x in range(10))  # Generator, not list
print(list(squares))  # [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

# Processing large file
def read_large_file(filename):
    with open(filename) as f:
        for line in f:  # File object is already an iterator
            yield line.strip()

# Memory efficient - only one line in memory at a time
for line in read_large_file("huge.log"):
    process(line)
```

**Production Failure**: A developer converted a generator to a list to iterate twice. The list had 10M elements and consumed 800MB RAM, causing OOM errors. Solution: Use `itertools.tee()` to create multiple iterators from one generator.

**Senior Engineer Perspective**: Generators are Python's superpower for memory efficiency. Use them for large datasets, infinite sequences, and pipeline processing. Prefer generator expressions over list comprehensions when you don't need the full list.

---

### 2.6 Context Managers (`with` statement)

**Definition**: Objects that manage resources (files, locks, connections) with automatic setup and teardown.

**Simple Explanation**: Context managers ensure resources are properly cleaned up, even if an error occurs. Like a smart door that automatically locks when you leave, even if you forget.

**What Interviewers Test**:
- "How does the `with` statement work?"
- "Write a custom context manager"

**Follow-up Trap**: "What happens if an exception occurs inside `with`?"  
**Correct Answer**: The `__exit__` method is still called. If `__exit__` returns `True`, the exception is suppressed; if `False` or `None`, exception propagates.

**How It Works**:
1. `with` statement calls `__enter__()` on context manager
2. `__enter__()` returns a value (usually `self` or a resource)
3. Code block executes
4. `__exit__()` is **always** called, even on exceptions
5. `__exit__()` receives exception info (type, value, traceback)

**Real-World Use Case**: Database connections must be closed even if a query fails. Context managers ensure cleanup happens automatically, preventing connection leaks.

**Code Example**:
```python
# Built-in context manager (file)
with open("data.txt", "r") as f:
    content = f.read()
# File automatically closed, even if exception occurs

# Custom context manager (class-based)
class DatabaseConnection:
    def __enter__(self):
        self.conn = connect_to_db()
        return self.conn
    
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.conn.close()
        return False  # Don't suppress exceptions

# Usage
with DatabaseConnection() as db:
    db.execute("SELECT * FROM users")

# Context manager (function-based with @contextmanager)
from contextlib import contextmanager

@contextmanager
def timer():
    start = time.time()
    yield
    print(f"Elapsed: {time.time() - start:.2f}s")

with timer():
    expensive_operation()
```

**Production Failure**: A service had 1000 open database connections because developers forgot to close them. The database server ran out of connections and crashed. Migrating to context managers fixed the leak.

**Senior Engineer Perspective**: Always use context managers for resources. They're exception-safe and make code cleaner. Use `contextlib` utilities (`@contextmanager`, `closing`, `suppress`) for common patterns.

---

### 2.7 Metaclasses

**Definition**: Classes that create classes. The class of a class.

**Simple Explanation**: If a class is a blueprint for objects, a metaclass is a blueprint for classes. It controls how classes are created, allowing you to modify class creation behavior.

**What Interviewers Test**:
- "What is a metaclass? When would you use it?"
- "What's the relationship between `type` and metaclasses?"

**Follow-up Trap**: "Can you give a practical example?"  
**Correct Answer**: Django's ORM uses metaclasses to transform model definitions into database queries. SQLAlchemy uses them for declarative models.

**How It Works**:
1. When Python sees `class MyClass:`, it calls the metaclass's `__new__()` and `__init__()`
2. Default metaclass is `type`
3. You can customize by setting `__metaclass__` or inheriting from a class with a custom metaclass
4. Metaclass can modify class attributes, add methods, register classes

**Real-World Use Case**: A framework needs to auto-register all API endpoints. Using a metaclass, every class that inherits from `APIEndpoint` is automatically registered in a router.

**Code Example**:
```python
# Metaclass that auto-registers classes
class RegistryMeta(type):
    registry = {}
    
    def __new__(cls, name, bases, attrs):
        new_class = super().__new__(cls, name, bases, attrs)
        if name != "BaseAPI":
            cls.registry[name] = new_class
        return new_class

class BaseAPI(metaclass=RegistryMeta):
    pass

class UserAPI(BaseAPI):
    def get(self):
        return "users"

class ProductAPI(BaseAPI):
    def get(self):
        return "products"

print(RegistryMeta.registry)
# {'UserAPI': <class '__main__.UserAPI'>, 'ProductAPI': <class '__main__.ProductAPI'>}
```

**Production Failure**: A developer used metaclasses to auto-generate database models. The metaclass logic was too complex and caused 30-second import times. They refactored to use decorators instead.

**Senior Engineer Perspective**: Metaclasses are powerful but obscure. Use them sparingly—only when you need to modify class creation. Prefer decorators or class decorators for most use cases. "If you don't know if you need a metaclass, you don't need one."

---

### 2.8 Descriptors

**Definition**: Objects that define how attribute access works (`__get__`, `__set__`, `__delete__`).

**Simple Explanation**: Descriptors let you customize what happens when you access an attribute. They're how `property()` works under the hood.

**What Interviewers Test**:
- "How does `@property` work?"
- "What's the difference between a descriptor and a property?"

**Follow-up Trap**: "When would you use a descriptor vs a property?"  
**Correct Answer**: Use descriptors when you need the same behavior across multiple attributes. Use `@property` for single-attribute customization.

**How It Works**:
1. Descriptor protocol: `__get__()`, `__set__()`, `__delete__()`
2. When you access `obj.attr`, Python checks if `attr` is a descriptor
3. If descriptor, calls `__get__()` or `__set__()` instead of normal attribute access
4. `property()` is a descriptor that calls getter/setter functions

**Real-World Use Case**: A model needs type validation for multiple fields. Instead of writing getters/setters for each, a descriptor validates all attributes automatically.

**Code Example**:
```python
# Descriptor for type validation
class TypedProperty:
    def __init__(self, expected_type):
        self.expected_type = expected_type
        self.name = None
    
    def __set_name__(self, owner, name):
        self.name = name
    
    def __get__(self, obj, objtype=None):
        return obj.__dict__.get(self.name)
    
    def __set__(self, obj, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"{self.name} must be {self.expected_type.__name__}")
        obj.__dict__[self.name] = value

class User:
    name = TypedProperty(str)
    age = TypedProperty(int)
    
    def __init__(self, name, age):
        self.name = name
        self.age = age

user = User("Alice", 30)
user.age = "thirty"  # TypeError: age must be int
```

**Production Failure**: A developer used descriptors for validation but forgot to handle `None` values. This caused `None` to bypass validation and corrupt the database.

**Senior Engineer Perspective**: Descriptors are the foundation of Python's attribute system. Use them for reusable validation, lazy evaluation, or computed attributes. But prefer `@property` for simple cases.

---

## 3. How It Works (Internals)

### 3.1 Python Execution Model

**Step-by-Step Flow**:
1. **Source Code** → Python parser converts to **Abstract Syntax Tree (AST)**
2. **AST** → Compiler converts to **bytecode** (`.pyc` files)
3. **Bytecode** → Python Virtual Machine (PVM) interprets bytecode
4. **PVM** → Executes opcodes, manages stack, calls C functions for built-ins

**Memory Model**:
- **Stack**: Stores function calls, local variables (references to objects)
- **Heap**: Stores actual objects (lists, dicts, user-defined objects)
- **Namespaces**: Dict-like structures mapping names to objects

**Reference Counting**:
- Each object has a `refcount` field
- When variable references object, `refcount++`
- When variable goes out of scope, `refcount--`
- At `refcount = 0`, object is immediately deallocated (unless in cycle)

**Garbage Collection**:
- **Generation 0**: New objects
- **Generation 1**: Survived one GC
- **Generation 2**: Survived multiple GCs
- GC runs when generation 0 threshold reached
- Objects in older generations checked less frequently

---

## 4. Real-World Use Case: E-Commerce API Service

**System**: Payment processing microservice handling 10,000 requests/second

**Python Concepts Used**:

1. **Async/Await (asyncio)**: I/O-bound operations (database, external APIs) use async to handle concurrency without threads
2. **Decorators**: `@rate_limit`, `@cache`, `@authenticate` wrap endpoints
3. **Context Managers**: Database connections, Redis connections use `with` statements
4. **Generators**: Streaming large order lists without loading into memory
5. **Descriptors**: Type validation for payment amounts, currency codes
6. **Exception Handling**: Custom exceptions (`PaymentFailedError`, `InsufficientFundsError`)

**Why Python Over Alternatives**:
- **Rapid Development**: New payment methods added in days, not weeks
- **Rich Ecosystem**: Stripe SDK, Redis client, async frameworks (FastAPI)
- **I/O-Bound**: Most time spent waiting for database/API responses, not CPU computation
- **Team Velocity**: Python's readability means faster onboarding

**Trade-offs**:
- **Latency**: 50ms average (acceptable for payments, not for HFT)
- **Scaling**: Horizontal scaling (multiple instances) compensates for GIL
- **Memory**: Higher than Go/Rust, but acceptable with proper garbage collection tuning

---

## 5. Code Examples

### 5.1 Proper Exception Handling

```python
# GOOD: Specific exceptions, proper cleanup
class PaymentError(Exception):
    pass

class InsufficientFundsError(PaymentError):
    pass

def process_payment(user_id, amount):
    try:
        balance = get_balance(user_id)
        if balance < amount:
            raise InsufficientFundsError(f"Balance {balance} < {amount}")
        
        # Critical section - must succeed or rollback
        deduct_balance(user_id, amount)
        record_transaction(user_id, amount)
        return {"status": "success"}
    
    except InsufficientFundsError:
        # Re-raise with context
        raise
    except DatabaseError as e:
        # Log and wrap
        logger.error(f"Database error: {e}")
        raise PaymentError("Payment processing failed") from e
    except Exception as e:
        # Catch-all with logging
        logger.exception("Unexpected error")
        raise PaymentError("Unknown error") from e
    finally:
        # Always cleanup
        close_db_connection()
```

**What Would Break**: Catching `Exception` without re-raising loses the original exception context. Missing `finally` could leak database connections.

---

### 5.2 Async/Await Pattern

```python
import asyncio
import aiohttp

async def fetch_user_data(user_id):
    async with aiohttp.ClientSession() as session:
        async with session.get(f"https://api.example.com/users/{user_id}") as response:
            return await response.json()

async def process_users(user_ids):
    tasks = [fetch_user_data(uid) for uid in user_ids]
    return await asyncio.gather(*tasks)

# Usage
user_ids = [1, 2, 3, 4, 5]
results = asyncio.run(process_users(user_ids))
```

**What Would Break**: Using `requests` (synchronous) instead of `aiohttp` would block the event loop, defeating the purpose of async.

---

## 6. Best Practices

### Do's ✅

1. **Use Type Hints**: `def process(user_id: int) -> dict:` improves IDE support and catches bugs
2. **Follow PEP 8**: Consistent style improves readability
3. **Use Virtual Environments**: `venv` or `poetry` for dependency isolation
4. **Handle Exceptions Explicitly**: Don't use bare `except:`
5. **Use Context Managers**: Always use `with` for resources
6. **Profile Before Optimizing**: Use `cProfile` to find bottlenecks
7. **Use `__slots__`**: For classes with many instances to save memory
8. **Leverage Built-ins**: `collections.defaultdict`, `itertools`, `functools`

### Don'ts ❌

1. **Don't Modify Lists While Iterating**: Creates unpredictable behavior
2. **Don't Use Mutable Default Arguments**: `def func(items=[])` is a trap
3. **Don't Ignore Exceptions**: At least log them
4. **Don't Use `==` for `None`**: Use `is None` (identity check)
5. **Don't Import `*`**: Pollutes namespace, makes code unclear
6. **Don't Use `eval()` or `exec()`**: Security risk
7. **Don't Rely on CPython Implementation Details**: Code should work on PyPy, Jython

### Performance Tips

- **Use Generators**: For large datasets, generators save memory
- **Use `collections.deque`**: O(1) append/pop from both ends vs O(n) for list
- **Use Sets for Membership Tests**: O(1) vs O(n) for lists
- **Use `functools.lru_cache`**: Memoization for expensive computations
- **Profile with `cProfile`**: `python -m cProfile script.py`

### Security Implications

- **SQL Injection**: Use parameterized queries, never string formatting
- **Code Injection**: Never use `eval()` with user input
- **Dependency Vulnerabilities**: Use `safety` or `pip-audit` to check packages
- **Secrets Management**: Never hardcode API keys, use environment variables or secret managers

### Scalability Concerns

- **GIL Limits CPU Parallelism**: Use `multiprocessing` for CPU-bound tasks
- **Memory Overhead**: Python objects have overhead (~24-56 bytes per object)
- **Global State**: Avoid module-level variables in long-running processes
- **Connection Pooling**: Reuse database/HTTP connections, don't create new ones per request

---

## 7. Common Mistakes

### 7.1 Mutable Default Arguments

```python
# BAD
def add_item(item, items=[]):
    items.append(item)
    return items

print(add_item(1))  # [1]
print(add_item(2))  # [1, 2] - WRONG! Default list is shared

# GOOD
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

**Why Interviewers Reject This**: Shows lack of understanding of Python's object model. This bug is subtle and causes production issues.

**Correct Mental Model**: Default arguments are evaluated once at function definition time, not each call. Mutable defaults are shared across all calls.

---

### 7.2 Modifying List While Iterating

```python
# BAD
items = [1, 2, 3, 4, 5]
for item in items:
    if item % 2 == 0:
        items.remove(item)  # Modifies list during iteration - unpredictable!

# GOOD
items = [x for x in items if x % 2 != 0]  # List comprehension
# OR
items = [1, 2, 3, 4, 5]
items = [x for x in items if x % 2 != 0]
```

**Why Interviewers Reject This**: Shows lack of understanding of iteration protocol. This can skip elements or raise `RuntimeError`.

---

### 7.3 Using `==` Instead of `is` for Singletons

```python
# BAD
if user == None:  # Calls __eq__, can be overridden
    pass

# GOOD
if user is None:  # Identity check, always correct
    pass

# Also applies to: True, False, Ellipsis, NotImplemented
```

**Why Interviewers Reject This**: `==` can be overridden and may not work as expected. `is` checks object identity, which is what you want for singletons.

---

### 7.4 Not Using Context Managers

```python
# BAD
file = open("data.txt")
content = file.read()
file.close()  # What if exception occurs before this?

# GOOD
with open("data.txt") as file:
    content = file.read()
# Automatically closed, even on exception
```

**Why Interviewers Reject This**: Resource leaks are a serious production issue. Context managers are the Pythonic way to handle resources.

---

## 8. Interview Questions & Answers

### A. Basic (Warm-up)

**Q1: What is the difference between `list` and `tuple`?**

**Short Answer**: Lists are mutable (can be modified), tuples are immutable. Lists use `[]`, tuples use `()`.

**Follow-up Trap**: "When would you use a tuple over a list?"  
**Ideal Answer**: Use tuples for fixed collections (coordinates, database records), dictionary keys (must be hashable), or when you want to prevent accidental modification. Lists are for dynamic collections that need to grow/shrink.

---

**Q2: Explain the difference between `==` and `is`.**

**Short Answer**: `==` checks value equality (calls `__eq__`), `is` checks object identity (same memory address).

**Follow-up Trap**: "When would `==` and `is` give different results?"  
**Ideal Answer**: For mutable objects, two different objects can have the same value (`==` True) but different identities (`is` False). For immutable objects like small integers, Python caches them, so `is` might be True even for different assignments.

```python
a = [1, 2, 3]
b = [1, 2, 3]
print(a == b)  # True (same values)
print(a is b)  # False (different objects)

x = 256
y = 256
print(x is y)  # True (small integers are cached)
```

---

**Q3: What is a lambda function?**

**Short Answer**: An anonymous function defined with `lambda` keyword. Syntax: `lambda args: expression`.

**Follow-up Trap**: "When should you use lambda vs a regular function?"  
**Ideal Answer**: Use lambda for short, one-line functions passed as arguments (e.g., `sorted(items, key=lambda x: x.age)`). Use regular functions for anything more complex or reusable.

---

### B. Intermediate (Most Common)

**Q4: How does Python's garbage collection work?**

**Short Answer**: Python uses reference counting (immediate cleanup) plus a generational garbage collector (for circular references). Objects are deleted when reference count reaches 0, or when GC detects they're unreachable.

**Follow-up Trap**: "What's the difference between `del` and garbage collection?"  
**Ideal Answer**: `del` removes a name binding (decrements reference count). If count reaches 0, object is immediately deallocated. GC only runs for circular references that reference counting can't handle.

---

**Q5: Explain the Global Interpreter Lock (GIL).**

**Short Answer**: The GIL is a mutex that allows only one thread to execute Python bytecode at a time. It prevents true parallelism for CPU-bound tasks but doesn't affect I/O-bound operations.

**Follow-up Trap**: "How do you work around the GIL?"  
**Ideal Answer**: For CPU-bound tasks, use `multiprocessing` (separate processes, each with its own GIL). For I/O-bound tasks, use `threading` (GIL is released during I/O) or `asyncio` (single-threaded, event-driven).

---

**Q6: What are decorators? How do they work?**

**Short Answer**: Decorators are functions that modify other functions. `@decorator` is syntactic sugar for `func = decorator(func)`. Decorators receive the original function, wrap it, and return the wrapper.

**Follow-up Trap**: "Write a decorator that caches function results."  
**Ideal Answer**:

```python
from functools import wraps

def cache(func):
    cache_dict = {}
    @wraps(func)
    def wrapper(*args, **kwargs):
        key = str(args) + str(kwargs)
        if key not in cache_dict:
            cache_dict[key] = func(*args, **kwargs)
        return cache_dict[key]
    return wrapper

# Better: use functools.lru_cache
from functools import lru_cache

@lru_cache(maxsize=128)
def expensive_function(n):
    # ...
    pass
```

---

**Q7: What is the difference between `__str__` and `__repr__`?**

**Short Answer**: `__str__` is for user-friendly string representation (used by `print()`), `__repr__` is for developer-friendly representation (used by `repr()`, should be unambiguous and ideally recreate the object).

**Follow-up Trap**: "When would they return different values?"  
**Ideal Answer**: `__str__` can be more readable (`"User(Alice, 30)"`), while `__repr__` should be precise (`"User(name='Alice', age=30)"`). If `__str__` is not defined, `__repr__` is used as fallback.

---

### C. Advanced / Tricky (Senior-level)

**Q8: Explain Python's Method Resolution Order (MRO).**

**Short Answer**: MRO determines the order in which Python searches for methods in inheritance hierarchies. Python uses C3 linearization algorithm to create a consistent, predictable order.

**Follow-up Trap**: "What happens in diamond inheritance?"  
**Ideal Answer**: C3 linearization ensures each class appears only once in MRO and respects the order in base classes. Example:

```python
class A: pass
class B(A): pass
class C(A): pass
class D(B, C): pass

print(D.__mro__)
# (<class 'D'>, <class 'B'>, <class 'C'>, <class 'A'>, <class 'object'>)
```

---

**Q9: What are metaclasses? Give a practical use case.**

**Short Answer**: Metaclasses are classes that create classes. They control class creation behavior. Practical use: Django ORM models, SQLAlchemy declarative models, API framework registration.

**Follow-up Trap**: "When would you use a metaclass vs a class decorator?"  
**Ideal Answer**: Use metaclasses when you need to modify class creation itself (add methods, change inheritance). Use class decorators when you just need to wrap or modify an existing class. Metaclasses are more powerful but harder to understand.

---

**Q10: How does `async/await` work under the hood?**

**Short Answer**: `async` functions return coroutine objects. `await` suspends execution and yields control to the event loop. The event loop schedules other coroutines and resumes the awaited coroutine when its result is ready.

**Follow-up Trap**: "What's the difference between `asyncio.create_task()` and `await`?"  
**Ideal Answer**: `await` waits for the coroutine to complete. `create_task()` schedules the coroutine to run concurrently and returns a Task object you can await later. Use `create_task()` when you want to run multiple coroutines concurrently.

```python
# Sequential (slow)
result1 = await fetch_data1()
result2 = await fetch_data2()  # Waits for result1

# Concurrent (fast)
task1 = asyncio.create_task(fetch_data1())
task2 = asyncio.create_task(fetch_data2())
result1 = await task1
result2 = await task2  # Both run concurrently
```

---

## 9. Quick Revision

- **Variables are references**, not boxes. `a = b` means both point to the same object.
- **GIL prevents true parallelism** for CPU-bound tasks. Use `multiprocessing` for CPU-bound, `threading`/`asyncio` for I/O-bound.
- **Reference counting** deallocates objects immediately when count = 0. **GC** handles circular references.
- **Decorators** are functions that modify functions. `@decorator` = `func = decorator(func)`.
- **Generators** yield values lazily, saving memory. They're one-shot iterators.
- **Context managers** (`with` statement) ensure cleanup via `__enter__`/`__exit__`.
- **Metaclasses** create classes. Use sparingly—only when you need to modify class creation.
- **Descriptors** control attribute access (`__get__`, `__set__`). `@property` is a descriptor.
- **MRO** (Method Resolution Order) uses C3 linearization for inheritance hierarchies.
- **`==` checks value**, `is` checks identity. Use `is` for `None`, `True`, `False`.
- **Mutable default arguments** are shared across calls. Use `None` as default.
- **`async/await`** uses an event loop** to handle concurrency without threads.

---

## 10. References

### Official Documentation
- [Python Official Docs](https://docs.python.org/3/)
- [Python Language Reference](https://docs.python.org/3/reference/)
- [PEP 8 - Style Guide](https://pep8.org/)
- [Python Data Model](https://docs.python.org/3/reference/datamodel.html)

### Trusted Engineering Blogs
- [Real Python](https://realpython.com/) - In-depth Python tutorials
- [Ned Batchelder's Blog](https://nedbatchelder.com/blog/) - Python internals explained
- [Python's GIL Explained](https://realpython.com/python-gil/)
- [Fluent Python by Luciano Ramalho](https://www.oreilly.com/library/view/fluent-python/9781491946237/) - Advanced Python concepts

---

## Production Failures & Senior Engineer Perspectives

### How Concepts Fail in Production

**GIL Bottleneck**: A data processing service used threads for parallel computation. Performance didn't improve because CPU-bound Python code can't utilize multiple cores. **Solution**: Migrated CPU-intensive parts to `multiprocessing` or C extensions.

**Memory Leaks from Circular References**: Long-running service accumulated memory because objects had circular references. Reference counting couldn't free them. **Solution**: Used `weakref` to break cycles, added explicit `del` statements, tuned GC thresholds.

**Decorator Metadata Loss**: Stack traces showed `wrapper` instead of actual function names because `@wraps` was missing. **Solution**: Always use `@wraps(func)` in decorators.

**Mutable Default Arguments**: A function with `items=[]` default accumulated data across calls, causing data corruption. **Solution**: Use `None` as default, create new list inside function.

**Generator Exhaustion**: Code tried to iterate over a generator twice, getting empty results the second time. **Solution**: Convert to list if multiple iterations needed, or use `itertools.tee()`.

### How Senior Engineers Think

**Trade-offs Over Dogma**: Python isn't always the answer. Use it for I/O-bound, rapid development, data processing. Avoid it for low-latency, memory-constrained, CPU-intensive systems.

**Profile First**: Don't optimize blindly. Use `cProfile`, `memory_profiler`, `line_profiler` to find actual bottlenecks. Most "slow Python" code is actually slow algorithms or I/O waits.

**Readability > Cleverness**: Metaclasses, descriptors, and advanced features are powerful but obscure. Prefer simple, explicit code. Use advanced features only when they solve a real problem.

**Ecosystem Matters**: Python's strength is its libraries. Don't reinvent the wheel. Use `requests` for HTTP, `SQLAlchemy` for databases, `pandas` for data. But understand what these libraries do under the hood.

**Production-Ready Code**: Always handle exceptions, use context managers, log properly, type hint for maintainability. Code that works on your machine isn't production-ready.

---

**Last Updated**: January 2025  
**Target Companies**: Amazon, Flipkart, Razorpay, Product & Service-based companies  
**Interview Level**: Senior Engineer (10+ years), Technical Architect
