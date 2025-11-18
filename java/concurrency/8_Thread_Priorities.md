# How Thread Priorities Work

Every thread in Java has a priority. By default, a thread takes the priority of the thread that created it. 
You can change a thread’s priority using the `setPriority` method. The priority must be a number between `MIN_PRIORITY (1)` and `MAX_PRIORITY (10)`. 
The normal priority, `NORM_PRIORITY`, is `5`.

The thread scheduler in Java usually prefers threads with higher priority when deciding which thread to run next. 
However, thread priorities do not work exactly the same on every computer. This is because Java threads rely on the 
operating system’s thread system. Some systems may have more or fewer priority levels than Java, so Java priorities are
“mapped” to the closest operating system priority.

For example:

- On Windows, there are seven priority levels. Multiple Java priorities may end up using the same Windows priority.
- On Linux with OpenJDK, all threads are treated equally, and priorities are ignored.

In early versions of Java, thread priorities could help control which thread ran first, especially when Java did not 
use operating system threads. Today, they are not very useful, and most programs do not need to set thread priorities.

For virtual threads, the priority is always `NORM_PRIORITY`, and trying to change it will have no effect.

## Important Methods and Constants

- `void setPriority(int newPriority)` – Changes the priority of the thread. The value must be between 
    `Thread.MIN_PRIORITY` and `Thread.MAX_PRIORITY`. Use `Thread.NORM_PRIORITY` for a normal priority.
- `Thread.MIN_PRIORITY` – The lowest possible priority (1).
- `Thread.NORM_PRIORITY` – The default priority (5).
- `Thread.MAX_PRIORITY` – The highest possible priority (10).

**Example:**

```java
Thread highThread = new Thread(() -> {
    System.out.println("This thread has high priority.");
});
highThread.setPriority(Thread.MAX_PRIORITY);

Thread lowThread = new Thread(() -> {
    System.out.println("This thread has low priority.");
});
lowThread.setPriority(Thread.MIN_PRIORITY);

highThread.start();
lowThread.start();
```

In this example, the scheduler may run `highThread` before `lowThread`, but the behavior can change depending on 
the operating system. On Linux, both threads may run in any order because Linux ignores Java thread priorities.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Handling Uncaught Exceptionss](./7_Handling_Uncaught_Exceptions.md)
- [Creating Thread with Factory/Builder](./9_Creating_Thread_with_Factory_Builder.md)