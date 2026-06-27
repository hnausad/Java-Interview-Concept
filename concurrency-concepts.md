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

## Custom `AtomicInteger` Implementation
Standard implementations use sun.misc.Unsafe or VarHandle to access memory directly. We can mock a custom version using a 
loop and a synchronized CAS simulation to see how it works conceptually under the hood:
```java
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

### Custom Semaphore Implementation
```java
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
```java
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

# 4. Locks & ReentrantLock
A ReentrantLock allows a thread to re-acquire a lock it already holds without deadlocking itself.

Difference between synchronized and ReentrantLock
synchronized is implicit and block-scoped. ReentrantLock allows manual `.lock()` and `.unlock()` across different methods.

ReentrantLock provides advanced features: fairness policies, lock polling `(tryLock())`, and interrupted lock waiting.

### Custom `ReentrantLock` Implementation
```java
public class CustomReentrantLock {
    private boolean isLocked = false;
    private Thread lockingThread = null;
    private int lockCount = 0;

    public synchronized void lock() throws InterruptedException {
        Thread callingThread = Thread.currentThread();
        while (isLocked && lockingThread != callingThread) {
            wait();
        }
        isLocked = true;
        lockCount++;
        lockingThread = callingThread;
    }

    public synchronized void unlock() {
        if (Thread.currentThread() == this.lockingThread) {
            lockCount--;
            if (lockCount == 0) {
                isLocked = false;
                lockingThread = null;
                notify();
            }
        }
    }
}

```
Solving Producer-Consumer via newCondition()
Condition variables decouple thread signaling into unique wait sets per lock instance.

```java
public class ConditionProducerConsumer {
    private final ReentrantLock lock = new ReentrantLock();
    private final Condition notFull = lock.newCondition();
    private final Condition notEmpty = lock.newCondition();
    private final Queue<Integer> queue = new LinkedList<>();
    private final int capacity = 10;

    public void produce(int val) throws InterruptedException {
        lock.lock();
        try {
            while (queue.size() == capacity) {
                notFull.await(); // Wait until there's space
            }
            queue.add(val);
            notEmpty.signalAll(); // Signal consumers
        } finally {
            lock.unlock();
        }
    }

    public int consume() throws InterruptedException {
        lock.lock();
        try {
            while (queue.isEmpty()) {
                notEmpty.await(); // Wait until there's data
            }
            int val = queue.poll();
            notFull.signalAll(); // Signal producers
            return val;
        } finally {
            lock.unlock();
        }
    }
}
```

### Applications of `ReentrantLock`
High-concurrency data structures where starvation must be controlled via fair locking.

Algorithms needing non-blocking lock acquisition (tryLock()) to prevent immediate thread stall.

Implementing complex, nested multi-resource locking patterns.
```java
public class CustomReadWriteLock {
    private int readers = 0;
    private int writers = 0;
    private int writeRequests = 0;

    public synchronized void lockRead() throws InterruptedException {
        while (writers > 0 || writeRequests > 0) {
            wait();
        }
        readers++;
    }

    public synchronized void unlockRead() {
        readers--;
        notifyAll();
    }

    public synchronized void lockWrite() throws InterruptedException {
        writeRequests++;
        while (readers > 0 || writers > 0) {
            wait();
        }
        writeRequests--;
        writers++;
    }

    public synchronized void unlockWrite() {
        writers--;
        notifyAll();
    }
}

```

6. Thread Synchronizers: CountDownLatch vs. CyclicBarrier
   ### Feature	CountDownLatch	                                                CyclicBarrier
       Reuse	Cannot be reset. One-time use.	                                Can be reset and reused after threads break through.
   Mechanism	Waiting threads block until counter reaches 0 via countDown().	Threads wait for each other at a barrier point via await().
   Focus        One or more threads wait for N events.                          N threads wait for each other to coordinate.

### Custom CountDownLatch Implementation
```java
public class CustomCountDownLatch {
    private int count;

    public CustomCountDownLatch(int count) {
        this.count = count;
    }

    public synchronized void await() throws InterruptedException {
        while (count > 0) {
            wait();
        }
    }

    public synchronized void countDown() {
        if (count > 0) {
            count--;
            if (count == 0) {
                notifyAll();
            }
        }
    }
}

```
