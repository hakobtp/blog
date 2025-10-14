# Declaring an Annotation

```
Author: Ter-Petrosyan Hakob
```

---

In Java, annotations are a special kind of code marker that gives extra information to the compiler or to tools that process your code.
To create your own annotation, you use an annotation interface. This is done with the `@interface` keyword.

Every annotation starts with an `@interface` declaration.
Inside the interface, you define methods that represent the elements (or properties) of your annotation.

For example, let’s define an annotation called `@RunTimes` that marks how many times a test should run:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RunTimes {
    int count();
}
```

Here’s what happens:

- The annotation `@RunTimes` can only be used on methods (because of `@Target(ElementType.METHOD)`).
- It can be accessed while the program is running (because of `@Retention(RetentionPolicy.RUNTIME)`).
- It has one element, `count`, which tells how many times the test should run.

Later, a testing tool could use reflection to read this annotation and call the test method the given number of times.

## Understanding Annotation Elements

In annotation interfaces:

- Each method defines one element.
- Methods cannot take parameters, cannot throw exceptions, and cannot be generic.
- You can give elements default values, so programmers don’t always have to set them manually.

Example with defaults:

```java
public @interface TaskInfo {
    String author() default "Unknown";
    int priority() default 1;
}
```

If someone writes `@TaskInfo`, Java will automatically use `"Unknown"` as the author and `1` as the priority unless other values are provided.

## Meta-Annotations: Annotations About Annotations

Annotations like `@Target` and `@Retention` are called meta-annotations — they describe how other annotations behave.

### @Target

This tells where your annotation can be placed.

Example:

```java
@Target({ElementType.TYPE, ElementType.FIELD})
public @interface JsonSerializable {}
```

Here, `@JsonSerializable` can be applied to classes and fields.
If you try to use it on a method, the compiler will show an error.

Common element types you can use with `@Target` include:

| ElementType      | Used On                           |
| ---------------- | --------------------------------- |
| TYPE             | Classes, interfaces, enums        |
| FIELD            | Fields (including enum constants) |
| METHOD           | Methods                           |
| PARAMETER        | Method or constructor parameters  |
| CONSTRUCTOR      | Constructors                      |
| PACKAGE          | Packages                          |
| RECORD_COMPONENT | Record components                 |
| TYPE_PARAMETER   | Type parameters                   |
| TYPE_USE         | Any use of a type (Can be used anywhere a type is written (variable types, casts, generics, etc.).)                 |


### @Retention

This defines how long the annotation stays available.

- `RetentionPolicy.SOURCE` → only in source code, not in `.class` files.
- `RetentionPolicy.CLASS` → stored in `.class` files but ignored at runtime (default).
- `RetentionPolicy.RUNTIME` → available while the program runs (used with reflection).

For example:
```java
@Retention(RetentionPolicy.SOURCE)
public @interface CompileNote {}
```

This one is only visible to tools that process the source code — it disappears after compilation.

##Default Values and Computation

Defaults aren’t stored inside annotations themselves.
When you change a default and recompile the annotation interface, any old class using it will automatically get the new default value the next time it’s loaded.

## Technical Notes

All annotation interfaces automatically extend the `java.lang.annotation.Annotation` interface.
You don’t implement annotation interfaces yourself — the Java compiler and runtime create proxy objects when needed.

- `java.lang.annotation.Annotation`
   - `Class<? extends Annotation> annotationType()` — returns the `Class` object that represents the annotation interface of this annotation. 
      Note that if you call `getClass` on an annotation object, it gives the real class, not the interface.
   - `boolean equals(Object other)` — returns `true` if the other object uses the same annotation interface as this one, and if all their element values are the same.
   - `int hashCode()` — returns a hash code that matches the equals method. It is based on the annotation interface name and the element values.     
   - `String toString()` — returns a string form that includes the annotation interface name and the names and values of its elements.


```java
package com.htp;


import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;

@Retention(RetentionPolicy.RUNTIME)
@interface CustomAnnotation {
    String author();
    int version();
}

@CustomAnnotation(author = "Hakob", version = 1)
class MyClass {}


public class AppMain {
    public static void main(String[] args) {
        // Get the annotation from MyClass
        CustomAnnotation info = MyClass.class.getAnnotation(CustomAnnotation.class);

        // 1. annotationType()
        System.out.println("Annotation type: " + info.annotationType());

        // 2. equals()
        CustomAnnotation sameInfo = MyClass.class.getAnnotation(CustomAnnotation.class);
        System.out.println("Equals: " + info.equals(sameInfo));

        // 3. hashCode()
        System.out.println("Hash code: " + info.hashCode());

        // 4. toString()
        System.out.println("To string: " + info.toString());
    }
}

```

Output:

```java
Annotation type: interface com.htp.CustomAnnotation
Equals: true
Hash code: -740212903
To string: @com.htp.CustomAnnotation(author="Hakob", version=1)
```

## Summary

- Annotations in Java are defined using the `@interface` keyword.
- Each method in the interface defines an element of the annotation.
- `@Target` decides where the annotation can be used.
- `@Retention` decides how long it stays available.
- You can set default values for elements to make your annotation easier to use.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Introduction to Annotations](./1_Introduction_to_annotations.md)
- [Annotations in the Java API](./3_Annotations_in_the_Java_API.md)