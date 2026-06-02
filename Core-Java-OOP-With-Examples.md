Here is the complete breakdown of every single question, paired with practical, real-world Java examples to make them stick.

---

## Part 1: Java Core & OOP Pillars

### 21. What are the four pillars of OOP?

* **Encapsulation:** Hiding internal data.
```java
public class BankAccount {
    private double balance; // Hidden data

    public void deposit(double amount) { // Controlled access
        if (amount > 0) balance += amount;
    }
}

```


* **Abstraction:** Hiding complexity.
```java
// You know it brakes, but you don't care how the hydraulic fluid moves
public interface BrakeSystem {
    void applyBrake();
}

```


* **Inheritance:** Reusing code.
```java
public class Vehicle { int speed; }
public class Car extends Vehicle { int numberOfDoors; } // Inherits speed

```


* **Polymorphism:** Many forms.
```java
Animal myAnimal = new Dog();
myAnimal.makeSound(); // Executes Dog's bark, not generic animal sound

```



---

### 22. Difference between abstraction and encapsulation?

* **Abstraction (What it does):** Think of a **TV Remote**. You only see the buttons (Power, Volume). The complex circuitry inside is abstracted away.
* **Encapsulation (How it secures it):** Think of a **Medical Capsule**. The inner medicine layout is wrapped and protected by the outer shell so it isn't contaminated or altered.

```java
// ABSTRACTION: Focuses on interface design
public interface PaymentGateway {
    void processPayment(double amount); 
}

// ENCAPSULATION: Focuses on data security
public class CreditCardPayment implements PaymentGateway {
    private String cardNumber; // Private to protect sensitive data

    public void processPayment(double amount) {
        // Implementation details hidden inside the class
    }
}

```

---

### 23. Difference between composition and inheritance?

* **Inheritance ("Is-A"):** A `Tesla` **is a** `Car`. If the parent class changes, the child class can break accidentally.
* **Composition ("Has-A"):** A `Car` **has an** `Engine`. It is highly flexible because you can swap components easily.

```java
// INHERITANCE (Rigid)
public class Engine {}
public class CombustionEngine extends Engine {} 

// COMPOSITION (Flexible)
public class Car {
    private Engine engine; // Car HAS AN engine

    public Car(Engine engine) {
        this.engine = engine; // Can pass a V8Engine, ElectricEngine, etc. dynamically
    }
}

```

---

### 24. What is polymorphism?

* **Compile-time (Overloading):** The compiler looks at the method arguments to determine which method to execute.
```java
public class Calculator {
    int add(int a, int b) { return a + b; }
    double add(double a, double b) { return a + b; } // Overloaded
}

```


* **Runtime (Overriding):** The JVM looks at the actual object type *at execution time*.
```java
public class Animal { void sound() { System.out.println("Animal sound"); } }
public class Cat extends Animal { void sound() { System.out.println("Meow"); } }

// Usage:
Animal generic = new Cat();
generic.sound(); // Output: "Meow" (Resolved at runtime)

```



---

## Part 2: Advanced Java Concepts & Patterns

### 25. What are immutable classes?

An object whose state cannot be altered after creation. `String` and wrapper classes like `Integer` are great examples.

```java
// 1. final class prevents inheritance
public final class ImmutablePoint {
    // 2. private final fields
    private final int x;
    private final int y;

    public ImmutablePoint(int x, int y) {
        this.x = x;
        this.y = y;
    }

    // 3. Only getters, no setters
    public int getX() { return x; }
    public int getY() { return y; }
}

```

---

### 26. Why is String immutable in Java?

Imagine a **String Pool** in Java memory where two variables point to the same text literal to save space:

```java
String s1 = "Password123";
String s2 = "Password123"; // Both point to the same memory slot

```

If Strings were mutable, and you changed `s1` to `"Admin123"`, `s2` would automatically change to `"Admin123"` as well! This would create massive security vulnerabilities and thread-safety nightmares across your application.

---

### 27. Difference between checked and unchecked exceptions?

* **Checked Exception (Compile-time catch):** Outside forces your code cannot control (e.g., a file being missing). The compiler forces you to write a fallback.
```java
// Will NOT compile unless wrapped in try-catch or declared with 'throws'
FileReader file = new FileReader("C:\\test.txt"); 

```


* **Unchecked Exception (Runtime flaw):** Programmatic errors or bad logic.
```java
// Compiles perfectly, but crashes with NullPointerException at runtime
String text = null;
System.out.println(text.length()); 

```



---

### 28. Difference between final, finally, and finalize?

* **`final` (Modifier):** "This is the end of modifications."
```java
final int SPEED_LIMIT = 120; // Cannot change value

```


* **`finally` (Block):** "No matter what happens, execute this cleanup."
```java
try {
    db.connect();
} catch(Exception e) {
    log.error(e);
} finally {
    db.close(); // Always runs, even if a crash happens above
}

```


* **`finalize()` (Method):** A deprecated method called by the Garbage Collector right before an object is permanently destroyed. *Avoid using it.*

---

### 29. What are marker interfaces?

An empty interface used as a "stamp" or label to pass instructions to the JVM.

```java
// Notice there are no methods inside
public interface Serializable {} 

// Usage:
public class UserProfile implements Serializable {
    // Java now knows this class is safe to convert into binary streams/files
}

```

---

### 30. Difference between method overloading and overriding?

* **Overloading:** Same method name, completely different inputs, same class.
* **Overriding:** Redefining a parent's method inside a child class to change behavior.

```java
public class Processor {
    // OVERLOADING
    void process(String text) {}
    void process(int number) {} 
}

public class AdvancedProcessor extends Processor {
    // OVERRIDING
    @Override
    void process(String text) {
        System.out.println("Processing with special upgrades!");
    }
}

```

---

### 31. What is dependency injection?

Instead of a class hardcoding its own internal tools, the tools are handed ("injected") to it by an external framework.

```java
// WITHOUT DI (Hardcoded dependency - difficult to test)
public class Car {
    private Engine engine = new V8Engine(); 
}

// WITH DI (Flexible - easy to pass mock engines for testing)
public class Car {
    private Engine engine;

    public Car(Engine engine) { // Engine is injected here
        this.engine = engine;
    }
}

```

---

### 32. Explain SOLID principles with examples.

* **S (Single Responsibility):** A class should do one job.
* *Bad:* `Invoice` class calculates tax and prints PDFs.
* *Good:* `Invoice` calculates tax; `InvoicePrinter` prints PDFs.


* **O (Open/Closed):** Add features by adding new code, not modifying old code.
* *Example:* If you introduce a new `CryptoPayment`, create a new class implementing a `Payment` interface instead of writing another `if/else` inside your existing `PaymentProcessor` class.


* **L (Liskov Substitution):** Subclasses shouldn't break parent expectations.
* *Bad:* A `Ostrich` class inherits from `Bird`, but throws an `UnsupportedOperationException` when the `fly()` method is called.


* **I (Interface Segregation):** Don't force thick interfaces on clients.
* *Good:* Split a large `SmartDevice` interface into smaller `Printer` and `Scanner` interfaces so a basic printer doesn't have to implement empty `scan()` methods.


* **D (Dependency Inversion):** Depend on abstractions, not concrete items.
* *Good:* A `Store` class should depend on a general `PaymentGateway` interface, not a concrete `PayPalService` class.



---

### 33. What design patterns are commonly used in backend systems?

* **Builder Pattern:** Avoids messy constructors with 10 arguments.
```java
User user = new UserBuilder().setName("Alice").setAge(25).setCountry("India").build();

```


* **Factory Pattern:** Dynamically provisions object creation logic.
```java
Notification notify = NotificationFactory.getNotificationChannel("SMS"); // Returns SMSNotification object

```


* **Strategy Pattern:** Dynamically changes business algorithms.
```java
checkout.setPaymentStrategy(new UPIPayment());
checkout.process();

```



---

### 34. What is singleton pattern and its drawbacks?

Ensures a class has only one single active instance globally.

```java
public class DatabaseConnection {
    private static DatabaseConnection instance = new DatabaseConnection();
    private DatabaseConnection() {} // Private constructor prevents 'new' keyword usage
    public static DatabaseConnection getInstance() { return instance; }
}

```

* **Drawback Example:** It creates global state. If you run 5 automated testing suites at the same time, they will all share the same singleton connection instance simultaneously. If Test A changes a setting on it, Test B will randomly fail because its state was modified under the hood.

---

### 35. Difference between deep copy and shallow copy?

Assume you copy a `Person` object who holds an internal `Address` object reference:

* **Shallow Copy:** Copies the `Person` box but references the *exact same* physical `Address` object in memory. If you change the street name in the copied person, the original person's street address changes too.
* **Deep Copy:** Completely duplicates both the `Person` box *and creates a brand-new distinct `Address` object* in memory. They are 100% disconnected.

---

Which of these architectural patterns or principles would you like to drill down into further with concrete code refactoring scenarios?
