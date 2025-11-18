# Creating Thread with Factory/Builder

Sometimes, you need to create many threads that share the same properties, like names, priorities, or exception handlers. 
Java provides tools to make this easier: `ThreadFactory` and `Thread.Builder`.


## ThreadFactory

A `ThreadFactory` is an interface with a single method:

```java
Thread newThread(Runnable r)
```

This method creates a new thread that will run the code in the given `Runnable`.

You can make your own class that implements `ThreadFactory`, or you can use a builder to create factories easily (Java 21 and later).

## Thread Builders

Thread builders let you configure threads before creating them. Java provides two types of builders:

- `Thread.ofPlatform()` – for platform threads (normal OS threads).
- `Thread.ofVirtual()` – for virtual threads (lightweight threads managed by the JVM).

You can set different properties for the threads you build. For example:

```java
Thread.Builder builder = Thread.ofVirtual().name("request-", 1);
```

This creates a builder that will name threads like `request-1`, `request-2`, etc. You can also customize:

- **Daemon threads:** `builder.daemon(true)` – threads that stop automatically when the program ends.
- **Exception handlers:** `builder.uncaughtExceptionHandler(handler)` – handles errors if the thread fails.
- **Thread groups**, **stack size**, and **priority** (for platform threads; priority is usually not recommended).

Once your builder is ready, you can create threads in different ways:

```java
Thread t1 = builder.unstarted(myRunnable);  // thread is created but not started
Thread t2 = builder.start(myRunnable);      // thread is created and starts immediately
```

Or you can make a ThreadFactory from the builder:

```java
ThreadFactory factory = builder.factory();
Thread t3 = factory.newThread(myRunnable);  // creates a thread using the factory
```

**Example:**

```java
Runnable task = () -> System.out.println("Running in thread: " + Thread.currentThread().getName());

Thread.Builder builder = Thread.ofVirtual().name("worker-", 1);

Thread t1 = builder.start(task);
Thread t2 = builder.start(task);

ThreadFactory factory = builder.factory();
Thread t3 = factory.newThread(task);
t3.start();
```

In this example:

- Threads `t1` and `t2` start immediately.
- `t3` is created using the factory and then started.
- All threads have names like `worker-1`, `worker-2`, etc.

## Why Use Builders and Factories?

- **Consistency** – all threads have the same settings.
- **Convenience** – you don’t have to set each property every time.
- **Better management** – easy to create large numbers of threads for tasks like handling requests in a server.

## Reference: Thread API Methods

- **`java.util.concurrent.ThreadFactory`** (Java 5.0)
    - `Thread newThread(Runnable r)` – creates a thread that, when started, executes the run method of the given Runnable.
- **`java.lang.Thread`** (Java 1.0)    
    - `Thread.Builder.OfPlatform ofPlatform()` – creates a builder for platform threads.
    - `Thread.Builder.OfVirtual ofVirtual()` – creates a builder for virtual threads.
- **`java.lang.Thread.Builder`** (Java 21)
    - `name(String prefix, long start)` – names threads with the given prefix and a counter starting from start.
    - `name(String name)` – sets a specific name for the thread.
    - `factory()` – creates a factory to build threads with this configuration.
    - `unstarted(Runnable task)` – creates a thread but does not start it.
    - `start(Runnable task)` – creates a thread and starts it immediately.
    - `uncaughtExceptionHandler(Thread.UncaughtExceptionHandler ueh)` – sets a handler for exceptions that are not caught in the thread.    
- **`Thread.Builder.OfPlatform`** (Java 21)
    - `daemon() / daemon(boolean on)` – creates daemon threads (stop automatically when program ends).
    - `stackSize(long stackSize)` – sets a stack size hint for the thread.
    - `group(ThreadGroup group)` – assigns the thread to a specific group.
    - `priority(int priority)` – sets the thread priority (not recommended in modern Java).    

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [How Thread Priorities Work](./8_Thread_Priorities.md)
- [Callables and Futures](./10_Callables_and_Futures.md)