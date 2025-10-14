# Source-Level Annotation Processing

```
Author: Ter-Petrosyan Hakob
```

---

In the previous section, you learned how to use reflection to read annotations while a program is running.
Now, let’s see another powerful use of annotations — processing them directly in source code before the program even runs.

This kind of processing can help you automatically create extra source files, configuration files, documentation, or any other resource you need. In short, you can write code that writes more code for you.

## Annotation Processors — How They Work

Annotation processing is built into the Java compiler.
When you compile a program, the compiler can look for annotations and call special tools called annotation processors.

You can run them like this:

```bash
javac -processor ProcessorClassName1,ProcessorClassName2 MySourceFile.java
```

Each processor checks for specific annotations and acts only on the ones it cares about.
If a processor creates new source files, the compiler will process those new files as well. This continues until there are no more files to generate.

> **Important rule:** Annotation processors cannot modify existing files — they can only create new ones.

### A Custom Processor

An annotation processor is a Java class that implements the `Processor` interface, usually by extending the `AbstractProcessor` class.
You must tell the compiler which annotations your processor supports.

For example:
```java
@SupportedAnnotationTypes("annotations.AutoLog")
@SupportedSourceVersion(SourceVersion.RELEASE_21)
public class AutoLogProcessor extends AbstractProcessor {
    public boolean process(Set<? extends TypeElement> annotations, 
                           RoundEnvironment roundEnv) {
        // Processor logic goes here
        return true;
    }
}
```

This processor looks for the annotation `@AutoLog`.
You could also use wildcards like `"com.example.*"` (to include all annotations in a package) or `"*"` (to include all annotations everywhere).

The `process` method runs once per round of compilation.

It receives:
- A set of all found annotations, and
- A `RoundEnvironment` object that provides information about the elements (classes, methods, etc.) found in that round.

### The Language Model API

When analyzing annotations in source code, you use the Language Model API, not reflection.

Difference from Reflection:

- The reflection API shows how classes and methods look inside the Java Virtual Machine after compilation.
- The language model API shows how they look in source code, following Java’s language rules.

The compiler represents your program as a tree made of elements such as:

- `TypeElement` → classes and interfaces
- `VariableElement` → fields and variables
- `ExecutableElement` → methods and constructors

These elements are similar to `Class`, `Field`, and `Method` objects from reflection, but they work before the code is turned into bytecode.

You can use methods like:

```java
Set<? extends Element> getElementsAnnotatedWith(Class<? extends Annotation> a)
Set<? extends Element> getElementsAnnotatedWithAny(Set<Class<? extends Annotation>> annotations)
```

to find all elements that have certain annotations.
The second method is useful when one element has multiple annotations of the same type (called repeated annotations).

You can also call:
```java
A getAnnotation(Class<A> annotationType)
A[] getAnnotationsByType(Class<A> annotationType)
```
to get a specific annotation or all repeated ones.

- If you need a class’s fields or methods, call `getEnclosedElements()` on its `TypeElement`.
- To get names, use `getSimpleName()` or `getQualifiedName()` — they return a `Name` object that you can convert to a string.

## Using Annotations to Generate Source Code

Let’s try a practical example.

Suppose we want to automatically create `toString()` methods for certain classes.
Because annotation processors can’t edit existing files, we’ll generate a new helper class instead.

We define our annotation: 
```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.SOURCE)
public @interface AutoString {
    boolean includeName() default true;
}
```

Then, use it in your code:

```java
@AutoString
public class Book {
    private String title;
    private String author;

    @AutoString(includeName = false)
    public String getTitle() { return title; }

    @AutoString
    public String getAuthor() { return author; }
}
```

The processor will generate a new class, for example `AutoStrings.java`, containing methods like this:

```java
public class AutoStrings {
    public static String toString(Book obj) {
        var sb = new StringBuilder();
        sb.append("Book[");
        sb.append("title=").append(obj.getTitle()).append(", ");
        sb.append("author=").append(obj.getAuthor());
        sb.append("]");
        return sb.toString();
    }

    public static String toString(Object obj) {
        return java.util.Objects.toString(obj);
    }
}
```

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Using Annotations at Runtime](./4_Using_Annotations_at_Runtime.md)