## Thread

```
Author: Ter-Petrosyan Hakob
```

---

You may already know about multitasking, which is when your computer seems to run several programs at the same time. For instance, you might be typing a document while listening to music. Even if your computer has only one CPU, it can still make it look like multiple programs run together by quickly switching between them.

Multithreading is similar, but it happens inside a single program. A multithreaded program can perform several tasks at the same time. Each task runs in a thread, which is a small unit of execution inside a program. Programs that can run multiple threads at once are called multithreaded programs.

## Threads vs Processes

You might wonder: what is the difference between threads and processes?

- Each process has its own separate memory and variables.
- Threads share the same memory and variables within a program.

Sharing variables can be risky, because two threads might try to change the same data at the same time. However, it also makes communication between threads faster and simpler than communication between separate processes. Additionally, threads are lighter than processes, meaning they use less memory and are faster to start and stop.

## Why Multithreading is Useful

Multithreading is important in many real-world applications. For example:

- A file downloader can download multiple files at the same time.
- A chat application can receive messages while sending others.
- A video game can process user input and animate graphics simultaneously.

## Running Thread

Here is how you can run a task in a separate thread:

1. Put the task’s code inside the run method of a class that implements the `Runnable` interface:
    ```java
    public interface Runnable {
        void run();
    }

    ```
2. Since `Runnable` is a functional interface, you can also use a lambda expression:    
    ```java
    Runnable task = () -> {
        // your task code here
    };
    ```
3. Create a Thread object from the Runnable:
    ```java
    Thread t = new Thread(task);
    ```
4. Start the thread:
    ```java
    t.start();
    ```    

## Simultaneous Package Delivery

Imagine a small delivery company with four trucks. Each truck delivers packages to different customers. Instead of sending one truck at a time, the company can send multiple trucks at the same time using threads.

```java
Runnable truck1 = () -> {
    try {
        for (int i = 0; i < 5; i++) {
            System.out.println("Truck 1 delivers package " + (i+1));
            Thread.sleep((int)(500 * Math.random())); // random delay
        }
    } catch (InterruptedException e) {
        System.out.println("Truck 1 stopped");
    }
};

Runnable truck2 = () -> {
    try {
        for (int i = 0; i < 5; i++) {
            System.out.println("Truck 2 delivers package " + (i+1));
            Thread.sleep((int)(500 * Math.random()));
        }
    } catch (InterruptedException e) {
        System.out.println("Truck 2 stopped");
    }
};

new Thread(truck1).start();
new Thread(truck2).start();
```

When this program runs, you will see messages from both trucks interleaved, showing that they are delivering packages concurrently.

> **Important:** Do not call the run method directly. This will execute the task in the current thread only. Use `start()` to create a new thread.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
