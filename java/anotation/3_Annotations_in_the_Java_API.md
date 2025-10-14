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

| Annotation             | Used On                            | What It Does                                                                  |
| ---------------------- | ---------------------------------- | ----------------------------------------------------------------------------- |
| `@Override`            | Methods                            | Makes sure a method really overrides one from the superclass.                 |
| `@Deprecated`          | All declarations                   | Marks something as old or unsafe to use.                                      |
| `@SuppressWarnings`    | All declarations (except packages) | Hides specific compiler warnings.                                             |
| `@SafeVarargs`         | Methods and constructors           | Confirms that using varargs (variable arguments) is safe.                     |
| `@FunctionalInterface` | Interfaces                         | Tells that this interface is functional (it has exactly one abstract method). |
| `@Generated`           | All declarations                   | Shows that a piece of code was created by a tool or generator.                |

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Declaring an Annotation](./2_Declaring_an_Annotation.md)