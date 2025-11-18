# Handling Uncaught Exceptions

```
Author: Ter-Petrosyan Hakob
```

---

In Java, each thread runs its own run method. This method cannot throw checked exceptions, but it can stop suddenly if an unchecked exception happens. 
When that occurs, the thread ends immediately.

But here’s the problem: there is no normal catch block that can catch this exception in the usual way. Instead, Java provides a special mechanism called an uncaught exception handler. This handler receives the exception right before the thread dies.

## What Is an Uncaught Exception Handler?

An uncaught exception handler is a special object that decides what to do when a thread fails unexpectedly. 
To create one, you need a class that implements the `Thread.UncaughtExceptionHandler` interface. This interface has just one method:

```java
void uncaughtException(Thread t, Throwable e)
```

- `t` – the thread where the exception happened
- `e` – the exception that caused the thread to stop


## How to Install a Handler

You can attach a handler to any single thread using:

```java
thread.setUncaughtExceptionHandler(myHandler);
```

You can also set a default handler for all threads using:

```java
Thread.setDefaultUncaughtExceptionHandler(myHandler);
```

A common use is to log the exception in a file, so you can investigate later. If you don’t set any handler, Java uses a built-in default behavior.

## What Happens Without a Handler?

If a thread does not have a custom handler, Java follows these rules in order:

1. **Thread group parent** – If the thread belongs to a group that has a parent, Java calls the parent’s `uncaughtException` method.
2. **Default handler** – If a default handler exists (`Thread.getDefaultUncaughtExceptionHandler()`), it is called.
3. **ThreadDeath exception** – If the exception is a `ThreadDeath` object (an old way to stop threads), nothing happens.
4. **Print to console** – If none of the above apply, Java prints the thread name and a stack trace to `System.err`.

The stack trace is the familiar error message you often see when a program crashes.

## Thread Groups and Exceptions

Technically, each thread belongs to a thread group, which can define the uncaught exception behavior. A thread group is like a small collection of threads managed together. By default, all threads you create belong to the main group, but you can create new groups if you want.

> **Important note:** Modern Java programs rarely use thread groups because newer tools 
> (like executors and thread pools) handle threads more efficiently. So it’s usually better to avoid thread groups in your code.

**Example:** Logging Uncaught Exceptions

Here’s an example where we log an uncaught exception to the console:

```java
class MyHandler implements Thread.UncaughtExceptionHandler {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.println("Thread " + t.getName() + " failed.");
        System.out.println("Reason: " + e.getMessage());
    }
}

public class TestThread {
    public static void main(String[] args) {
        Thread thread = new Thread(() -> {
            throw new RuntimeException("Oops! Something went wrong.");
        });
        thread.setUncaughtExceptionHandler(new MyHandler());
        thread.start();
    }
}
```

Output example:

```
Thread Thread-0 failed.
Reason: Oops! Something went wrong.
```

This way, even though the thread crashes, we can see what happened and take action, like saving the error in a log file.

> **Takeaway:** Uncaught exception handlers let you control how threads fail. Without them, Java prints 
> a stack trace, which may be useful but not always enough. For production programs, logging or reporting uncaught exceptions is highly recommended.


## Flow of an Uncaught Exception

Here’s a simple diagram showing what happens when a thread throws an exception:

```
Thread throws an unchecked exception
               │
               ▼
Does the thread have a custom handler?
       ┌─────────────┴─────────────┐
       │                           │
      Yes                          No
       │                           │
       ▼                           ▼
Custom handler runs          Does thread group have a parent?
                               ┌───────────┴───────────┐
                               │                       │
                              Yes                      No
                               │                       │
                               ▼                       ▼
                  Parent group's handler runs   Is there a default handler?
                                                      ┌───────┴───────┐
                                                      │               │
                                                     Yes              No
                                                      │               │
                                                      ▼               ▼
                                        Default handler runs    Print stack trace

```

## Multiple Threads with Handlers

This example shows how different threads can have different handlers:

```java
class MyHandler implements Thread.UncaughtExceptionHandler {
    private String name;

    public MyHandler(String name) {
        this.name = name;
    }

    @Override
    public void uncaughtException(Thread t, Throwable e) {
        System.out.println("Handler " + name + " caught exception in " + t.getName());
        System.out.println("Reason: " + e.getMessage());
    }
}

public class MultiThreadExample {
    public static void main(String[] args) {

        // Default handler for all threads
        Thread.setDefaultUncaughtExceptionHandler((t, e) -> {
            System.out.println("Default handler: " + t.getName() + " crashed!");
            System.out.println("Error: " + e.getMessage());
        });

        // Thread with a custom handler
        Thread thread1 = new Thread(() -> {
            throw new RuntimeException("Oops! Thread 1 failed.");
        });
        thread1.setUncaughtExceptionHandler(new MyHandler("CustomHandler1"));

        // Thread without a custom handler (uses default)
        Thread thread2 = new Thread(() -> {
            throw new RuntimeException("Oops! Thread 2 failed.");
        });

        // Another thread with its own custom handler
        Thread thread3 = new Thread(() -> {
            throw new RuntimeException("Oops! Thread 3 failed.");
        });
        thread3.setUncaughtExceptionHandler(new MyHandler("CustomHandler3"));

        thread1.start();
        thread2.start();
        thread3.start();
    }
}
```

Example Output:

```java
Handler CustomHandler1 caught exception in Thread-0
Reason: Oops! Thread 1 failed.
Default handler: Thread-1 crashed!
Error: Oops! Thread 2 failed.
Handler CustomHandler3 caught exception in Thread-2
Reason: Oops! Thread 3 failed.
```

---

- `java.lang.Thread`
    - `static void setDefaultUncaughtExceptionHandler(Thread.UncaughtExceptionHandler handler)`  Sets the default handler for exceptions that are not caught in any thread.
    - `static Thread.UncaughtExceptionHandler getDefaultUncaughtExceptionHandler()` Returns the default handler for exceptions that are not caught in any thread.
    - `void setUncaughtExceptionHandler(Thread.UncaughtExceptionHandler handler)` Sets the handler for exceptions that are not caught in this thread.
    - `Thread.UncaughtExceptionHandler getUncaughtExceptionHandler()` Returns the handler for exceptions that are not caught in this thread.  
      If no handler is set, the thread group will handle the exception.

---

- `java.lang.Thread.UncaughtExceptionHandler`
    - `void uncaughtException(Thread t, Throwable e)` Called when a thread ends because of an exception that is not caught.  
      This method can be used to log a custom report or handle the exception in a special wa

---

- `java.lang.ThreadGroup` 
    - `void uncaughtException(Thread t, Throwable e)` Called when a thread in this group ends because of an exception that is not caught.  
      If this group has a parent, the parent’s method is called.  
      If there is no parent but the Thread class has a default handler, that handler is called.  
      Otherwise, the exception’s stack trace is printed to the standard error stream.


```java
public class ThreadGroupExample {

    public static void main(String[] args) {
        // Create a thread group
        ThreadGroup group = new ThreadGroup("MyGroup");

        // Create a thread in the group
        Thread t1 = new Thread(group, () -> {
            System.out.println("Thread 1 is running");
            throw new RuntimeException("Oops!"); // This will cause an uncaught exception
        });

        // Create another thread in the same group
        Thread t2 = new Thread(group, () -> {
            System.out.println("Thread 2 is running");
        });

        // Override uncaughtException for this thread group
        group.setUncaughtExceptionHandler((thread, exception) -> {
            System.out.println("Exception in thread " + thread.getName() + ": " + exception.getMessage());
        });

        // Start threads
        t1.start();
        t2.start();
    }
}
```

1. We create a `ThreadGroup` called "MyGroup".
2. Two threads are created in that group. The first thread throws an uncaught exception.
3. The `uncaughtException` handler in the thread group catches the exception and prints a custom message.
4. If we did not have the handler, the exception would go to the default handler or print a stack trace.

## Summary
- Threads can die unexpectedly because of unchecked exceptions.
- Uncaught exception handlers let you control what happens when a thread fails.
- Each thread can have its own handler, or you can use a global default handler.
- Modern programs usually avoid thread groups and rely on executors and logging.
- Using handlers properly ensures your program can report errors and recover gracefully, even when threads crash.


---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Daemon Threads and Thread Names and IDs](./6_Daemon_Threads_and_Thread_Names_and_IDs.md)
- [How Thread Priorities Work](./8_Thread_Priorities.md)