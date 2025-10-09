## Thread

```
Author: Ter-Petrosyan Hakob
```

---

You may already know about multitasking, which is when your computer seems to run several programs at the same time. For instance, you might be typing a document while listening to music. Even if your computer has only one CPU, it can still make it look like multiple programs run together by quickly switching between them.

Multithreading is similar, but it happens inside a single program. A multithreaded program can perform several tasks at the same time. Each task runs in a thread, which is a small unit of execution inside a program. Programs that can run multiple threads at once are called multithreaded programs.

## Threads vs. Processes

You might wonder: what is the difference between threads and processes?

- Each process has its own separate memory and variables.
- Threads share the same memory and variables within a program.

Sharing variables can be risky, because two threads might try to change the same data at the same time. However, it also makes communication between threads faster and simpler than communication between separate processes. Additionally, threads are lighter than processes, meaning they use less memory and are faster to start and stop.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
