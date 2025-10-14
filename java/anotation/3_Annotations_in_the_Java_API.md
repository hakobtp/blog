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
| `@Deprecated`          | All declarations                   | Marks something as old or unsafe to use.                                      |
| `@SuppressWarnings`    | All declarations (except packages) | Hides specific compiler warnings.                                             |
| `@SafeVarargs`         | Methods and constructors           | Confirms that using varargs (variable arguments) is safe.                     |
| `@FunctionalInterface` | Interfaces                         | Tells that this interface is functional (it has exactly one abstract method). |
| `@Generated`           | All declarations                   | Shows that a piece of code was created by a tool or generator.                |

--- 

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

---

Use this annotation when a method or class should no longer be used — maybe because a newer version exists.

```java
@Deprecated
void oldMethod() {
    System.out.println("This method is outdated.");
}
```

When other developers call it, the compiler shows a warning message.
You can also give more details using JavaDoc’s `@deprecated` tag.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Declaring an Annotation](./2_Declaring_an_Annotation.md)