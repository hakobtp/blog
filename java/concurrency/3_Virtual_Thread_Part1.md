# Virtual Thread Part 1

```
Author: Ter-Petrosyan Hakob
```

---

In Java, a normal thread usually runs on a platform thread. A platform thread is created and managed by the operating system where your Java program is running. For many applications, this works well because modern operating systems are good at managing threads and dividing CPU time among them.

However, platform threads are not lightweight. Creating a thread uses memory and CPU resources. Each thread reserves some memory for its stack and requires several thousand CPU instructions to start. Because of this, a computer can only handle a limited number of threads efficiently.

This limitation becomes noticeable in applications that handle many tasks at the same time. For example, a web server might need to respond to thousands of users at once. Most of the time, each task waits for something external, like a database query or a request to another service. These tasks are mostly idle, so the CPU isn’t doing much work, but traditional threads still occupy memory and resources.

## Non-Blocking APIs: One Approach

One common way to solve this is by using non-blocking programming. In traditional programming, you might write:

```java
String result = service.getData(params);
process(result);
```

Here, the thread waits until `getData` finishes. The thread can’t do anything else while waiting.

With non-blocking APIs, you write the code differently:

```java
service.getData(params, response -> process(response));
```

Now, the request returns immediately, and the thread can work on something else. When the response is ready, the callback function (`response -> process(response)`) is called.

Non-blocking APIs allow programs to handle many requests efficiently, but they make code harder to read and write. Instead of using normal loops, conditionals, and exceptions, you must write your program in a series of callbacks, which can become confusing.

## Virtual Threads: A Simpler Solution

Java 21 introduced virtual threads. A virtual thread is a lightweight thread that can pause when it waits for something (like a database query) and resume later.

Virtual threads work like this:

1. A virtual thread runs on a platform thread.
2. When it encounters a blocking operation (like waiting for data), it is “unmounted” from the platform thread.
3. Once the result is ready, the virtual thread can be rescheduled and mounted on a platform thread again.

Think of virtual threads like virtual memory. Just like virtual memory allows a computer to use more memory than physically available, virtual threads let you run many more threads than there are platform threads.

By default, Java uses one carrier thread per CPU core to schedule virtual threads. You can change this setting with:

```
jdk.virtualThreadScheduler.parallelism
```

## Using Virtual Threads

Virtual threads have the same API as normal threads, making them easy to use. For example, you can create a virtual thread like this:

```java
Thread vt = Thread.startVirtualThread(() -> {
    System.out.println("Hello from a virtual thread!");
});
```

You can check if a thread is virtual:

```java
if (vt.isVirtual()) {
    System.out.println("This is a virtual thread!");
}
```

## Key points to remember:

- Virtual threads are lightweight and can handle thousands of concurrent tasks.
- They simplify writing concurrent programs because you don’t need complex callbacks.
- They are especially useful for tasks that spend a lot of time waiting for external resources, like web servers or database applications.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Thread States](./2_Thread_States.md)
- [Virtual Thread Part 2](./4_Virtual_Thread_Part2.md)