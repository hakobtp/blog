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

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Using Annotations at Runtime](./4_Using_Annotations_at_Runtime.md)