Here is a comprehensive, interview-ready guide to answering these core Java and OOP questions.

---

## Part 1: Java Core & OOP Pillars

### 21. What are the four pillars of OOP?

The four pillars are the foundation of Object-Oriented Programming:

* **Encapsulation:** Wrapping data (variables) and code (methods) together as a single unit and restricting direct access to internal states (using `private` modifiers and getters/setters).
* **Abstraction:** Hiding complex implementation details and showing only the essential features of an object (using `abstract` classes and `interfaces`).
* **Inheritance:** The mechanism by which one class acquires the properties and behaviors of a parent class (`extends`), promoting code reusability.
* **Polymorphism:** The ability of an object to take on many forms—allowing a single action to behave differently based on the object invoking it.

### 22. Difference between abstraction and encapsulation?

| Feature | Abstraction | Encapsulation |
| --- | --- | --- |
| **Core Focus** | Hiding **complexity** (Focuses on *what* the object does). | Hiding **data** and internal states (Focuses on *how* to secure it). |
| **Implementation** | Achieved using **Abstract classes** and **Interfaces**. | Achieved using **access modifiers** (`private`, `protected`) and getters/setters. |
| **Design Level** | Happens at the **design phase** of an application. | Happens at the **implementation phase**. |

### 23. Difference between composition and inheritance?

* **Inheritance ("Is-A" relationship):** A class derives from a parent class (e.g., `Car extends Vehicle`). It creates **tight coupling** because changes in the parent class directly impact the child class.
* **Composition ("Has-A" relationship):** A class holds a reference to an instance of another class (e.g., `Car has an Engine`). It offers **loose coupling** and greater flexibility because you can change the component's behavior at runtime.

> **Design Tip:** Always prefer *composition over inheritance* to maintain flexible, modular code.

### 24. What is polymorphism?

Polymorphism allows methods to execute different behaviors based on the context. Java supports two types:

1. **Compile-time (Static) Polymorphism:** Achieved via **Method Overloading** (same method name, different parameter lists within the same class). The compiler decides which method to call.
2. **Runtime (Dynamic) Polymorphism:** Achieved via **Method Overriding** (subclass provides a specific implementation of a method declared in its parent class). The JVM resolves the call at runtime based on the actual object instance.

---

## Part 2: Advanced Java Concepts & Patterns

### 25. What are immutable classes?

An immutable class is a class whose state (object data) **cannot be modified** after it is created.

* **How to create one in Java:**
1. Declare the class as `final` (prevents subclassing).
2. Make all fields `private` and `final`.
3. Provide no setter methods.
4. Perform a **deep copy** for any mutable fields during initialization and when returning them from getters.



### 26. Why is String immutable in Java?

* **String Pool Economy:** Java caches string literals in a special memory area. If strings were mutable, changing one reference would silently corrupt all other references pointing to that same literal.
* **Security:** Strings are widely used for file paths, network URLs, and database credentials. If mutable, a hacker could alter the string value *after* security checks pass.
* **Thread Safety:** Because their state cannot change, strings are inherently thread-safe and can be shared across multiple threads without synchronization.
* **HashCode Caching:** The hashcode of a `String` is calculated and cached when created. This makes it incredibly fast when used as a key in collections like `HashMap`.

### 27. Difference between checked and unchecked exceptions?

* **Checked Exceptions:** Exceptions that are checked at compile-time (e.g., `IOException`, `SQLException`). The compiler forces you to either handle them with a `try-catch` block or declare them in the method signature using `throws`.
* **Unchecked Exceptions:** Exceptions that inherit from `RuntimeException` (e.g., `NullPointerException`, `ArrayIndexOutOfBoundsException`). They occur at runtime, usually due to logical flaws, and do not need to be explicitly declared or caught.

### 28. Difference between final, finally, and finalize?

* **`final` (Keyword):** Used to apply restrictions. A `final` variable cannot be reassigned; a `final` method cannot be overridden; a `final` class cannot be inherited.
* **`finally` (Block):** A control block used with `try-catch` to execute cleanup code (like closing database connections). It **always executes**, whether an exception is thrown or caught.
* **`finalize()` (Method):** A method protected by the `Object` class that the Garbage Collector calls just before destroying an object. *(Note: This is completely deprecated in modern Java).*

### 29. What are marker interfaces?

A marker interface is an interface with **absolutely no methods or fields**. It serves as a tag to signal to the JVM or compiler that an object possesses a specific capability or behavior.

* **Examples:** `Serializable` (allows object serialization) and `Cloneable` (allows the use of the `clone()` method).

### 30. Difference between method overloading and overriding?

| Feature | Method Overloading | Method Overriding |
| --- | --- | --- |
| **Scope** | Happens within the **same class**. | Happens between a **Parent and Child class**. |
| **Signature** | Must have a **different** parameter list. | Must have the **exact same** parameter list. |
| **Binding** | Resolved at **compile-time**. | Resolved at **runtime**. |

### 31. What is dependency injection?

Dependency Injection (DI) is a design pattern used to achieve loose coupling. Instead of a class manually instantiating its own dependencies using the `new` keyword, the dependencies are **passed (injected) into it** by an external framework or container (like Spring). It can be done via constructors, setters, or fields.

### 32. Explain SOLID principles with examples.

* **S - Single Responsibility:** A class should have one, and only one, reason to change. *(Example: Separate a `User` entity from a `UserEmailSender` class).*
* **O - Open/Closed:** Software entities should be open for extension but closed for modification. *(Example: Using interfaces for payment strategies so you can add `CryptoPayment` without modifying existing credit card logic).*
* **L - Liskov Substitution:** Derived classes must be completely substitutable for their base classes without breaking the app. *(Example: If a `Ostrich` class extends `Bird` but throws an error on `fly()`, it breaks this principle. You should split them into `FlyingBird` and `NonFlyingBird`).*
* **I - Interface Segregation:** Clients shouldn't be forced to depend on methods they don't use. *(Example: Instead of one massive `Worker` interface with `code()` and `cook()`, split them into `Developer` and `Chef` interfaces).*
* **D - Dependency Inversion:** Depend on abstractions (interfaces), not concrete implementations. *(Example: A `Car` should depend on an `Engine` interface, not a concrete `V8Engine` class).*

### 33. What design patterns are commonly used in backend systems?

* **Creational:** **Singleton** (for managing shared resources like connection pools), **Factory** (for generating object variants), and **Builder** (for creating objects with complex parameters).
* **Structural:** **Adapter** (translating incompatible interfaces), **Proxy** (lazy loading or security checks), and **Facade** (simplifying complex subsystem interfaces).
* **Behavioral:** **Observer** (event-driven messaging systems), **Strategy** (swapping business algorithms dynamically), and **Template Method** (defining a workflow outline with customizable steps).

### 34. What is singleton pattern and its drawbacks?

The Singleton pattern ensures that a class has **only one instance** globally within the JVM and provides a single point of access to it.

* **Drawbacks:**
* **Hard to Unit Test:** It introduces global state, making it difficult to isolate tests and mock dependencies.
* **Violates Single Responsibility:** The class manages its own lifecycle alongside its actual business logic.
* **Hidden Dependencies:** Since it's globally accessible, developers often call it deep inside code components without passing it explicitly, obfuscating architectural coupling.



### 35. Difference between deep copy and shallow copy?

* **Shallow Copy:** Copies the top-level object wrapper, but any nested object references inside it still point to the **same original memory addresses**. Modifying a nested object inside the copy impacts the original.
* **Deep Copy:** Copies the top-level object *and recursively duplicates every nested object* inside it. The new object is entirely independent of the original.
