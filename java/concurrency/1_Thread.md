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

> **Important:** 
>
> Do not call the run method directly. This will execute the task in the current thread only. Use `start()` to create a new thread.

> **Performance Note:**
>
> In our truck example, each thread uses `Math.random()` to create random delays. When many threads run at the same time, 
> calling `Math.random()` a lot can be slightly slow, because it uses a shared random number generator internally.
>
> For small programs, this is not a problem. But in high-performance or multithreaded programs, a better approach is to use 
> `ThreadLocalRandom`, which gives each thread its own random generator. This avoids threads competing with each other and makes your program faster.
>
> `int delay = ThreadLocalRandom.current().nextInt(100, 501); // random delay 100–500ms`
>
> This method is fast, safe, and recommended for multithreaded Java programs.

## Creating a Thread by Subclassing

Another way to create a thread is by subclassing the Thread class:

```java
class DeliveryTruck extends Thread {
    public void run() {
        for (int i = 0; i < 5; i++) {
            System.out.println(getName() + " delivers package " + (i+1));
        }
    }
}

DeliveryTruck t1 = new DeliveryTruck();
t1.start();
```

However, this approach is less flexible, because it mixes the task with the thread mechanism. Using `Runnable` is recommended.

## Key Points

- Threads allow multiple tasks in a single program to run concurrently.
- Threads share memory, unlike separate processes.
- Always use `Thread.start()` to run a thread.
- You can use `Runnable` for better flexibility, or subclass `Thread` (less recommended).
- Generating random numbers in many threads can be slightly inefficient, but it is usually fine for small programs.

---

- `java.lang.Thread`
    - `Thread(Runnable target)` Creates a new thread. When the thread runs, it will execute the `run()` method of the given `Runnable` object.
    - `void start()` Starts the thread. The thread will begin running the `run()` method, and the program continues immediately without waiting. The new thread runs at the same time as other threads.
    - `void run()` Executes the code in the `run()` method of the associated `Runnable`. You usually do not call this directly; use `start()` instead.
    - `static void sleep(long millis)`  Pauses the current thread for the given number of milliseconds.
    - `static void sleep(Duration duration)` Pauses the current thread for the time given by a `Duration` object.

---

- `java.lang.Runnable` 
    - `void run()` You must override this method to write the instructions for the task you want the thread to perform.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Thread States](./2_Thread_States.md)