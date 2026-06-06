**Java 8** (released in 2014) is arguably the most critical version to master for an interview. It fundamentally shifted Java from a strictly **imperative** language (telling the computer *how* to do things step-by-step) to a **declarative/functional** language (telling the computer *what* you want achieved).

If an interviewer asks you about Java 8, they are looking for deep proficiency in four pillars: **Lambdas, Functional Interfaces, Streams, and Optionals**. Let's break them down meticulously.

---

## 1. Lambda Expressions & Functional Interfaces

### The Concept

Before Java 8, if you wanted to pass a piece of logic or behavior as a parameter to a method (for example, telling a thread what code to run, or a button what to do when clicked), you had to wrap that logic inside an **Anonymous Inner Class**. This required an immense amount of boilerplate code.

Java 8 introduced **Lambda Expressions**, which are essentially lightweight, anonymous methods that can be treated as first-class citizens (passed around as variables).

For a lambda expression to exist, it needs a target type. That target must be a **Functional Interface**—an interface that contains **exactly one abstract method**. Java 8 introduced the `@FunctionalInterface` annotation to enforce this contract at compile-time.

### Code Example: Old Way vs. Java 8 Way

Let's look at creating a background thread using the `Runnable` interface (which has only one method: `run()`).

```java
public class LambdaExample {
    public static void main(String[] args) {
        
        // --- THE OLD WAY (Anonymous Inner Class) ---
        Runnable oldRunnable = new Runnable() {
            @Override
            public void run() {
                System.out.println("Thread is running the old way!");
            }
        };
        new Thread(oldRunnable).start();


        // --- THE JAVA 8 WAY (Lambda Expression) ---
        // Syntax: (parameters) -> { body }
        Runnable newRunnable = () -> System.out.println("Thread running via Lambda!");
        new Thread(newRunnable).start();
    }
}

```

### The 4 Core Functional Interfaces to Memorize

Java 8 packaged the most common programming use cases into the `java.util.function` package. You must know these four for interviews:

| Interface | Method Signature | Purpose | Real-world Analogy |
| --- | --- | --- | --- |
| **`Predicate<T>`** | `boolean test(T t)` | Takes an input, evaluates a condition, returns a boolean. | A bouncer checking IDs. |
| **`Function<T, R>`** | `R apply(T t)` | Takes an input of type T, transforms it, returns type R. | A currency converter. |
| **`Consumer<T>`** | `void accept(T t)` | Takes an input, processes it, returns nothing. | A printing machine or logger. |
| **`Supplier<T>`** | `T get()` | Takes no input, generates and returns an object of type T. | A factory or random number generator. |

---

## 2. The Streams API

### The Concept

The Streams API is **not** a data structure (it doesn't store data). Instead, it is a powerful pipeline that carries data from a source (like a `Collection` or an `Array`) through a sequence of **Intermediate Operations** (which transform or filter the data lazily) and wraps up with a **Terminal Operation** (which produces a result or side-effect).

Key characteristics to mention in an interview:

* **Lazy Evaluation:** Intermediate operations are not executed until the terminal operation is invoked.
* **Non-Destructive:** Streams do not modify the underlying data source; they create a new result.

### Production Code Example

Imagine you have a list of `Employee` objects, and your goal is to:

1. Filter out employees under the age of 30.
2. Extract their salaries.
3. Calculate the average salary of these senior employees.

```java
import java.util.*;
import java.util.stream.Collectors;

class Employee {
    private String name;
    private int age;
    private double salary;

    public Employee(String name, int age, double salary) {
        this.name = name; this.age = age; this.salary = salary;
    }
    public int getAge() { return age; }
    public double getSalary() { return salary; }
}

public class StreamDeepDive {
    public static void main(String[] args) {
        List<Employee> employees = Arrays.asList(
            new Employee("Alice", 35, 90000),
            new Employee("Bob", 24, 50000),
            new Employee("Charlie", 40, 120000),
            new Employee("David", 28, 60000)
        );

        // --- THE OLD WAY (Imperative Loop) ---
        double totalSalary = 0;
        int count = 0;
        for (Employee e : employees) {
            if (e.getAge() >= 30) {
                totalSalary += e.getSalary();
                count++;
            }
        }
        double oldAverage = count > 0 ? totalSalary / count : 0;
        System.out.println("Old Way Average: " + oldAverage);


        // --- THE JAVA 8 WAY (Streams API Pipeline) ---
        OptionalDouble averageSalary = employees.stream() // 1. Source
            .filter(emp -> emp.getAge() >= 30)           // 2. Intermediate Op: Filter (Predicate)
            .mapToDouble(Employee::getSalary)            // 3. Intermediate Op: Map to primitive double
            .average();                                  // 4. Terminal Op: Calculates average

        if (averageSalary.isPresent()) {
            System.out.println("Streams Average: " + averageSalary.getAsDouble());
        }
    }
}

```

---

## 3. The `Optional<T>` Class

### The Concept

The notorious `NullPointerException` (NPE) has plagued Java applications for decades. Typically, developers wrote defensive `if (obj != null)` blocks everywhere.

Java 8 introduced `Optional<T>`, a wrapper container object that may or may not contain a non-null value. **Crucial Interview Point:** `Optional` was not invented to replace every null check in your application. Its explicit design goal is to be used as a **method return type** to clearly tell the caller: *"This method might return a value, or it might return nothing. Handle both cases explicitly."*

### Code Example: Defensive Coding vs. Functional Optional

Let's say we have a database lookup method that might fail to find a user.

```java
import java.util.Optional;

public class OptionalExample {

    // Mock database lookup
    public static String findUserOld(int id) {
        if (id == 999) return "John Doe";
        return null; // Returning raw null forces the caller to remember to check it
    }

    // Java 8 approach using Optional
    public static Optional<String> findUserNew(int id) {
        if (id == 999) return Optional.of("John Doe");
        return Optional.empty(); // Explicitly states no value exists
    }

    public static void main(String[] args) {
        // --- THE OLD WAY ---
        String user = findUserOld(111);
        // If you forget this if-check, the next line crashes with a NullPointerException
        if (user != null) {
            System.out.println(user.toUpperCase());
        }

        // --- THE JAVA 8 WAY (Fluent Handling) ---
        Optional<String> userOpt = findUserNew(111);

        // Approach A: Fallback value using orElse()
        String finalName = userOpt.orElse("Guest User");
        System.out.println(finalName);

        // Approach B: Functional chain using ifPresent() (Accepts a Consumer)
        userOpt.ifPresent(name -> System.out.println(name.toUpperCase()));
        
        // Approach C: Throw a custom exception instantly if missing
        // String nameOrException = userOpt.orElseThrow(() -> new RuntimeException("User not found"));
    }
}

```

---

## 4. Default Methods in Interfaces

### The Problem it Solved

Before Java 8, if an interface was part of a major public library and you added a new abstract method to it, **every single class in the world that implemented that interface broke immediately** because they didn't implement the new method.

### How it Works

Java 8 allowed developers to add implementations inside interfaces using the `default` keyword. This allowed the Java core team to add the `.stream()` method to the existing `Collection` interface without breaking a single legacy application worldwide.

```java
interface Vehicle {
    void startEngine(); // Standard abstract method

    // Java 8 Default Method
    default void turnOnRadio() {
        System.out.println("Playing music... Default radio implementation.");
    }
}

class Car implements Vehicle {
    // Car is ONLY forced to implement abstract methods
    @Override
    public void startEngine() {
        System.out.println("Vroom!");
    }
    // turnOnRadio() is inherited automatically, but can be overridden if desired!
}

```

---
