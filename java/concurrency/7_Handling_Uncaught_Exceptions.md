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

---



---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Daemon Threads and Thread Names and IDs](./6_Daemon_Threads_and_Thread_Names_and_IDs.md)