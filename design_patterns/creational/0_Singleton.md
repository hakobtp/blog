# Singleton

The Singleton Pattern in Java is a popular design pattern. It belongs to the group called Creational Design Patterns, which focus on ways to create objects.
At first, the idea of a singleton looks very simple. But when we try to use it in real projects, some extra problems appear.

In this article, we will:

- Learn the main ideas behind the Singleton Pattern.
- Explore different ways to write it in Java.
- Look at common mistakes and best practices.

## Principles of the Singleton Pattern

The Singleton Pattern follows a few rules:

1. **Only one object** of the class is created inside the Java program.
2. **Global access point:** there must be a simple way for other parts of the program to get that object.
3. Singletons are useful when we need to share resources like:
    - Logging tools (to record errors and messages).
    - Database connections.
    - Caches (to store temporary data).
    - Thread pools (to manage groups of threads).

Think of a singleton as a city’s power station: the city only needs one, and everyone connects to it.

## How to Create a Singleton in Java

Every singleton has three basic parts:
1. A private constructor – stops other classes from creating new objects.
2. A private static variable – stores the single object.
3. A public static method – gives access to the object.
Different approaches exist, but they all share these three rules.


## Eager Initialization

With eager initialization, the object is created when the class is loaded.
This is simple, but it can waste resources if the object is never used.


Example: a singleton that manages a printer queue.

```java
public class PrinterManager {

    private static final PrinterManager instance = new PrinterManager();

    private PrinterManager() {}

    public static PrinterManager getInstance() {
        return instance;
    }
}
```

This works if the object is light and always useful.
But if the object is heavy (like a database connection), it’s better to wait until it’s really needed.

## Static Block Initialization

Static block initialization is almost the same as eager initialization.
The difference is that we can add exception handling when creating the object.

```java
public class SafePrinterManager {

    private static SafePrinterManager instance;

    private SafePrinterManager() {}

    static {
        try {
            instance = new SafePrinterManager();
        } catch (Exception e) {
            throw new RuntimeException("Error creating singleton");
        }
    }

    public static SafePrinterManager getInstance() {
        return instance;
    }
}
```

Still, the object is created before use, which is not always ideal.

## Lazy Initialization

Lazy initialization means: create the object only when someone asks for it.
This saves memory and resources.


**Example:** a singleton that represents a game scoreboard.

```java
public class ScoreBoard {

    private static ScoreBoard instance;

    private ScoreBoard() {}

    public static ScoreBoard getInstance() {
        if (instance == null) {
            instance = new ScoreBoard();
        }
        return instance;
    }
}
```

**Problem:** This version is not safe in multi-threaded programs. Two threads could create two objects at the same time.

---

-  [Home](./../../README.md)
-  [Design Patterns](./../tutorials.md)