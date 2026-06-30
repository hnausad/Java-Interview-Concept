# Java Design Patterns — Complete Reference

A detailed guide to all major design patterns used in Java, with explanations, real-life analogies, use cases, Spring Framework usage, and code examples.

---

## CREATIONAL PATTERNS

### 1. Singleton

**Explain:** Ensures a class has only one instance and provides a global access point to it. The class controls its own instantiation, usually by making the constructor private and exposing a static method to get the instance.

**Real-life example:** A country has only one government, one president's office, or one central bank — there's exactly one instance that everyone refers to.

**Use case:** Logging frameworks, configuration managers, connection pools, caches — anything where having multiple instances would cause inconsistency or waste resources.

**Where it's used in Spring:** Spring beans are singleton-scoped by default. When you declare a `@Bean` or `@Component`, the Spring container creates exactly one instance per container (unless you change the scope to `prototype`, `request`, etc.).

**Code example:**
```java
public class ConfigManager {
    private static volatile ConfigManager instance;
    private final Map<String, String> settings = new HashMap<>();

    private ConfigManager() {
        settings.put("env", "production");
    }

    public static ConfigManager getInstance() {
        if (instance == null) {
            synchronized (ConfigManager.class) {
                if (instance == null) {
                    instance = new ConfigManager();
                }
            }
        }
        return instance;
    }

    public String get(String key) {
        return settings.get(key);
    }
}

// Better: enum-based singleton (thread-safe, serialization-safe by default)
public enum ConfigManagerEnum {
    INSTANCE;
    private final Map<String, String> settings = new HashMap<>();
    public String get(String key) { return settings.get(key); }
}
```

---

### 2. Factory Method

**Explain:** Defines an interface for creating an object but lets subclasses decide which class to instantiate. The creation logic is delegated to a method rather than calling `new` directly.

**Real-life example:** A pizza store has a `createPizza()` method; depending on which franchise (NYPizzaStore, ChicagoPizzaStore), a different style of pizza is produced, but the ordering process is the same.

**Use case:** When the exact type of object needed isn't known until runtime, or when you want to decouple object creation from usage code.

**Where it's used in Spring:** `BeanFactory` and `ApplicationContext` themselves are factory implementations — they decide which bean instance to return based on configuration. `FactoryBean<T>` interface lets you write custom factory logic for complex bean creation.

**Code example:**
```java
interface Notification {
    void notifyUser();
}

class EmailNotification implements Notification {
    public void notifyUser() { System.out.println("Sending Email"); }
}

class SmsNotification implements Notification {
    public void notifyUser() { System.out.println("Sending SMS"); }
}

abstract class NotificationFactory {
    abstract Notification createNotification();
}

class EmailNotificationFactory extends NotificationFactory {
    Notification createNotification() { return new EmailNotification(); }
}

class SmsNotificationFactory extends NotificationFactory {
    Notification createNotification() { return new SmsNotification(); }
}

// Usage
NotificationFactory factory = new EmailNotificationFactory();
Notification n = factory.createNotification();
n.notifyUser();
```

---

### 3. Abstract Factory

**Explain:** Provides an interface for creating families of related objects without specifying their concrete classes. It's a "factory of factories."

**Real-life example:** A furniture manufacturer producing matching sets — a Victorian-style factory makes Victorian chairs, sofas, and tables together; a Modern-style factory makes the modern equivalents, ensuring the pieces are stylistically consistent.

**Use case:** Cross-platform UI toolkits (Windows vs Mac widget families), or database-driver families where you need a consistent set of related connection/statement/result objects.

**Where it's used in Spring:** `BeanFactory` hierarchy combined with `ApplicationContext` acts as an abstract factory for the whole object graph. Spring's JDBC abstraction layer (`DataSource` and related connection objects across different DB vendors) follows this idea conceptually.

**Code example:**
```java
interface Button { void render(); }
interface Checkbox { void render(); }

class WindowsButton implements Button { public void render() { System.out.println("Windows Button"); } }
class WindowsCheckbox implements Checkbox { public void render() { System.out.println("Windows Checkbox"); } }
class MacButton implements Button { public void render() { System.out.println("Mac Button"); } }
class MacCheckbox implements Checkbox { public void render() { System.out.println("Mac Checkbox"); } }

interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

class WindowsFactory implements GUIFactory {
    public Button createButton() { return new WindowsButton(); }
    public Checkbox createCheckbox() { return new WindowsCheckbox(); }
}

class MacFactory implements GUIFactory {
    public Button createButton() { return new MacButton(); }
    public Checkbox createCheckbox() { return new MacCheckbox(); }
}
```

---

### 4. Builder

**Explain:** Separates the construction of a complex object from its representation, allowing step-by-step construction and producing different configurations of the same object type.

**Real-life example:** Ordering a custom sandwich at Subway — you choose bread, protein, toppings, sauce step by step, and the final sandwich is assembled based on your choices.

**Use case:** Objects with many optional parameters (avoiding telescoping constructors), immutable objects, or objects requiring complex multi-step assembly (e.g., building an HTTP request).

**Where it's used in Spring:** `UriComponentsBuilder` for building URLs, `MockMvcRequestBuilders` in Spring Test, `BeanDefinitionBuilder` for programmatically constructing bean definitions.

**Code example:**
```java
public class User {
    private final String name;
    private final int age;
    private final String email;

    private User(Builder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.email = builder.email;
    }

    public static class Builder {
        private String name;
        private int age;
        private String email;

        public Builder name(String name) { this.name = name; return this; }
        public Builder age(int age) { this.age = age; return this; }
        public Builder email(String email) { this.email = email; return this; }
        public User build() { return new User(this); }
    }
}

// Usage
User user = new User.Builder()
        .name("Asha")
        .age(28)
        .email("asha@example.com")
        .build();
```

---

### 5. Prototype

**Explain:** Creates new objects by copying (cloning) an existing object, rather than instantiating a class from scratch — useful when object creation is expensive.

**Real-life example:** Photocopying a signed document instead of writing a new one from scratch — you get an identical copy quickly without redoing all the original work.

**Use case:** When object creation is costly (e.g., involves a database call or heavy computation) and you need many similar objects, such as duplicating a complex game character or a pre-configured report template.

**Where it's used in Spring:** Bean scope `prototype` itself signals this pattern — Spring creates a new bean instance every time it's requested, instead of reusing a singleton. Also, Spring's `Object.clone()`-based utilities and `BeanUtils.copyProperties()` reflect prototype-like copying.

**Code example:**
```java
public class Report implements Cloneable {
    private String title;
    private List<String> sections;

    public Report(String title, List<String> sections) {
        this.title = title;
        this.sections = new ArrayList<>(sections);
    }

    @Override
    public Report clone() {
        return new Report(this.title, this.sections);
    }
}

// Usage
Report base = new Report("Q1 Report", List.of("Summary", "Finance"));
Report copy = base.clone();
```

---

## STRUCTURAL PATTERNS

### 6. Adapter

**Explain:** Converts the interface of a class into another interface that clients expect, allowing incompatible interfaces to work together.

**Real-life example:** A travel power plug adapter lets a device with a US plug work in a European socket — it doesn't change the device or the socket, just bridges the mismatch.

**Use case:** Integrating a legacy class or third-party library whose interface doesn't match what your code expects, without modifying the original source.

**Where it's used in Spring:** `HandlerAdapter` in Spring MVC adapts different types of controllers (annotated, `Controller` interface-based, etc.) to a uniform dispatch mechanism used by `DispatcherServlet`.

**Code example:**
```java
interface MediaPlayer {
    void play(String fileName);
}

class LegacyMp4Player {
    void playMp4(String fileName) { System.out.println("Playing mp4: " + fileName); }
}

class MediaAdapter implements MediaPlayer {
    private final LegacyMp4Player legacyPlayer = new LegacyMp4Player();

    public void play(String fileName) {
        legacyPlayer.playMp4(fileName);
    }
}
```

---

### 7. Bridge

**Explain:** Decouples an abstraction from its implementation so the two can vary independently. Instead of a deep inheritance hierarchy, the abstraction holds a reference to an implementation interface.

**Real-life example:** A TV remote (abstraction) works with different TV brands (implementations) — the remote's buttons stay the same while the underlying TV logic varies.

**Use case:** When you have multiple dimensions of variation, e.g., different shapes that can be rendered with different rendering engines, avoiding a combinatorial explosion of subclasses.

**Where it's used in Spring:** Spring's `PlatformTransactionManager` abstraction is a bridge between transaction-handling code and different underlying implementations (JDBC, JPA, JTA), allowing the application logic to stay the same regardless of which transaction technology is used.

**Code example:**
```java
interface Renderer {
    void renderCircle(float radius);
}

class VectorRenderer implements Renderer {
    public void renderCircle(float radius) { System.out.println("Drawing circle as vectors, r=" + radius); }
}

class RasterRenderer implements Renderer {
    public void renderCircle(float radius) { System.out.println("Drawing circle as pixels, r=" + radius); }
}

abstract class Shape {
    protected Renderer renderer;
    Shape(Renderer renderer) { this.renderer = renderer; }
    abstract void draw();
}

class Circle extends Shape {
    private float radius;
    Circle(Renderer renderer, float radius) { super(renderer); this.radius = radius; }
    void draw() { renderer.renderCircle(radius); }
}
```

---

### 8. Composite

**Explain:** Composes objects into tree structures to represent part-whole hierarchies, letting clients treat individual objects and compositions of objects uniformly.

**Real-life example:** A company's organizational chart — an employee and a manager (who has employees under them) can both be treated as "organizational units" with a uniform interface.

**Use case:** File system structures (files and folders), UI component trees (panels containing buttons and other panels), or menu structures with nested submenus.

**Where it's used in Spring:** Spring Security's `CompositeFilter` chains multiple filters as one. Also `CompositeCacheManager` composes multiple `CacheManager` instances into a single one.

**Code example:**
```java
interface FileSystemItem {
    void showDetails();
}

class File implements FileSystemItem {
    private String name;
    File(String name) { this.name = name; }
    public void showDetails() { System.out.println("File: " + name); }
}

class Folder implements FileSystemItem {
    private String name;
    private List<FileSystemItem> items = new ArrayList<>();
    Folder(String name) { this.name = name; }
    void add(FileSystemItem item) { items.add(item); }
    public void showDetails() {
        System.out.println("Folder: " + name);
        for (FileSystemItem item : items) item.showDetails();
    }
}
```

---

### 9. Decorator

**Explain:** Attaches additional responsibilities to an object dynamically, wrapping it without altering its structure — an alternative to subclassing for extending behavior.

**Real-life example:** Adding toppings to a coffee — each topping (milk, sugar, whipped cream) wraps the base coffee and adds its own cost and description without changing the coffee class itself.

**Use case:** Java I/O streams are the classic example — wrapping a `FileInputStream` with `BufferedInputStream`, then with `DataInputStream`, to layer functionality.

**Where it's used in Spring:** `TransactionAwareDataSourceProxy` wraps a `DataSource` to add transaction awareness. Spring Security's filter chain wraps requests with additional security checks. `HttpServletRequestWrapper` (Servlet API, used heavily by Spring) also follows this pattern.

**Code example:**
```java
interface Coffee {
    double cost();
    String description();
}

class SimpleCoffee implements Coffee {
    public double cost() { return 2.0; }
    public String description() { return "Coffee"; }
}

abstract class CoffeeDecorator implements Coffee {
    protected Coffee decoratedCoffee;
    CoffeeDecorator(Coffee coffee) { this.decoratedCoffee = coffee; }
}

class MilkDecorator extends CoffeeDecorator {
    MilkDecorator(Coffee coffee) { super(coffee); }
    public double cost() { return decoratedCoffee.cost() + 0.5; }
    public String description() { return decoratedCoffee.description() + ", Milk"; }
}

// Usage
Coffee order = new MilkDecorator(new SimpleCoffee());
```

---

### 10. Facade

**Explain:** Provides a simplified, unified interface to a complex subsystem of classes, hiding internal complexity from the client.

**Real-life example:** A car's ignition button — pressing it triggers fuel injection, spark plugs, the starter motor, and more, but the driver only interacts with one simple button.

**Use case:** Simplifying interaction with a complex library or set of subsystems, such as a `OrderFacade` that internally coordinates inventory, payment, and shipping services.

**Where it's used in Spring:** `JdbcTemplate` is a facade over the verbose raw JDBC API (connections, statements, result sets, exception handling). `RestTemplate`/`WebClient` similarly facade complex HTTP client logic.

**Code example:**
```java
class InventoryService { void checkStock(String item) { System.out.println("Checking stock for " + item); } }
class PaymentService { void charge(double amount) { System.out.println("Charging $" + amount); } }
class ShippingService { void ship(String item) { System.out.println("Shipping " + item); } }

class OrderFacade {
    private InventoryService inventory = new InventoryService();
    private PaymentService payment = new PaymentService();
    private ShippingService shipping = new ShippingService();

    void placeOrder(String item, double amount) {
        inventory.checkStock(item);
        payment.charge(amount);
        shipping.ship(item);
    }
}
```

---

### 11. Flyweight

**Explain:** Minimizes memory usage by sharing as much data as possible between similar objects, separating intrinsic (shared) state from extrinsic (unique) state.

**Real-life example:** A library has one physical copy of "shared" reference text formatting/fonts used across thousands of printed pages — only the unique page content differs, while font glyph data is reused.

**Use case:** Rendering large numbers of similar objects efficiently, such as characters in a text editor, trees in a forest simulation, or particle systems in games.

**Where it's used in Spring:** Spring's bean container itself acts as a flyweight factory for singleton beans — shared instances are reused across the application rather than recreated. `Integer.valueOf()` caching in Java (not Spring-specific, but conceptually identical) is another classic example.

**Code example:**
```java
class TreeType {
    private String name;
    private String color;
    TreeType(String name, String color) { this.name = name; this.color = color; }
    void draw(int x, int y) { System.out.println("Drawing " + name + " at (" + x + "," + y + ")"); }
}

class TreeFactory {
    private static final Map<String, TreeType> types = new HashMap<>();
    static TreeType getTreeType(String name, String color) {
        String key = name + color;
        return types.computeIfAbsent(key, k -> new TreeType(name, color));
    }
}
```

---

### 12. Proxy

**Explain:** Provides a surrogate or placeholder for another object to control access to it — used for lazy loading, access control, logging, or remote access.

**Real-life example:** A credit card is a proxy for your bank account — it controls and tracks access to your funds without you handling cash directly each time.

**Use case:** Lazy-loading expensive resources, adding security checks before accessing an object, or creating remote proxies for distributed objects.

**Where it's used in Spring:** This is central to Spring — `@Transactional`, `@Async`, `@Cacheable`, and AOP in general work by generating dynamic proxies (JDK dynamic proxies or CGLIB) around your beans to inject cross-cutting behavior before/after method calls.

**Code example:**
```java
interface Image {
    void display();
}

class RealImage implements Image {
    private String filename;
    RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }
    private void loadFromDisk() { System.out.println("Loading " + filename); }
    public void display() { System.out.println("Displaying " + filename); }
}

class ProxyImage implements Image {
    private RealImage realImage;
    private String filename;
    ProxyImage(String filename) { this.filename = filename; }
    public void display() {
        if (realImage == null) realImage = new RealImage(filename); // lazy load
        realImage.display();
    }
}
```

---

## BEHAVIORAL PATTERNS

### 13. Chain of Responsibility

**Explain:** Passes a request along a chain of handlers; each handler decides either to process the request or pass it to the next handler in the chain.

**Real-life example:** Tech support escalation — a basic query is handled by Tier 1 support, but if it can't be resolved, it's escalated to Tier 2, then Tier 3.

**Use case:** Request processing pipelines like middleware, validation chains, or approval workflows (e.g., expense approval requiring multiple levels of sign-off based on amount).

**Where it's used in Spring:** The Servlet `FilterChain` (used extensively by Spring Security) is a textbook implementation — each filter processes the request and passes it along. Spring's `HandlerInterceptor` chain works similarly.

**Code example:**
```java
abstract class SupportHandler {
    protected SupportHandler next;
    void setNext(SupportHandler next) { this.next = next; }
    abstract void handle(int severity);
}

class Tier1Handler extends SupportHandler {
    void handle(int severity) {
        if (severity <= 1) System.out.println("Tier 1 resolved it");
        else if (next != null) next.handle(severity);
    }
}

class Tier2Handler extends SupportHandler {
    void handle(int severity) {
        if (severity <= 2) System.out.println("Tier 2 resolved it");
        else if (next != null) next.handle(severity);
    }
}
```

---

### 14. Command

**Explain:** Encapsulates a request as an object, allowing parameterization of clients with queues, requests, and operations, and supporting undoable actions.

**Real-life example:** A restaurant order slip — a waiter writes down a customer's order (the command), hands it to the kitchen, and the kitchen executes it independently of the waiter.

**Use case:** Implementing undo/redo functionality, task queues, GUI button actions, or job scheduling systems where actions need to be queued or logged.

**Where it's used in Spring:** Spring's `JdbcTemplate` uses command-like callback objects (`StatementCallback`, `PreparedStatementCallback`). Also, `Runnable`/`Callable` tasks submitted to Spring's `TaskExecutor` follow the command pattern.

**Code example:**
```java
interface Command {
    void execute();
}

class Light {
    void turnOn() { System.out.println("Light ON"); }
    void turnOff() { System.out.println("Light OFF"); }
}

class TurnOnCommand implements Command {
    private Light light;
    TurnOnCommand(Light light) { this.light = light; }
    public void execute() { light.turnOn(); }
}

class RemoteControl {
    private Command command;
    void setCommand(Command command) { this.command = command; }
    void pressButton() { command.execute(); }
}
```

---

### 15. Interpreter

**Explain:** Defines a representation for a language's grammar and an interpreter that uses the representation to evaluate sentences in that language.

**Real-life example:** A calculator app parsing and evaluating the expression "3 + 4 * 2" by interpreting each symbol according to grammar rules.

**Use case:** Building simple scripting/query languages, parsing rule engines, or evaluating SQL-like or regex-like expressions.

**Where it's used in Spring:** Spring Expression Language (SpEL), used in annotations like `@Value("#{someBean.someProperty}")` and `@PreAuthorize`, is a direct implementation of the Interpreter pattern, parsing and evaluating expressions at runtime.

**Code example:**
```java
interface Expression {
    int interpret();
}

class NumberExpression implements Expression {
    private int number;
    NumberExpression(int number) { this.number = number; }
    public int interpret() { return number; }
}

class AddExpression implements Expression {
    private Expression left, right;
    AddExpression(Expression left, Expression right) { this.left = left; this.right = right; }
    public int interpret() { return left.interpret() + right.interpret(); }
}

// Usage: interprets "3 + 4"
Expression expr = new AddExpression(new NumberExpression(3), new NumberExpression(4));
System.out.println(expr.interpret()); // 7
```

---

### 16. Iterator

**Explain:** Provides a way to access elements of a collection sequentially without exposing its underlying representation.

**Real-life example:** A TV remote's "channel up/down" buttons let you move through channels one at a time without needing to know how channels are internally stored or numbered.

**Use case:** Traversing custom data structures (trees, graphs, linked lists) in a uniform way, regardless of the internal structure.

**Where it's used in Spring:** Spring heavily uses Java's built-in `Iterable`/`Iterator` interfaces throughout its collection-handling code, e.g., iterating over registered beans via `ApplicationContext.getBeanDefinitionNames()` or iterating `BeanPostProcessor` lists during context startup.

**Code example:**
```java
class BookCollection implements Iterable<String> {
    private List<String> books = new ArrayList<>();
    void addBook(String book) { books.add(book); }
    public Iterator<String> iterator() { return books.iterator(); }
}

// Usage
BookCollection collection = new BookCollection();
collection.addBook("Effective Java");
for (String book : collection) System.out.println(book);
```

---

### 17. Mediator

**Explain:** Defines an object that encapsulates how a set of objects interact, promoting loose coupling by preventing objects from referring to each other directly.

**Real-life example:** An air traffic control tower — planes don't communicate directly with each other; they all communicate through the tower, which coordinates safe takeoffs and landings.

**Use case:** Chat room applications (users send messages through a central mediator), or complex UI dialogs where many components need to react to each other's state changes.

**Where it's used in Spring:** Spring's `ApplicationEventPublisher` and the event system (`ApplicationEvent`/`@EventListener`) act as a mediator — beans publish events without knowing which other beans are listening, and the `ApplicationContext` coordinates the dispatch.

**Code example:**
```java
interface ChatMediator {
    void sendMessage(String message, User user);
}

class ChatRoom implements ChatMediator {
    public void sendMessage(String message, User sender) {
        System.out.println(sender.getName() + " sends: " + message);
    }
}

class User {
    private String name;
    private ChatMediator mediator;
    User(String name, ChatMediator mediator) { this.name = name; this.mediator = mediator; }
    String getName() { return name; }
    void send(String message) { mediator.sendMessage(message, this); }
}
```

---

### 18. Memento

**Explain:** Captures and externalizes an object's internal state without violating encapsulation, so the object can be restored to that state later.

**Real-life example:** A video game's "save point" — you save your progress, keep playing, and can reload that exact saved state if you fail later.

**Use case:** Undo mechanisms in text editors, transaction rollback features, or checkpoint/restore systems in long-running processes.

**Where it's used in Spring:** Spring's `@Transactional` rollback mechanism conceptually mirrors memento — it captures a "savepoint" of database state before risky operations so the system can revert on failure. Spring Session's snapshotting of session state for persistence also reflects this idea.

**Code example:**
```java
class EditorMemento {
    private final String content;
    EditorMemento(String content) { this.content = content; }
    String getContent() { return content; }
}

class Editor {
    private String content = "";
    void type(String words) { content += words; }
    EditorMemento save() { return new EditorMemento(content); }
    void restore(EditorMemento memento) { content = memento.getContent(); }
    String getContent() { return content; }
}
```

---

### 19. Observer

**Explain:** Defines a one-to-many dependency between objects so that when one object (the subject) changes state, all its dependents (observers) are notified automatically.

**Real-life example:** A YouTube channel's subscribers — when the channel uploads a new video, all subscribers automatically get notified without the channel having to check on each one individually.

**Use case:** Event-driven systems, GUI listeners, pub-sub messaging, or real-time data feeds like stock price updates notifying multiple display widgets.

**Where it's used in Spring:** Spring's event publishing model — `ApplicationEvent`, `ApplicationListener<T>`, and `@EventListener` — is a direct implementation of Observer. Beans publish events via `ApplicationEventPublisher`, and listener beans are notified automatically.

**Code example:**
```java
interface Observer {
    void update(String event);
}

class EmailSubscriber implements Observer {
    public void update(String event) { System.out.println("Email notified: " + event); }
}

class EventPublisher {
    private List<Observer> observers = new ArrayList<>();
    void subscribe(Observer o) { observers.add(o); }
    void publish(String event) {
        for (Observer o : observers) o.update(event);
    }
}
```

---

### 20. State

**Explain:** Allows an object to alter its behavior when its internal state changes, appearing as if the object changed its class.

**Real-life example:** A traffic light — its behavior (which light is on) changes based on its current state (red, yellow, green), and each state determines what happens next.

**Use case:** Order processing workflows (pending, shipped, delivered, cancelled), document approval workflows, or media player state (playing, paused, stopped).

**Where it's used in Spring:** While not a core Spring framework feature, Spring State Machine (a related Spring project) is built explicitly around this pattern, modeling state transitions for workflows like order processing or approval pipelines.

**Code example:**
```java
interface OrderState {
    void next(OrderContext context);
}

class PendingState implements OrderState {
    public void next(OrderContext context) {
        System.out.println("Order shipped");
        context.setState(new ShippedState());
    }
}

class ShippedState implements OrderState {
    public void next(OrderContext context) {
        System.out.println("Order delivered");
        context.setState(new DeliveredState());
    }
}

class DeliveredState implements OrderState {
    public void next(OrderContext context) { System.out.println("Already delivered"); }
}

class OrderContext {
    private OrderState state = new PendingState();
    void setState(OrderState state) { this.state = state; }
    void next() { state.next(this); }
}
```

---

### 21. Strategy

**Explain:** Defines a family of interchangeable algorithms, encapsulates each one, and lets the algorithm vary independently of the clients that use it.

**Real-life example:** Choosing a payment method at checkout — credit card, PayPal, or wallet — the checkout process stays the same, but the payment algorithm used differs based on selection.

**Use case:** Sorting algorithms selectable at runtime, different validation strategies, or multiple pricing/discount calculation strategies in an e-commerce system.

**Where it's used in Spring:** Spring Security's `AuthenticationProvider` implementations are interchangeable strategies for authenticating users. Also, `Resource` loading strategies (`ClassPathResource`, `FileSystemResource`, `UrlResource`) follow this pattern, and `PlatformTransactionManager` implementations are strategies for transaction handling.

**Code example:**
```java
interface PaymentStrategy {
    void pay(double amount);
}

class CreditCardPayment implements PaymentStrategy {
    public void pay(double amount) { System.out.println("Paid $" + amount + " by credit card"); }
}

class PayPalPayment implements PaymentStrategy {
    public void pay(double amount) { System.out.println("Paid $" + amount + " via PayPal"); }
}

class Checkout {
    private PaymentStrategy strategy;
    Checkout(PaymentStrategy strategy) { this.strategy = strategy; }
    void process(double amount) { strategy.pay(amount); }
}
```

---

### 22. Template Method

**Explain:** Defines the skeleton of an algorithm in a base class, deferring some specific steps to subclasses without changing the algorithm's overall structure.

**Real-life example:** A recipe for making tea or coffee — boil water, brew, pour into cup, add condiments — the steps are the same overall, but "brew" and "add condiments" differ for tea vs coffee.

**Use case:** Frameworks that define a fixed processing flow but allow customization at specific steps, such as test setup/teardown structures or data import pipelines with custom parsing logic.

**Where it's used in Spring:** `JdbcTemplate`, `RestTemplate`, and `TransactionTemplate` are literally named after this pattern — they define a fixed skeleton (open connection, execute, handle exceptions, close connection) while letting you plug in custom logic via callbacks. `AbstractApplicationContext.refresh()` also follows a template method structure.

**Code example:**
```java
abstract class DataProcessor {
    final void process() {
        readData();
        transformData();
        saveData();
    }
    abstract void readData();
    abstract void transformData();
    void saveData() { System.out.println("Saving to database (default)"); }
}

class CsvDataProcessor extends DataProcessor {
    void readData() { System.out.println("Reading CSV data"); }
    void transformData() { System.out.println("Transforming CSV data"); }
}
```

---

### 23. Visitor

**Explain:** Lets you define a new operation on a set of objects without changing the classes of the elements on which it operates, by separating the algorithm from the object structure.

**Real-life example:** A tax auditor (visitor) "visits" different types of business entities (corporations, sole proprietorships, partnerships) and applies the appropriate tax rules for each, without those businesses needing to know how to calculate their own taxes.

**Use case:** Operating on heterogeneous object structures like an AST (abstract syntax tree) in compilers, where different operations (type-checking, code generation, pretty-printing) need to be applied to the same tree of node types.

**Where it's used in Spring:** Spring's `BeanDefinitionVisitor` traverses and modifies `BeanDefinition` property values (e.g., resolving placeholders) without the bean definition classes needing to know about that specific operation.

**Code example:**
```java
interface Visitor {
    void visit(Circle circle);
    void visit(Square square);
}

interface Shape {
    void accept(Visitor visitor);
}

class Circle implements Shape {
    public void accept(Visitor visitor) { visitor.visit(this); }
}

class Square implements Shape {
    public void accept(Visitor visitor) { visitor.visit(this); }
}

class AreaCalculator implements Visitor {
    public void visit(Circle circle) { System.out.println("Calculating circle area"); }
    public void visit(Square square) { System.out.println("Calculating square area"); }
}
```

---

## Quick Reference Table

| Pattern | Category | Spring Example |
|---|---|---|
| Singleton | Creational | Default bean scope |
| Factory Method | Creational | `BeanFactory`, `FactoryBean` |
| Abstract Factory | Creational | `ApplicationContext` hierarchy |
| Builder | Creational | `UriComponentsBuilder` |
| Prototype | Creational | `prototype` bean scope |
| Adapter | Structural | `HandlerAdapter` |
| Bridge | Structural | `PlatformTransactionManager` |
| Composite | Structural | `CompositeFilter` |
| Decorator | Structural | `TransactionAwareDataSourceProxy` |
| Facade | Structural | `JdbcTemplate`, `RestTemplate` |
| Flyweight | Structural | Singleton bean cache |
| Proxy | Structural | AOP, `@Transactional`, `@Cacheable` |
| Chain of Responsibility | Behavioral | Servlet `FilterChain` |
| Command | Behavioral | `JdbcTemplate` callbacks |
| Interpreter | Behavioral | SpEL (`@Value`, `@PreAuthorize`) |
| Iterator | Behavioral | Bean definition iteration |
| Mediator | Behavioral | `ApplicationEventPublisher` |
| Memento | Behavioral | Transaction rollback/savepoints |
| Observer | Behavioral | `ApplicationListener`, `@EventListener` |
| State | Behavioral | Spring State Machine |
| Strategy | Behavioral | `AuthenticationProvider` |
| Template Method | Behavioral | `JdbcTemplate`, `RestTemplate` |
| Visitor | Behavioral | `BeanDefinitionVisitor` |
