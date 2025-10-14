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

## A Custom Processor

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

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Using Annotations at Runtime](./4_Using_Annotations_at_Runtime.md)