# Source-Level Annotation Processing

```
Author: Ter-Petrosyan Hakob
```

---

Tired of writing the same repetitive code? Think about `toString()` methods, 
builders, or mappers. It's boring, and it's easy to make mistakes. What if you could teach the Java compiler to write that code for you?

In the previous article, we explored how to use reflection to inspect annotations while a program is running. Now, 
let's explore an even more powerful technique: source-level annotation processing. This allows us to analyze annotations 
and generate new files before the program even compiles.

With this superpower, you can automatically generate:

- Boilerplate code (builders, mappers, etc.)
- Configuration files (XML, JSON, properties)
- API documentation
- And much more!

Get ready to make the compiler your personal assistant. 

## How Do Annotation Processors Work?

Annotation processing is a special feature built directly into the Java compiler (`javac`). Think of 
the compiler as a factory assembly line. As your source code (`.java` files) moves down the line, the compiler can call special tools called annotation processors.

These processors are like little robots on the assembly line. They scan your code for specific annotations they've been trained to find.

Here’s the process:

1. **Scan:** The compiler starts compiling your code and finds annotations like `@MyCustomAnnotation`.
2. **Activate:** It activates any registered processors that handle those annotations.
3. **Process:** The processor analyzes the annotated code (e.g., a class or method) and generates new source files.
4. **Loop:** If new source files were created, the compiler runs another "round" of compilation to process those new files. This continues until no new files are generated.

> **NOTE:** Annotation processors can only generate new files. They can never, ever modify existing source code

## Building Your First Annotation Processor

An annotation processor is just a Java class. It needs to implement the Processor interface, but we usually make our lives easier by extending the AbstractProcessor class.

To make it work, you must tell the compiler two things:

1. Which annotations should this processor handle?
2. Which Java version is it compatible with?

Here’s the basic structure:

```java
// Tell the compiler this processor handles our @GenerateBuilder annotation
@SupportedAnnotationTypes("com.example.annotations.GenerateBuilder") 
// Tell the compiler this processor works with Java 21+ features
@SupportedSourceVersion(SourceVersion.RELEASE_21) 
public class GenerateBuilderProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        // All the magic happens here!
        // Return 'true' to signal that you've handled the annotations.
        return true; 
    }
}
```

- `@SupportedAnnotationTypes`**:** 
    - You can specify the full path to an annotation, 
    - use a wildcard like `"com.example.*"`, 
    - or even `"*"` to inspect all annotations (though this is less efficient).
- `@SupportedSourceVersion`**:** This ensures your processor doesn't fail on newer Java syntax.
- `process()`**:** This is the main method where all your logic goes. It runs during each compilation round.


## Source Code vs. Runtime: The Language Model API

When a processor analyzes your code, the code hasn't been compiled into bytecode yet. This means we cannot use reflection. 
Reflection works on loaded classes in the JVM (at runtime), but we are working with raw source code (at compile-time).

So, how do we analyze the code? We use the Language Model API.

Here’s a simple analogy
- Reflection is like inspecting a fully built car in a showroom. You can see its final parts and features.
- The Language Model API is like looking at the car's original blueprints. You see the design, the components, and how everything is supposed to connect before it's built.

The compiler represents your source code as a tree of `Element` objects. The most common ones are:

- `TypeElement`**:** Represents a class or interface (like `Class` in reflection).
- `ExecutableElement`**:** Represents a method or constructor (like `Method` or `Constructor` in reflection).
- `VariableElement`: Represents a field, parameter, or local variable (like `Field` in reflection).

You can use the `RoundEnvironment` object inside your process method to find all elements annotated with your target annotation:

```java
// Inside the process() method
Set<? extends Element> annotatedElements = roundEnv.getElementsAnnotatedWith(GenerateBuilder.class);
```

## Practical Example: Generating a Fluent Builder

Let's create a real-world example. Manually writing a Builder pattern for a class is tedious. We can automate it!

**Our goal**: Annotate a simple class and have the processor generate a complete Builder class for it.

### Define the Annotation

First, we need an annotation. Its only job is to mark the classes we want to process.
`@Retention(RetentionPolicy.SOURCE)` is crucial—it tells the compiler to discard the annotation after compilation because it's no longer needed.

```java
package com.example.annotations;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE) // This annotation can only be used on classes
@Retention(RetentionPolicy.SOURCE) // Available only during compilation
public @interface GenerateBuilder {
}
```


---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Using Annotations at Runtime](./4_Using_Annotations_at_Runtime.md)