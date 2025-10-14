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

```java
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

## Example of Full Flow with Custom Methods

Here’s how everything fits together in a single example:

```java
import java.io.*;

class Account implements Serializable {
    String username;
    transient String password;

    Account(String username, String password) {
        this.username = username;
        this.password = password;
    }

    // Step 1: Replace object before writing
    @Serial
    private Object writeReplace() throws ObjectStreamException {
        System.out.println("writeReplace() called");
        return this;
    }

    // Step 2: Customize writing
    @Serial
    private void writeObject(ObjectOutputStream out) throws IOException {
        System.out.println("writeObject() called");
        out.defaultWriteObject();
        out.writeObject(encrypt(password));
    }

    // Step 3: Customize reading
    @Serial
    private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
        System.out.println("readObject() called");
        in.defaultReadObject();
        password = decrypt((String) in.readObject());
    }

    // Step 4: Replace object after reading
    @Serial
    private Object readResolve() throws ObjectStreamException {
        System.out.println("readResolve() called");
        return this;
    }

    // Step 5: Handle case when no data
    @Serial
    private void readObjectNoData() throws ObjectStreamException {
        System.out.println("readObjectNoData() called");
    }

    private String encrypt(String s) { return new StringBuilder(s).reverse().toString(); }
    private String decrypt(String s) { return new StringBuilder(s).reverse().toString(); }
}
```

Output flow (simplified):

```
writeReplace() called
writeObject() called
readObject() called
readResolve() called
```

Summary Diagram (Simplified)
```
Serialization:
writeReplace() → writeObject() → (data written)

Deserialization:
(read data) → readObject() / readObjectNoData() → readResolve()
```

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)