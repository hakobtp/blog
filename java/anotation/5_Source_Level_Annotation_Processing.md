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

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Using Annotations at Runtime](./4_Using_Annotations_at_Runtime.md)