# Serialization and Customization Methods

```
Author: Ter-Petrosyan Hakob
```

---

Serialization is the process of converting an object into a sequence of bytes so that it can be easily saved to a file or sent across a network.
The opposite process is called deserialization — it recreates the object from those bytes.

Serialization allows you to:
- Store an object’s state permanently (for example, saving user data).
- Transfer objects between computers or over the internet.
- Send objects between different parts of a system (like between services in a distributed app).

How Serialization Works in Java

In Java, serialization is built into the standard library through two main classes:

- `ObjectOutputStream` — writes objects to a stream (serialization)
- `ObjectInputStream` — reads objects from a stream (deserialization)

Here’s a basic example:

```java
import java.io.*;

class User implements Serializable {
    String name;
    int score;

    User(String name, int score) {
        this.name = name;
        this.score = score;
    }
}

public class SaveUser {
    public static void main(String[] args) throws Exception {
        User user = new User("Hakob", 100);

        // Serialize
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("user.dat"));
        out.writeObject(user);
        out.close();

        // Deserialize
        ObjectInputStream in = new ObjectInputStream(new FileInputStream("user.dat"));
        User restored = (User) in.readObject();
        in.close();

        System.out.println(restored.name + " - " + restored.score);
    }
}
```

When you run this, Java writes the `User` object to the file `user.dat` and then reads it back.

## Making a Class Serializable

To make a class serializable, you simply implement the `Serializable` interface:

```java
class User implements Serializable { ... }
```

This interface doesn’t have any methods — it’s a marker interface.
It tells Java’s serialization mechanism that objects of this class can be converted into bytes.

### The Role of serialVersionUID

Every serializable class should define a constant called `serialVersionUID`:

```java
@Serial
private static final long serialVersionUID = 1L;
```

It identifies the version of the class.
If you later change the class (for example, add a new field) but keep the same `serialVersionUID`,
Java knows that old serialized objects are still compatible with the new version.

If you don’t define it, Java will generate one automatically — but that can cause problems if the class changes.

### Customizing Serialization

Sometimes, you don’t want Java to automatically serialize every field.

You might want to:
- Skip sensitive information (like passwords),
- Encrypt data before saving,
- Rebuild transient fields after reading.

To do this, you can define special private methods with exact names and signatures.
Java’s serialization system automatically calls these methods if they exist.

Let’s go through them one by one.

#### writeObject(ObjectOutputStream out)

This method lets you control how the object is written to the stream.

```java
class Account implements Serializable {
    String username;
    transient String password; // not serialized

    @Serial
    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject(); // write normal fields
        out.writeObject(encrypt(password)); // write encrypted password manually
    }

    private String encrypt(String input) {
        return new StringBuilder(input).reverse().toString(); // simple example
    }
}
```

- The keyword `transient` means “**don’t serialize this field automatically**.”
- `out.defaultWriteObject()` writes the normal fields.
- You can then write extra data (like encrypted or calculated values).

#### readObject(ObjectInputStream in)

This method is the partner of `writeObject`.
It lets you control how the object is read back from the stream.

```java
@Serial
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
    in.defaultReadObject(); // read normal fields
    password = decrypt((String) in.readObject()); // restore password
}

private String decrypt(String input) {
    return new StringBuilder(input).reverse().toString();
}
```

Java automatically calls `readObject()` during deserialization.
If this method doesn’t exist, it uses the default behavior.

#### readObjectNoData()

This method is called when no data is available for the current class —
for example, if you’re reading an old serialized object from a version where the class didn’t exist yet.

```java
@Serial
private void readObjectNoData() throws ObjectStreamException {
    System.out.println("No data found for this class. Using defaults.");
}
```

You can use it to set default values or handle missing information safely.

#### writeReplace()

This method gives you a chance to replace the object being serialized with another one.

```java
@Serial
private Object writeReplace() throws ObjectStreamException {
    return new AccountProxy(username); // replace with a simpler object
}
```

This is often used when you don’t want to expose sensitive or complex objects during serialization.
The returned object (like `AccountProxy`) is the one that gets serialized instead.

#### readResolve()

This method runs after deserialization.
It allows you to replace the newly created object with another one — useful for singletons or caching.

```java
class Config implements Serializable {
    private static final Config INSTANCE = new Config();

    private Config() {}

    public static Config getInstance() {
        return INSTANCE;
    }

    @Serial
    private Object readResolve() throws ObjectStreamException {
        return INSTANCE; // always return the same singleton
    }
}
```

Without `readResolve()`, deserialization would create a new object, breaking the singleton pattern.

#### Summary of Special Methods

| Method                                | Called When                    | Purpose                       |
| ------------------------------------- | ------------------------------ | ----------------------------- |
| `writeObject(ObjectOutputStream out)` | During serialization           | Custom write logic            |
| `readObject(ObjectInputStream in)`    | During deserialization         | Custom read logic             |
| `readObjectNoData()`                  | When no serialized data exists | Set default values            |
| `writeReplace()`                      | Before serialization           | Replace object before writing |
| `readResolve()`                       | After deserialization          | Replace object after reading  |


## Best Practices

- Always define a `serialVersionUID` in every serializable class.
- Use @`Serial` for all serialization methods and fields — it improves readability and compiler checks.
- Avoid serializing sensitive data directly (encrypt or mark it transient).
- Prefer JSON or other formats for modern systems — built-in serialization is mostly for internal use.


## Serialization (Writing the Object)

```
 ┌─────────────────────────────┐
 │  ObjectOutputStream out     │
 └──────────────┬──────────────┘
                │
                ▼
     ┌───────────────────────────┐
     │  Object is being written  │
     └──────────────┬────────────┘
                    │
                    ▼
       Does the class implement Serializable?
                    │
          ┌─────────┴──────────┐
          │                    │
         YES                  NO
          │                    │
          ▼                    ▼
  ┌────────────────┐     Throw NotSerializableException
  │  Check for     │
  │  writeReplace()│
  └──────┬─────────┘
         │
         ▼
If present → Replace object with returned one
         │
         ▼
 ┌─────────────────────────────┐
 │ Call writeObject(out) if it │
 │ exists, else default write  │
 └──────────────┬──────────────┘
                │
                ▼
 Writes bytes to stream → file/network
                │
                ▼
        Serialization done
```

## Deserialization (Reading the Object)

```
 ┌─────────────────────────────┐
 │  ObjectInputStream in       │
 └──────────────┬──────────────┘
                │
                ▼
   Read class metadata (name, UID, etc.)
                │
                ▼
 Match with loaded class version
                │
       ┌────────┴────────┐
       │                 │
    Match            Mismatch 
       │                 │
       ▼                 ▼
  Continue          InvalidClassException
       │
       ▼
Check for readObject() method
       │
       ▼
 ┌─────────────────────────────┐
 │ Call readObject(in) if it   │
 │ exists, else default read   │
 └──────────────┬──────────────┘
                │
                ▼
If no data for this class → call readObjectNoData()
                │
                ▼
Call readResolve() if defined → replace object
                │
                ▼
       Deserialization done 
```

## Real-World Use Case: Saving and Loading User Profiles

Serialization is not just a theoretical topic — it’s used in many real applications, especially when data must be saved to a file or sent over a network.
Let’s see how the methods like `writeObject`, `readObject`, and `readResolve` work together in a realistic situation.

Imagine a simple game that saves each player’s profile (username, score, and level) so the player can continue late

#### Step 1: Define a Serializable Class

```java
import java.io.*;

class PlayerProfile implements Serializable {
    @Serial
    private static final long serialVersionUID = 1L;

    private String username;
    private int score;
    private int level;

    // transient → don’t save it automatically
    private transient long lastLoginTime;

    PlayerProfile(String username, int score, int level) {
        this.username = username;
        this.score = score;
        this.level = level;
        this.lastLoginTime = System.currentTimeMillis();
    }

    // Custom serialization logic
    @Serial
    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject(); // write non-transient fields
        out.writeLong(System.currentTimeMillis()); // manually write login time
        System.out.println("writeObject() called for " + username);
    }

    @Serial
    private void readObject(ObjectInputStream in)
            throws IOException, ClassNotFoundException {
        in.defaultReadObject(); // read non-transient fields
        this.lastLoginTime = in.readLong(); // manually read login time
        System.out.println("readObject() called for " + username);
    }

    // Fix deserialized object (optional)
    @Serial
    private Object readResolve() throws ObjectStreamException {
        System.out.println("readResolve() called for " + username);
        if (score < 0) score = 0; // Ensure data integrity
        return this;
    }

    @Override
    public String toString() {
        return username + " — Level " + level + " — Score " + score;
    }
}
```

#### Step 2: Saving and Loading Data

```java
public class GameDataDemo {
    public static void main(String[] args) throws Exception {
        PlayerProfile p1 = new PlayerProfile("Hakob", 1500, 5);

        // Save (serialize)
        try (ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("player.dat"))) {
            out.writeObject(p1);
        }

        // Load (deserialize)
        try (ObjectInputStream in = new ObjectInputStream(new FileInputStream("player.dat"))) {
            PlayerProfile p2 = (PlayerProfile) in.readObject();
            System.out.println("Loaded: " + p2);
        }
    }
}
```

####  What Happens Internally

| Stage         | Method Called   | Purpose                                |
| ------------- | --------------- | -------------------------------------- |
| Writing       | `writeObject()` | Customizes what data gets saved        |
| Reading       | `readObject()`  | Recreates object from saved data       |
| After Reading | `readResolve()` | Ensures object is valid or replaces it |

#### Why Use These Methods?

- **Data Security:** You can encrypt or skip sensitive fields (e.g., passwords).
- **Version Compatibility:** You can handle older or newer versions of classes.
- **Data Integrity:** You can fix broken or incomplete data during deserialization.
- **Flexibility:** You can customize the saving/loading process to fit your needs.

#### Key Takeaways

- `writeObject()` and `readObject()` give you full control over serialization.
- `readResolve()` and `writeReplace()` let you replace objects during the process.
- `readObjectNoData()` is used when no serialized data is available for the current class (rare but useful in versioning).
- `@Serial` helps the compiler check that you’re using these methods correctly.
- Always declare a `serialVersionUID` for consistency across versions.

---


- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)