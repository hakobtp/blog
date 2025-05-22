# 📘 Java Record

```
Author: Ter-Petrosyan Hakob
```

---

Java’s virtual threads (from `Project Loom`) are a game-changer for high-concurrency applications. 
They let us create millions of threads without blowing up memory or overwhelming the OS. 
In this post, we’ll explore how virtual threads work under the hood in simple terms, 
and how they differ from traditional platform threads (the normal Java threads backed by OS threads). 
We’ll look at how the JVM schedules virtual threads, what mounting and unmounting mean, how stack memory is handled, 
what continuations are, how virtual threads behave during blocking calls and synchronized blocks, and some limitations 
(like pinning) and best practices to get the most out of this new feature.

## What Are Virtual Threads (vs Platform Threads)?

**Platform threads** (the regular threads we’ve always used in Java) are essentially a one-to-one wrapper 
around an OS thread. Each platform thread is managed by the operating system, has a relatively large fixed stack 
(often around 1MB by default), and context-switching between them is handled by the OS. Because they’re heavy, 
creating thousands of platform threads can exhaust system resources.

**Virtual threads** introduced as a preview in JDK 19 and finalized in JDK 21, are 
lightweight threads managed by the JVM rather than the OS. A virtual thread is not tied one-to-one to a native thread. 
Instead, many virtual threads can share a few OS threads under the hood. Key differences between virtual and platform threads include:

- **Mapping:** A platform thread is a 1:1 mapping to an OS thread. A virtual thread is scheduled onto an 
    OS thread only when it’s doing work. Think of virtual threads as tasks that run on a pool of OS threads (often called carrier threads).
- **Memory footprint:** Platform threads carry a large memory stack (megabytes) allocated from the OS. Virtual threads start with a small stack 
    (just a few hundred bytes) and their stack frames are stored on the Java heap. This makes virtual threads cheaper in terms of memory.
- **Scalability:** Because of their low cost, you can create hundreds of thousands or even millions of virtual threads in an application 
    without running out of memory. They are designed to enable a thread-per-task programming style for high-throughput servers, 
    where each concurrent task (like a user request) gets its own thread.
- **Scheduling:** Platform threads are scheduled preemptively by the OS kernel (the OS decides when to context-switch between threads). 
    Virtual threads, on the other hand, are scheduled by the JVM at runtime and use a form of cooperative scheduling – 
    they run until they reach a point where they voluntarily yield (usually by blocking or waiting). We’ll explain this more shortly.    
- **CPU utilization:** Virtual threads don’t magically give you more CPU than you have cores. If you run CPU-intensive work on many virtual threads, 
    it behaves similar to many platform threads – the OS will time-slice across the available carrier threads. The big win is with I/O or other blocking operations, where virtual threads that are waiting don’t consume any OS thread or CPU time.    

In short, a virtual thread is a lightweight, managed thread that the JVM can park and resume efficiently. 
This lets us write code in the simple thread-per-request style and still handle huge concurrency.    

---

## 📌 Explore More

- 🏠 [Home](./../../README.md)
- ☕ [Java Tutorials](./../tutorials.md)

---