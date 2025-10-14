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

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)