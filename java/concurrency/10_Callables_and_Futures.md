## Callables and Futures

A `Runnable` represents a task that runs asynchronously. Think of it like a method that does not take input and 
does not return a value. A `Callable` is similar but returns a value. It has a single method:

```java
public interface Callable<V> {
    V call() throws Exception;
}
```

- `V` is the type of the value the task returns.
- For example, `Callable<Integer>` represents a task that returns an `Integer` when it finishes.

A `Future` holds the result of an asynchronous task. When you schedule a task, you get a `Future` object. Later, you can use it to `get` the result when it is ready.


The `Future<V>` interface provides several important methods:
- `V get()` – waits until the task is finished, then returns the result.
- `V get(long timeout, TimeUnit unit)` – waits for a specific time; throws TimeoutException if the task does not finish in time.
- `V resultNow()` – returns the result immediately, only if the task finished successfully.
- `Throwable exceptionNow()` – returns the exception if the task failed.
- `void cancel(boolean mayInterrupt)` – tries to cancel the task.
- `boolean isCancelled()` – returns true if the task was cancelled.
- `boolean isDone()` – returns true if the task finished (successfully, failed, or cancelled).
- `Future.State state()` – shows the current state: RUNNING, SUCCESS, FAILED, or CANCELLED.

**Tip:** Always check the task state before calling `resultNow()` or `exceptionNow()`.


## Example with FutureTask

`FutureTask` implements both `Runnable` and `Future`, so you can run it in a thread and get its result:

```java
Callable<Integer> task = () -> {
    Thread.sleep(1000);
    return 42;
};

FutureTask<Integer> futureTask = new FutureTask<>(task);
Thread.ofPlatform().start(futureTask); // runs the task in a thread

Integer result = futureTask.get(); // waits for the task to finish
System.out.println("Task result: " + result);
```

- `futureTask` is a `Runnable` when you start it.
- `futureTask` is a `Future` when you get the result.

You can also wrap a `Runnable` in a `FutureTask`, but it returns `null`:

```java
Runnable myRunnable = () -> System.out.println("Running a simple task");
FutureTask<Void> futureTask = new FutureTask<>(myRunnable, null);
Thread.ofPlatform().start(futureTask);
futureTask.get(); // waits for completion, returns null
```

## Canceling Tasks

In Java, you can attempt to cancel a task that is running asynchronously (for example, a `Callable` or `FutureTask`) using the cancel method:

```java
futureTask.cancel(true);
```

The behavior of cancel depends on two factors:

- Whether the task has started running or not.
- The value of the mayInterrupt parameter.

## Task Has Not Started Yet

If the task has not yet started, cancellation is simple:
- The task is marked as cancelled.
- It will never run.
- `isCancelled()` returns `true`.
- `isDone()` also returns `true`.

**Example:**
```java
FutureTask<Integer> futureTask = new FutureTask<>(() -> {
    System.out.println("Running task");
    return 42;
});

// Task not started yet
futureTask.cancel(true);
System.out.println(futureTask.isCancelled()); // true
System.out.println(futureTask.isDone());      // true
```

The task body (`System.out.println("Running task")`) never executes.

### Task Is Already Running

If the task has started, cancellation becomes tricky:
```java
futureTask.cancel(true);
```

- `mayInterrupt = true` → Java tries to interrupt the thread running the task.
- `mayInterrupt = false` → The task continues to run to completion.

Interrupting a thread **does not automatically stop it**. In Java:
- Each thread has an interrupted status.
- Calling `thread.interrupt()` sets this status to `true`.
- The task code must check this status and stop if necessary.

Example of cooperative cancellation:
```java
Callable<Void> task = () -> {
    while (!Thread.currentThread().isInterrupted()) {
        // Do some work
        System.out.println("Working...");
        Thread.sleep(500); // sleep can throw InterruptedException
    }
    System.out.println("Task was interrupted!");
    return null;
};

FutureTask<Void> futureTask = new FutureTask<>(task);
Thread t = new Thread(futureTask);
t.start();

Thread.sleep(1000);
futureTask.cancel(true); // interrupts the task

```

Output may look like:
```
Working...
Working...
Task was interrupted!
```

**Key points:**

- Thread checks for interruption using `Thread.currentThread().isInterrupted()`.
- Some methods like `Thread.sleep()` automatically throw `InterruptedException` when interrupted.
- If the task ignores interruption, it will continue running even after `cancel(true)`.

**Why mayInterrupt Exists:**
- `true` → attempt to stop a running task by sending an interrupt signal.
- `false` → do not interrupt, just mark the task as cancelled; it may still finish normally.

**Always remember:** Java cannot force-stop a thread safely. Cancellation is cooperative. The task must cooperate by periodically checking its interrupted statu

## Summary

- **Runnable:** no return value.
- **Callable:** returns a value.
- **Future:** holds the result of an asynchronous task.
- **FutureTask:** can be run in a thread and provides a result.

Using Callables and Futures makes it easier to split a program into subtasks, run them in parallel, and combine results efficiently.

## Reference: Thread and Task API Methods

- **`java.util.concurrent.Callable<V>`** (Java 5.0)
    - `V call()` – runs a task that returns a result.
- **`java.util.concurrent.Future<V>`** (Java 5.0)
    - `V get()` – waits for the result.
    - `V get(long time, TimeUnit unit)` – waits for the result with timeout; may throw `TimeoutException`, `ExecutionException`, or `InterruptedException`.
    - `V resultNow()` – gets the result immediately; throws `IllegalStateException` if task not finished.
    - `Throwable exceptionNow(`) – gets the exception if task failed; throws `IllegalStateException` if task succeeded.
    - `boolean cancel(boolean mayInterrupt)` – tries to cancel the task.
    - `boolean isCancelled()` – returns true if task was cancelled.
    - `boolean isDone()` – returns true if task finished or was cancelled.
    - `Future.State state()` – returns RUNNING, SUCCESS, FAILED, or CANCELLED.
- **`java.util.concurrent.FutureTask<V>`** (Java 5.0)
    - `FutureTask(Callable<V> task)` – creates a FutureTask from a Callable.
    - `FutureTask(Runnable task, V result)` – creates a FutureTask from a Runnable, returning result after completion.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Creating Thread with Factory/Builder](./9_Creating_Thread_with_Factory_Builder.md)