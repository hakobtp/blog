# Handling Uncaught Exceptions

```
Author: Ter-Petrosyan Hakob
```

---

In Java, each thread runs its own run method. This method cannot throw checked exceptions, but it can stop suddenly if an unchecked exception happens. When that occurs, the thread ends immediately.

But here’s the problem: there is no normal catch block that can catch this exception in the usual way. Instead, Java provides a special mechanism called an uncaught exception handler. This handler receives the exception right before the thread dies.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Daemon Threads and Thread Names and IDs](./6_Daemon_Threads_and_Thread_Names_and_IDs.md)