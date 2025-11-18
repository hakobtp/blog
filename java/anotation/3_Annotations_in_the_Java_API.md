# Annotations in the Java API

```
Author: Ter-Petrosyan Hakob
```

---

Java comes with several built-in annotations that you can use right away — you don’t have to define them yourself.

They are stored in packages such as:
- `java.lang`
- `java.lang.annotation`
- `javax.annotation`

Some of these are meta-annotations, meaning they describe how other annotations behave.
The rest are regular annotations — you can apply them directly to your classes, methods, or variables.

Let’s look at the most important ones that every Java developer should know.

### Common Annotations You Use in Code

| Annotation             | Used On                            | What It Does                                                                  |
| ---------------------- | ---------------------------------- | ----------------------------------------------------------------------------- |
| `@Override`            | Methods                            | Makes sure a method really overrides one from the superclass.                 |
| `@Serial`              | Methods and fields                  | Confirms that a method or field is used for Java serialization.              |
| `@Deprecated`          | All declarations                   | Marks something as old or unsafe to use.                                      |
| `@SuppressWarnings`    | All declarations (except packages) | Hides specific compiler warnings.                                             |
| `@SafeVarargs`         | Methods and constructors           | Confirms that using varargs (variable arguments) is safe.                     |
| `@FunctionalInterface` | Interfaces                         | Tells that this interface is functional (it has exactly one abstract method). |
| `@Generated`           | All declarations                   | Shows that a piece of code was created by a tool or generator.                |

#### @Override

When you write a method that overrides another one from a superclass, it’s a good idea to use `@Override`.

```java
class Animal {
    void makeSound() {}
}

class Dog extends Animal {
    @Override
    void makeSound() {
        System.out.println("Bark!");
    }
}
```

If you accidentally misspell the method name or use the wrong parameters, the compiler will warn you.
That’s why this annotation prevents subtle bugs.

#### @Serial

`@Serial` is an annotation introduced in Java 14 to make serialization-related methods clearer and safer.
It helps the compiler confirm that special methods in a class are really meant for Java object serialization.

Serialization means turning an object into a stream of bytes so it can be saved to a file or sent over a network.

Before Java 14, developers relied only on method names like `writeObject`, `readObject`, or `readResolve` to tell the JVM that these were special serialization methods.
But the compiler couldn’t verify that you typed the method name or signature correctly — a tiny mistake meant the method would be ignored. `@Serial` fixes that problem by giving the compiler a clear sign.

```java
@Serial
private void writeObject(ObjectOutputStream out) throws IOException {
    out.defaultWriteObject();
}
```

If you now accidentally use the wrong method name, like `writeObjct`,
the compiler will show an error — it knows `@Serial` must go only on valid serialization methods.



- `@Serial` applies to:
    - **Methods** — like `writeObject`, `readObject`, `readObjectNoData`, `writeReplace`, `readResolve`.
    - `Fields` — like `serialVersionUID`, which gives a class version number for serialization.

#### @Deprecated

Use this annotation when a method or class should no longer be used — maybe because a newer version exists.

```java
@Deprecated
void oldMethod() {
    System.out.println("This method is outdated.");
}
```

When other developers call it, the compiler shows a warning message.
You can also give more details using JavaDoc’s `@deprecated` tag.

#### @SuppressWarning

Sometimes the compiler shows warnings that you already know about and can safely ignore.
You can use this annotation to hide them.

```java
@SuppressWarnings("unchecked")
List rawList = new ArrayList();
```

However, this should be used carefully — suppressing warnings might hide real problems in your code.

#### @SafeVarargs

This annotation tells the compiler that it’s safe to use a varargs parameter (an argument list with `...`).
It prevents warnings about “heap pollution” — a problem that can happen when mixing generics and varargs.

```java
@SafeVarargs
static <T> void printAll(T... items) {
    for (T item : items) {
        System.out.println(item);
    }
}
```

You can use it only on `final` or `static` methods, and on `constructors`.

#### @FunctionalInterface

This one marks an interface as a functional interface, which means it must have only one abstract method.
Functional interfaces are used with lambda expressions.

```java
@FunctionalInterface
interface Calculator {
    int compute(int a, int b);
}
```

If you add a second abstract method, the compiler will report an error.

Example of use:
```java
Calculator add = (x, y) -> x + y;
System.out.println(add.compute(3, 4)); // 7
```

#### @Generated

When you see this annotation, it usually means a tool created the code, not a human.
For example, frameworks like Lombok or MapStruct often mark generated methods or classes with it.
This helps other tools know not to modify or analyze that code

### Meta-Annotations

Meta-annotations are special because they describe how other annotations work.
Here are the most important ones:

| Meta-Annotation | Used On          | What It Does                                                                      |
| --------------- | ---------------- | --------------------------------------------------------------------------------- |
| `@Target`       | Annotation types | Defines **where** the annotation can be used (e.g., on methods, classes, fields). |
| `@Retention`    | Annotation types | Defines **how long** the annotation is kept (in source, class, or runtime).       |
| `@Documented`   | Annotation types | Makes sure the annotation appears in the generated documentation.                 |
| `@Inherited`    | Annotation types | Lets subclasses automatically **inherit** an annotation from their superclass.    |
| `@Repeatable`   | Annotation types | Allows an annotation to be used **more than once** on the same element.           |


---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Declaring an Annotation](./2_Declaring_an_Annotation.md)
- [Using Annotations at Runtime](./4_Using_Annotations_at_Runtime.md)