# OOPs (Object-Oriented Programming) Interview Preparation Guide 🚀

This guide provides a technical and interview-oriented deep dive into Object-Oriented Programming (OOPs). It transitions from basic definitions to the architectural principles (SOLID) expected of Senior Engineers.

---

## 📖 Overview

### What is OOPs? (Simple Explanation)
**Object-Oriented Programming (OOPs)** is a programming paradigm based on the concept of "objects," which can contain data (attributes) and code (methods). 

Think of a **Blueprint vs. a House**:
- The **Class** is the blueprint (the plan).
- The **Object** is the actual house built from that plan.

### Why it is important in real-world applications
- **Modularity**: Code is organized into independent units, making large systems manageable.
- **Reusability**: Through inheritance and composition, you don't have to rewrite common logic.
- **Maintainability**: Changes in one part of the system (e.g., how a 'Payment' is processed) don't necessarily break other parts.
- **Scalability**: Easier to extend functionality (e.g., adding a new 'CryptoPayment' type) without modifying existing codebase.

---

## 🧩 Core Concepts

### The 4 Pillars of OOPs

| Pillar | Definition | Simple Analogy |
| :--- | :--- | :--- |
| **Encapsulation** | Bundling data and methods that operate on that data within a single unit (Class) and restricting direct access. | A **Capsule**: You can't see the medicine inside; you just interact with the outer shell. |
| **Abstraction** | Hiding complex implementation details and showing only the necessary features of an object. | A **Car Dashboard**: You press a button to start the engine; you don't need to know how the internal combustion works. |
| **Inheritance** | A mechanism where a new class (Subclass) acquires properties and behaviors of an existing class (Superclass). | **Family Traits**: A child inherits eye color from a parent but can also have their own unique skills. |
| **Polymorphism** | The ability of a single function or method to behave differently based on the object it is acting upon. | **A Person**: The same person acts as a 'Developer' at work, a 'Customer' at a shop, and a 'Parent' at home. |

### Important Terminology
- **Class**: A blueprint or template for creating objects.
- **Object**: An instance of a class.
- **Constructor**: A special method called when an object is instantiated (e.g., `__init__` in Python).
- **Destructor**: Method called when an object is destroyed (`__del__` in Python). Used for cleanup.
- **Interface/Abstract Class**: A contract that defines *what* a class should do, but not *how*.
- **Method Overriding**: Redefining a parent class method in a child class (Runtime Polymorphism).
- **Method Overloading**: Defining multiple methods with the same name but different signatures (Compile-time Polymorphism).

### Access Modifiers & Data Control
Control how data is accessed and modified:

| Modifier | Description | Python Convention |
| :--- | :--- | :--- |
| **Public** | Accessible from anywhere. | `name` |
| **Protected** | Accessible within class and subclasses. | `_name` |
| **Private** | Accessible only within the class. | `__name` (uses name mangling) |

**Getters & Setters**: Methods used to access (`get`) and update (`set`) private fields. They allow for **validation logic** (e.g., checking if `age > 0` before setting).

---

## ⚙️ How It Works

### Relationships & Inheritance Types

#### 1. Object Relationships (Beyond Inheritance)
- **Association**: Objects know each other but are independent (Teacher <-> Student).
- **Aggregation**: "HAS-A" relationship where child can exist without parent (Department has Teachers).
- **Composition**: Strong "HAS-A" where child dies with parent (House has Rooms).

#### 2. Types of Inheritance
- **Single**: One child inherits from one parent.
- **Multilevel**: A child inherits from a parent, which inherits from another parent (Grandparent -> Parent -> Child).
- **Hierarchical**: Multiple children inherit from one parent (Parent -> Child1, Parent -> Child2).
- **Multiple**: One child inherits from multiple parents (leads to the Diamond Problem).

### The Object Lifecycle
1.  **Declaration**: Defining the class structure.
2.  **Instantiation**: Creating an instance in memory (using the `new` keyword or class call).
3.  **Initialization**: Setting initial values via the Constructor.
4.  **Interaction**: Calling methods to perform actions or change state.
5.  **Destruction**: Memory is reclaimed (Garbage Collection).

### Memory Concepts
- **Stack**: Stores method calls, local variables, and reference pointers. It is fast and has a fixed size.
- **Heap**: Stores the actual objects and instance variables. It is larger and managed by Garbage Collection.
- **Garbage Collection (GC)**: Automatic process of identifying and deleting objects that are no longer reachable in the heap.

### Essential Keywords
- **this / self**: Refers to the current instance of the class.
- **super / base**: Refers to the parent class (used to call parent constructors or methods).
- **static**: Belongs to the class itself, not instances. Shared across all objects.
- **final / sealed**: Prevents a class from being inherited or a method from being overridden.

### Diagrams Explained (In Words)
Imagine a **Payment System Architecture**:
- **Base Class**: `Payment` (Abstract) - defines `process_payment()`.
- **Derived Classes**: `CreditCardPayment`, `PayPalPayment`, `StripePayment`.
- **Polymorphism in action**: A `PaymentService` takes a list of `Payment` objects and calls `process_payment()` on each. The service doesn't care *how* each one pays; it just knows they all *can* pay.

---

## 🌍 Real-World Use Case: E-Commerce Payment System

### Scenario
An e-commerce platform needs to support multiple payment methods.

1.  **Abstraction**: We create a `PaymentProcessor` interface. The UI doesn't need to know if it's Stripe or PayPal.
2.  **Encapsulation**: The `CreditCard` class hides sensitive data like `cvv` and `expiry` as private variables, exposing only a `validate()` method.
3.  **Inheritance**: `StripeProcessor` and `PayPalProcessor` inherit from `BaseProcessor`.
4.  **Polymorphism**: The checkout function accepts any object that implements `BaseProcessor`.

---

## 💻 Code Examples (Python)

```python
from abc import ABC, abstractmethod

# 1. Abstraction: Abstract Base Class
class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount: float):
        pass

# 2. Inheritance & Polymorphism
class StripeProcessor(PaymentProcessor):
    def process_payment(self, amount: float):
        print(f"Processing ${amount} via Stripe (Stripe API call)...")

class PayPalProcessor(PaymentProcessor):
    def process_payment(self, amount: float):
        print(f"Processing ${amount} via PayPal (Redirecting to PayPal)...")

# 3. Encapsulation
class CreditCard:
    def __init__(self, card_no, cvv):
        self.__card_no = card_no  # Private attribute (Name Mangling)
        self.__cvv = cvv          # Private attribute

    def get_masked_card(self):
        return f"****-****-****-{self.__card_no[-4:]}"

# Usage
def checkout(processor: PaymentProcessor, amount: float):
    # Polymorphism: processor can be any subclass
    processor.process_payment(amount)

stripe = StripeProcessor()
paypal = PayPalProcessor()

checkout(stripe, 100.0)
checkout(paypal, 50.0)

card = CreditCard("1234567890123456", "123")
print(card.get_masked_card()) # Output: ****-****-****-3456
```

---

## ✅ Best Practices (The Senior Level)

### 1. SOLID Principles
- **S**ingle Responsibility, **O**pen/Closed, **L**iskov Substitution, **I**nterface Segregation, **D**ependency Inversion.

### 2. Design Axioms
- **DRY (Don't Repeat Yourself)**: Avoid code duplication by using inheritance, composition, or utility functions.
- **KISS (Keep It Simple, Stupid)**: Avoid over-engineering. Favor readable, straightforward logic over clever, complex code.

### 3. Composition Over Inheritance
Inheritance creates a "is-a" relationship (tightly coupled). **Composition** creates a "has-a" relationship (loosely coupled). 
*Example*: Instead of `User` inheriting from `Database`, a `User` class should *have* a `DatabaseConnection` object.

### 4. Favor Interfaces
Always program to an interface/abstract class, not an implementation. This makes your code mockable and testable.

---

## 🛠️ Error Handling & Advanced OOP

### Exception Handling in OOP
- **Try-Catch Blocks**: Handling runtime errors gracefully to prevent crashes.
- **Custom Exceptions**: Creating domain-specific error classes (e.g., `InsufficientFundsError`) to provide clear context.
- **Checked vs. Unchecked**: Some languages require handling specific errors (Checked), others leave it to the developer (Unchecked).

### Advanced Concepts
- **Dependency Injection (DI)**: Passing dependencies (like a database client) into a class constructor rather than creating them inside.
- **Reflection**: The ability of a program to inspect and modify its own structure (classes, methods) at runtime.
- **Immutability**: Designing objects whose state cannot be changed after creation (improves thread safety and predictability).

---

## ❌ Common Mistakes
- **Tight Coupling**: Making classes too dependent on each other.
- **God Objects**: Creating a single class that does everything (Violates SRP).
- **Deep Inheritance Hierarchies**: Over-using inheritance leads to "Fragile Base Class" problems.
- **Leaking Secrets**: Not using proper encapsulation for sensitive data.

---

## ❓ Interview Questions & Answers

**Q1: What is the difference between an Abstract Class and an Interface?**
*   **A:** An **Abstract Class** can have both abstract methods (no body) and concrete methods (with body). It's used when classes share some logic. An **Interface** (in languages like Java) only defines method signatures. In Python, we use `ABC` to achieve both.

**Q2: Why is Composition often preferred over Inheritance?**
*   **A:** Inheritance creates a rigid structure that's hard to change. Composition allows you to swap behaviors at runtime by injecting different objects, leading to more flexible and maintainable code.

**Q3: Explain Method Overriding vs. Overloading.**
*   **A:** **Overriding** happens in a child class to change a parent's method. **Overloading** happens in the same class by having the same method name with different parameters (compile-time polymorphism).

**Q4: How does Encapsulation improve security?**
*   **A:** It prevents unauthorized code from modifying the internal state of an object directly. By using getters/setters or private attributes, you can add validation logic before allowing a state change.

**Q5: What is the "Diamond Problem" in Inheritance?**
*   **A:** It occurs in multiple inheritance when a class inherits from two classes that both inherit from the same superclass. Python solves this using **MRO (Method Resolution Order)**.

---

## ⚡ Quick Revision
- **Class** = Template | **Object** = Instance.
- **Encapsulation** = Private data + Public methods.
- **Abstraction** = Hide details, show "what it does".
- **Inheritance** = "is-a" relationship.
- **Polymorphism** = One interface, multiple forms.
- **SOLID** = The 5 pillars of good OOP design.
- **Composition** = "has-a" relationship (Better for flexibility).

---

## 🔗 References
- [Python Official Docs - Classes](https://docs.python.org/3/tutorial/classes.html)
- [Refactoring.Guru - OOP Principles](https://refactoring.guru/design-patterns/oop-principles)
- [SOLID Principles Explained](https://en.wikipedia.org/wiki/SOLID)
- [Clean Code by Robert C. Martin](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)
