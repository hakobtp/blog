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

## Thread-Safe Singleton

To fix the multi-thread problem, we can make the method synchronized.

```java
public class ThreadSafeScoreBoard {

    private static ThreadSafeScoreBoard instance;

    private ThreadSafeScoreBoard() {}

    public static synchronized ThreadSafeScoreBoard getInstance() {
        if (instance == null) {
            instance = new ThreadSafeScoreBoard();
        }
        return instance;
    }
}
```

This works, but it is slower because synchronization adds overhead.
A common solution is double-checked locking:

```java
public static ThreadSafeScoreBoard getInstance() {
    if (instance == null) {
        synchronized (ThreadSafeScoreBoard.class) {
            if (instance == null) {
                instance = new ThreadSafeScoreBoard();
            }
        }
    }
    return instance;
}
```

This way, we only lock once, when the object is first created.


## Bill Pugh Singleton

A very clean method uses a static inner helper class.

```java
public class ConfigManager {

    private ConfigManager() {}

    private static class Holder {
        private static final ConfigManager INSTANCE = new ConfigManager();
    }

    public static ConfigManager getInstance() {
        return Holder.INSTANCE;
    }
}
```

- The helper class is loaded only when needed.
- This is fast, simple, and doesn’t need synchronization.
- This is the most popular singleton approach in modern Java.

## Using Reflection to destroy Singleton Pattern

Reflection can be used to destroy all the previous singleton implementation approaches. Here is an example class:

```java
public class ReflectionSingletonTest {

    public static void main(String[] args) {
        EagerInitializedSingleton instanceOne = EagerInitializedSingleton.getInstance();
        EagerInitializedSingleton instanceTwo = null;
        try {
            Constructor[] constructors = EagerInitializedSingleton.class.getDeclaredConstructors();
            for (Constructor constructor : constructors) {
                // This code will destroy the singleton pattern
                constructor.setAccessible(true);
                instanceTwo = (EagerInitializedSingleton) constructor.newInstance();
                break;
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        System.out.println(instanceOne.hashCode());
        System.out.println(instanceTwo.hashCode());
    }

}
```

When you run the preceding test class, you will notice that hashCode of both instances is not the same which destroys the singleton pattern. Reflection is very powerful and used in a lot of frameworks like Spring and Hibernate.


## Enum Singleton

To overcome this situation with Reflection, Joshua Bloch suggests the use of enum to implement the singleton design pattern as Java ensures that any enum value is instantiated only once in a Java program. Since Java Enum values are globally accessible, so is the singleton. The drawback is that the enum type is somewhat inflexible (for example, it does not allow lazy initialization).

```java
public enum EnumSingleton {

    INSTANCE;

    public static void doSomething() {
        // do something
    }
}
```

## Serialization and Singleton

Sometimes in distributed systems, we need to implement Serializable interface in the singleton class so that we can store its state in the file system and retrieve it at a later point in time. Here is a small singleton class that implements Serializable interface also:

```java
public class SerializedSingleton implements Serializable {

    private static final long serialVersionUID = -7604766932017737115L;

    private SerializedSingleton(){}

    private static class SingletonHelper {
        private static final SerializedSingleton instance = new SerializedSingleton();
    }

    public static SerializedSingleton getInstance() {
        return SingletonHelper.instance;
    }

}
```

The problem with serialized singleton class is that whenever we deserialize it, it will create a new instance of the class. Here is an example:

```java
public class SingletonSerializedTest {

    public static void main(String[] args) throws FileNotFoundException, IOException, ClassNotFoundException {
        SerializedSingleton instanceOne = SerializedSingleton.getInstance();
        ObjectOutput out = new ObjectOutputStream(new FileOutputStream("filename.ser"));
        out.writeObject(instanceOne);
        out.close();

        // deserialize from file to object
        ObjectInput in = new ObjectInputStream(new FileInputStream("filename.ser"));
        SerializedSingleton instanceTwo = (SerializedSingleton) in.readObject();
        in.close();

        System.out.println("instanceOne hashCode="+instanceOne.hashCode());
        System.out.println("instanceTwo hashCode="+instanceTwo.hashCode());
    }
}
```

That code produces this output:

```
Output
instanceOne hashCode=2011117821
instanceTwo hashCode=109647522
```

So it destroys the singleton pattern. To overcome this scenario, all we need to do is provide the implementation of `readResolve()` method.

```java
protected Object readResolve() {
    return getInstance();
}
```

After this, you will notice that hashCode of both instances is the same in the test program.

## Advantages of the Singleton Pattern

- **Only one object exists:** You can be sure that only one instance of the class is ever created. For example, the whole program can share the same log manager.
- **Easy global access:** Other parts of the program can get the same instance from a single, well-known place. It’s like having one customer service number that everyone dials.
- **Lazy loading possible:** The object can be created only when it is first needed, which saves memory and resources. For example, a database connector is opened only when someone runs a query.

## Disadvantages of the Singleton Pattern

- **Breaks the Single Responsibility Principle:** The class is responsible for both creating itself and controlling its only instance. This mixes two jobs into one.
- **Can hide weak design:** Sometimes developers use a singleton instead of designing proper relationships between classes. This makes parts of the program know too 
    much about each other. For example, if every class calls the same GlobalSettings object, they may become tightly connected.
- **Issues in multithreaded programs:** If not written carefully, two or more threads could create separate objects at the same time. This breaks the singleton rule.
- **Hard to test**
    - In unit tests, we often want to replace real objects with mock objects. Since the singleton has a private constructor and static methods, it’s 
        difficult to replace or extend. This makes automated testing more complex.
    - use dependency injection or avoid singleton if testability is very important.

---

- [Home](./../../README.md)
- [Design Patterns](./../tutorials.md)
- [Factory](./1_Factory.md)