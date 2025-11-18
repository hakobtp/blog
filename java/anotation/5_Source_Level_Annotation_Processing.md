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

### Use the Annotation

Now, let's annotate a simple data class. This is the class we write manually.

```java
package com.example.app;

import com.example.annotations.GenerateBuilder;

@GenerateBuilder
public class GameCharacter {
    private String name;
    private String characterClass;
    private int level;
    private int healthPoints;
    
    // We need a private constructor for the builder to use
    private GameCharacter() {}

    // Getters
    public String getName() { return name; }
    public String getCharacterClass() { return characterClass; }
    public int getLevel() { return level; }
    public int getHealthPoints() { return healthPoints; }

    @Override
    public String toString() {
        return characterClass + " " + name + " (Lvl " + level + ", HP: " + healthPoints + ")";
    }
}
```

### The Generated Code (Our Goal)

Our processor should automatically generate a new file, `GameCharacterBuilder.java`, that looks like this

```java
// THIS FILE IS GENERATED! DO NOT EDIT.
package com.example.app;

public class GameCharacterBuilder {
    private String name;
    private String characterClass;
    private int level;
    private int healthPoints;

    public GameCharacterBuilder withName(String name) {
        this.name = name;
        return this;
    }
    
    // ... other "with" methods for characterClass, level, etc. ...

    public GameCharacter build() {
        GameCharacter obj = new GameCharacter();
        // Here we would need reflection or setters to set private fields.
        // For simplicity, let's assume the processor would generate setters 
        // in the original class if it could, or use a public constructor.
        // A better approach would be generating a constructor that takes a builder.
        return obj; 
    }
}
```

> **NOTE:** A real-world builder would need a way to set private fields, often via a package-private constructor. 
> We'll keep the generation logic simple for this example.

### The Annotation Processor Logic

Here is a simplified version of the processor that generates the builder class.

```java
package com.example.processor;

import com.example.annotations.GenerateBuilder;

import javax.annotation.processing.AbstractProcessor;
import javax.annotation.processing.RoundEnvironment;
import javax.annotation.processing.SupportedAnnotationTypes;
import javax.annotation.processing.SupportedSourceVersion;
import javax.lang.model.SourceVersion;
import javax.lang.model.element.Element;
import javax.lang.model.element.ElementKind;
import javax.lang.model.element.TypeElement;
import javax.lang.model.element.VariableElement;
import javax.tools.JavaFileObject;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.List;
import java.util.Set;
import java.util.stream.Collectors;

@SupportedAnnotationTypes("com.example.annotations.GenerateBuilder")
@SupportedSourceVersion(SourceVersion.RELEASE_17)
public class GenerateBuilderProcessor extends AbstractProcessor {

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        // Find all classes annotated with @GenerateBuilder
        for (Element element : roundEnv.getElementsAnnotatedWith(GenerateBuilder.class)) {
            if (element.getKind() == ElementKind.CLASS) {
                try {
                    generateBuilderFile((TypeElement) element);
                } catch (IOException e) {
                    // Use the Messager to report errors during compilation
                    processingEnv.getMessager().printMessage(javax.tools.Diagnostic.Kind.ERROR, e.toString());
                }
            }
        }
        return true;
    }

    private void generateBuilderFile(TypeElement classElement) throws IOException {
        String packageName = classElement.getQualifiedName().toString().replace("." + classElement.getSimpleName(), "");
        String originalClassName = classElement.getSimpleName().toString();
        String builderClassName = originalClassName + "Builder";

        // Use the Filer to create a new source file
        JavaFileObject builderFile = processingEnv.getFiler().createSourceFile(packageName + "." + builderClassName);

        try (PrintWriter out = new PrintWriter(builderFile.openWriter())) {
            // Write package and class definition
            out.println("package " + packageName + ";");
            out.println("\n// THIS FILE IS GENERATED BY A PROCESSOR! DO NOT EDIT.");
            out.println("public class " + builderClassName + " {");

            // Get all fields of the original class
            List<VariableElement> fields = classElement.getEnclosedElements().stream()
                    .filter(e -> e.getKind() == ElementKind.FIELD)
                    .map(e -> (VariableElement) e)
                    .collect(Collectors.toList());

            // 1. Generate private fields in the builder
            for (VariableElement field : fields) {
                out.println("    private " + field.asType().toString() + " " + field.getSimpleName() + ";");
            }
            out.println();

            // 2. Generate "with" methods
            for (VariableElement field : fields) {
                String fieldName = field.getSimpleName().toString();
                String methodName = "with" + fieldName.substring(0, 1).toUpperCase() + fieldName.substring(1);
                out.println("    public " + builderClassName + " " + methodName + "(" + field.asType().toString() + " " + fieldName + ") {");
                out.println("        this." + fieldName + " = " + fieldName + ";");
                out.println("        return this;");
                out.println("    }");
                out.println();
            }

            // 3. Generate the final build() method
            out.println("    public " + originalClassName + " build() {");
            out.println("        // In a real implementation, you'd use a constructor or setters here.");
            out.println("        // This is a simplified example.");
            out.println("        " + originalClassName + " instance = new " + originalClassName + "();");
            for (VariableElement field : fields) {
                // This part is tricky because fields are private. A real processor
                // might require setters or a package-private constructor.
            }
            out.println("        return instance;");
            out.println("    }");

            out.println("}"); // Close class
        }
    }
}
```

### How to Use Your Processor in a Real Project

Running `javac` from the command line is rare today. Modern projects use build tools like `Maven` or `Gradle`. Here’s how to configure them to use your annotation processor.

#### For Maven (pom.xml)

You need to list the processor as a dependency and configure the `maven-compiler-plugin`.

```xml
<dependencies>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>my-app</artifactId>
        <version>1.0.0</version>
    </dependency>
    <dependency>
        <groupId>com.example</groupId>
        <artifactId>my-processor</artifactId>
        <version>1.0.0</version>
    </dependency>
</dependencies>

<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.8.1</version>
            <configuration>
                <source>17</source>
                <target>17</target>
                <annotationProcessorPaths>
                    <path>
                        <groupId>com.example</groupId>
                        <artifactId>my-processor</artifactId>
                        <version>1.0.0</version>
                    </path>
                </annotationProcessorPaths>
            </configuration>
        </plugin>
    </plugins>
</build>
```

#### For Gradle (build.gradle.kts or build.gradle)

Gradle makes this even easier with its `annotationProcessor` configuration.

```kotlin
// For build.gradle.kts (Kotlin DSL)
dependencies {
    // Add your processor to the annotationProcessor classpath
    annotationProcessor("com.example:my-processor:1.0.0")

    // Your application depends on the annotation definitions
    implementation("com.example:my-annotations:1.0.0")
}
```

## What Annotation Processors Can't Do (And How Lombok Cheats)

Remember the golden rule: processors can't change existing code.

This means you can't use a standard annotation processor to add a method to an existing class. 
Our `@GenerateBuilder` annotation couldn't add a `builder()` method directly inside the `GameCharacter` class. 
It had to create a separate `GameCharacterBuilder.java` file.

So how does Project Lombok work?

You’ve probably seen Lombok's `@Getter`, `@Setter`, and `@Builder` annotations, which magically add methods to your classes without creating new files.

Lombok "cheats." It doesn't use the standard, public annotation processing APIs. Instead, it hooks directly into the compiler's internal machinery to modify the 
`Abstract Syntax Tree (AST)`—the tree-like structure that represents your code—during compilation.

- **Standard Processors:** Add new branches to the forest (new files).
- **Lombok:** Changes the trees in the forest directly (modifies existing classes).

This approach is powerful but is not part of the standard Java specification. It can sometimes be brittle and break with new versions of the Java compiler.


During the compilation process, the Java compiler first parses your `.jav`a source code into an in-memory tree-like structure called an `Abstract Syntax Tree (AST)`. Think of the `AST` as a detailed blueprint of your code, representing every class, method, variable, and statement.

Instead of generating new files, Lombok hooks directly into the compiler's internal APIs to modify this AST in memory.

Here's the step-by-step process for an annotation like `@Getter`:

1. **Parsing:** The compiler reads your `Person.java` file and builds an `AST` for the `Person` class.
2. `Lombok's Turn`: The Lombok annotation processor is triggered. It finds the `AST` node for the `name` field, which is marked with `@Getter`.
3. **AST Modification:** Lombok then directly adds a new set of nodes to the `Person` class's `AST` that represent the `getName()` method. 
    It calculates the method name, return type, and body, and injects them right into the existing blueprint.
4. **Compilation Continues:** The compiler then proceeds to the next stage (code generation), using the modified AST. It sees the `getName()` 
    method as if you had written it yourself and generates the corresponding bytecode for it.

The key takeaway is that your original `.java` file on the disk is never touched. The "magic" happens entirely in the compiler's 
memory before the final `.class` file is written.    

That's an excellent question, and it gets to the heart of what makes Lombok so special and controversial.

You're right, the golden rule for standard annotation processors is that they cannot modify existing source files. Lombok gets around this by not being a "standard" processor. It uses a clever, non-standard technique.

Here’s how it works:

It Modifies the Code In-Memory
During the compilation process, the Java compiler first parses your .java source code into an in-memory tree-like structure called an Abstract Syntax Tree (AST). Think of the AST as a detailed blueprint of your code, representing every class, method, variable, and statement.

Instead of generating new files, Lombok hooks directly into the compiler's internal APIs to modify this AST in memory.

Here's the step-by-step process for an annotation like @Getter:

Parsing: The compiler reads your Person.java file and builds an AST for the Person class.

Lombok's Turn: The Lombok annotation processor is triggered. It finds the AST node for the name field, which is marked with @Getter.

AST Modification: Lombok then directly adds a new set of nodes to the Person class's AST that represent the getName() method. It calculates the method name, return type, and body, and injects them right into the existing blueprint.

Compilation Continues: The compiler then proceeds to the next stage (code generation), using the modified AST. It sees the getName() method as if you had written it yourself and generates the corresponding bytecode for it.

The key takeaway is that your original .java file on the disk is never touched. The "magic" happens entirely in the compiler's memory before the final .class file is written.

The Forest Analogy

- **Standard Annotation Processors:** Can plant new trees in the forest (generate new files). They cannot change the existing trees.
- **Lombok:** Walks up to an existing tree and grafts new branches onto it (modifies the AST of an existing class).

This approach is extremely powerful but relies on internal, undocumented compiler APIs, which is why Lombok can sometimes have compatibility issues with new Java versions or other tools that also manipulate the compilation process.

## Summary

Annotation processing is a key feature for modern Java development, powering many popular libraries for dependency injection (Dagger, Hilt), object mapping (MapStruct), and more.

Let's recap the key ideas:
- **Automate Your Work**: Processors let you write code that generates other code, saving you from repetitive tasks.
- **Compile-Time, Not Runtime:** Everything happens during compilation, so there is no performance penalty when your program runs.
- **The Golden Rule:** They can only create new files; they can never modify existing ones.
- **Language Model API:** You use this API (with `TypeElement`, `ExecutableElement`, etc.) to analyze source code before it's compiled.
- **Build Tool Integration:** Real-world projects use Maven or Gradle to manage and run annotation processors automatically.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Using Annotations at Runtime](./4_Using_Annotations_at_Runtime.md)