When developers discuss standard software design patterns, they are almost always referring to the **23 classic design patterns** cataloged in the landmark 1994 book *Design Patterns: Elements of Reusable Object-Oriented Software* by the "Gang of Four" (GoF).

These 23 patterns are strictly organized into three core pillars:

---

### 1. Creational Patterns (5 Patterns)

These focus on how objects are created, ensuring the system doesn't become tightly coupled to how its components are instantiated.

* **Singleton:** Ensures a class has only one global instance.
  * More Details:
  https://github.com/hnausad/Java-Interview-Concept/blob/java-design-pattern/singleton-design-pattern.md
* **Factory Method:** Creates objects without specifying the exact class to be created.
* **Abstract Factory:** Creates families of related objects without specifying their concrete classes.
* **Builder:** Constructs complex objects step-by-step.
* Constructors become hard to read and maintain when there are many parameters, especially optional ones. The Builder Pattern makes object creation more readable, flexible, and less error-prone by allowing step-by-step construction through method chaining. It also supports immutable objects and avoids the telescoping constructor problem.
* Why use the Builder Pattern?

 **Use it when:**
 
 An object has many optional parameters.
 Object creation involves multiple steps.
 You want to create immutable objects.
 You want to avoid constructors with too many parameters (known as the telescoping constructor problem).
* **Prototype:** Creates new objects by copying/cloning an existing instance.

### 2. Structural Patterns (7 Patterns)

These deal with class and object composition—how different classes connect and interact to form larger, flexible structures.

* **Adapter:** Converts the interface of a class into another interface clients expect (acts as a translator).
* **Decorator:** Dynamically adds behavior or responsibilities to an object without changing its core structure.
* **Facade:** Provides a simplified, high-level interface to a complex underlying subsystem.
* **Proxy:** Provides a placeholder or substitute for another object to control access to it.
* **Bridge:** Separates an object’s abstraction from its implementation so both can vary independently.
* **Composite:** Composes objects into tree structures to represent part-whole hierarchies.
* **Flyweight:** Minimizes memory usage by sharing as much data as possible with similar objects.

### 3. Behavioral Patterns (11 Patterns)

These deal specifically with communication, algorithms, and assigning responsibilities between objects.

* **Observer:** A publish-subscribe mechanism where multiple objects are notified when another object changes state.
* **Strategy:** Encapsulates a family of interchangeable algorithms and selects one at runtime.
* **Command:** Turns a request or action into a standalone object, allowing you to delay, queue, or undo operations.
* **State:** Allows an object to alter its behavior when its internal state changes.
* **Iterator:** Provides a way to access elements of a collection sequentially without exposing its underlying structure.
* **Template Method:** Defines the skeleton of an algorithm in a method, deferring some exact steps to subclasses.
* **Mediator:** Restricts direct communication between objects and forces them to collaborate only through a central mediator object.
* **Chain of Responsibility:** Passes a request along a chain of potential handlers until one handles it.
* **Memento:** Captures and restores an object's internal state (useful for undo mechanisms) without breaking encapsulation.
* **Visitor:** Separates an algorithm from the object structure it operates on, allowing you to add new operations to existing structures.
* **Interpreter:** Implements an expression interface to evaluate a specific language or grammar.

---

### Beyond the Classic 23

While these 23 are the foundation for coding interviews, software engineering has evolved. If you move from object-oriented programming to **System Architecture**, you will also encounter:

1. **Architectural Patterns:** MVC (Model-View-Controller), Microservices, and Event-Driven Architecture.
2. **Enterprise Patterns:** Data Transfer Object (DTO), Repository Pattern, and Dependency Injection.
3. **Cloud Scaling Patterns:** Circuit Breaker (for handling service timeouts) and CQRS (Command Query Responsibility Segregation, separating read and write databases).
