Multithreading in Java (and software engineering in general) is a massive topic. To make it easy to digest, 
we can break it down into five logical categories: The Basics, The Hazards (what goes wrong), Synchronization 
(how to fix it), Advanced Concurrency Tools, and Asynchronous Frameworks.

Here is the complete roadmap of multithreading concepts.

# 1. Executor & ExecutorService Framework
The Executor framework decouples task submission from task execution. Instead of manually spawning new Thread(), you pass tasks to a managed pool.

Differences between execute() and submit()
execute(Runnable): Defined in the Executor interface. It executes the task asynchronously. It returns void, meaning you have no way of knowing if the task completed successfully or catching its exceptions.

submit(Callable/Runnable): Defined in the ExecutorService interface. It returns a Future<T> object. It allows you to intercept exceptions and fetch a return value from your task.

Runnable vs. Callable
Runnable: Its method run() returns void and cannot throw checked exceptions.

Callable: Its method call() returns a value of type T and can throw checked exceptions.

# 2. Atomic Operations & Custom Implementations
Atomic operations are non-blocking, lock-free operations that leverage low-level CPU hardware instructions like CAS (Compare-And-Swap).

## Custom AtomicInteger Implementation
Standard implementations use sun.misc.Unsafe or VarHandle to access memory directly. We can mock a custom version using a 
loop and a synchronized CAS simulation to see how it works conceptually under the hood:
java```
public class CustomAtomicInteger {
    private volatile int value;

    public CustomAtomicInteger(int initialValue) {
        this.value = initialValue;
    }

    public int get() { return value; }

    // The Compare-And-Swap operation
    public synchronized boolean compareAndSet(int expectedValue, int newValue) {
        if (value == expectedValue) {
            value = newValue;
            return true;
        }
        return false;
    }

    // Increments and returns the updated value using a CAS loop
    public int incrementAndGet() {
        int current;
        int next;
        do {
            current = get();
            next = current + 1;
        } while (!compareAndSet(current, next)); // Retries if another thread modified it first
        return next;
    }
}

```

An identical structural pattern applies to custom AtomicLong (using long) and AtomicBoolean (mapping booleans to 1/0 or conditional CAS).

# 3. Semaphores & The Producer-Consumer Pattern
A Semaphore maintains a set of permits. Threads block if no permits are available.

## Custom Semaphore Implementation
java```
public class CustomSemaphore {
    private int permits;

    public CustomSemaphore(int permits) {
        this.permits = permits;
    }

    public synchronized void acquire() throws InterruptedException {
        while (permits <= 0) {
            wait();
        }
        permits--;
    }

    public synchronized void release() {
        permits++;
        notifyAll();
    }
}
```
### Producer-Consumer Pattern using Custom Semaphore
By tracking empty spaces and available items via two complementary semaphores, you avoid race conditions entirely.
java```
public class SemaphoreProducerConsumer {
    private static final int CAPACITY = 5;
    private final Queue<Integer> buffer = new LinkedList<>();
    
    private final CustomSemaphore empty = new CustomSemaphore(CAPACITY);
    private final CustomSemaphore full = new CustomSemaphore(0);
    private final Object mutex = new Object(); // For safe queue modification

    public void produce(int item) throws InterruptedException {
        empty.acquire(); // Decrements empty slots
        synchronized (mutex) {
            buffer.add(item);
        }
        full.release(); // Increments available items
    }

    public int consume() throws InterruptedException {
        full.acquire(); // Decrements available items
        int item;
        synchronized (mutex) {
            item = buffer.poll();
        }
        empty.release(); // Increments empty slots
        return item;
    }
}
```

