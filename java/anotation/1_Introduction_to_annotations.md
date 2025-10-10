# Introduction to Annotations

```
Author: Ter-Petrosyan Hakob
```

---

Annotations are small tags or notes that you add to your Java code. They give extra information to special tools or frameworks. 
These tools can look at your code directly or read the compiled class files that include those annotations.

Annotations themselves don’t change how your program runs. The Java compiler still creates the same bytecode, even if you remove all annotations.
Their main purpose is to help other tools understand your code or automate certain tasks.

## How Annotations Are Used

To actually use annotations, you need a tool or library that knows what those annotations mean. 
After adding the right annotations to your code, that tool can process them and perform useful actions.

Here are a few common examples:

- Testing frameworks like JUnit use annotations to mark methods that should be run as tests. 
    For example, if you mark a method with `@Test`, JUnit knows to run it automatically when you start your test suite.

- Persistence frameworks like Jakarta Persistence (JPA) use annotations to connect Java classes to database tables. 
    For instance, you can mark a class with `@Entity` to show that it represents a table, and mark fields with `@Column` 
    to describe how they match database columns. This way, you don’t have to write SQL queries yourself.

There are many other libraries and frameworks that use annotations to make your code shorter, cleaner, and easier to maintain.

## Using Annotations

Let’s look at a simple example of how an annotation works in Java:

```java
class MathTest {
   @Test
   void checkSum() {
      // test logic here
   }
}
```

Here, the annotation `@Test` is placed before the `checkSum` method.
In Java, annotations are written like modifiers (for example, public or static), but they always start with the `@` symbol.

On their own, annotations do nothing. They only become useful when a tool or framework looks for them.
For example, when you run JUnit, it automatically finds every method that has the `@Test` annotation and runs those methods as tests.

## Annotation Elements

Annotations can also have extra information, called elements.
These elements are written as key–value pairs, like this:

```java
@RepeatTest(times = 5, stopOnFail = true)
```

Each annotation defines what elements are allowed and what types they can have.
For example, an element’s value can be:

- A number (like `5` or `3.14`)
- A string (`"Hello"`)
- A class (`MyClass.class`)
- An enum value (`Level.HIGH`)
- Another annotation
- Or an array (a list of any of the above types)

Here’s an example of a custom annotation that uses many of these types:

```java
@IssueReport(
   id = 1024,
   fixed = false,
   assignedTo = "Lena",
   relatedClass = MathTest.class,
   priority = IssueReport.Priority.HIGH,
   reference = @Ref(id = 2048),
   reporters = {"Lena", "Tom"}
)
```

This annotation doesn’t belong to any real library — it’s just an example showing how different element types can appear together.

## Rules for Annotation Elements

There are some important rules to remember:

1. **All values must be known at compile time**. <br>
    You cannot use variables or runtime results in annotation values.
2. **You cannot use `null`.**<br>
    If you need an “empty” value, use an empty string (`""`) or empty array (`{}`) instead.    
3. Arrays with one element can drop the braces.
    ```java
    @IssueReport(reporters = "Lena")   // same as {"Lena"}
    ```
4. **Default values are allowed.**<br>
    If an annotation defines a default value for an element, you can skip that element when writing it.
    For example, imagine the `@RepeatTest` annotation has a default `stopOnFail = false`.
    Then both of these are equal:     
   
    ```java
    @RepeatTest(times = 5)
    @RepeatTest(times = 5, stopOnFail = false)
    ```
5. **The special element name `value`.** <br>
    If an annotation has only one element named `value`, you can omit the name:    

## Working with Multiple and Repeated Annotations

You can use more than one annotation on the same item in Java.

```java
@Test
@Tag("slow")
void testFileUpload() { }
```

Here, both `@Test` and `@Tag("slow")` are attached to the same method.

If an annotation type is declared repeatable, you can use it more than once:

```java
@Tag("network")
@Tag("important")
void testFileUpload() { }
```

Both `@Tag` annotations will be stored and processed together by the framework that supports them.

## Annotating Different Parts of Your Code

So far, we’ve only seen annotations on methods, but you can also add them to many other parts of your program.

You can place annotations on:

- **Classes and Interfaces**
- **Methods and constructors**
- **Fields and record components**
- **Local variables** (including those inside `for` loops or `try-with-resources`)
- **Parameters** (method parameters or `catch` blocks)
- **Type parameters** (for generic types)
- **Packages and modules**

Here are a few examples:

```java
@Entity
public class User { }

@SuppressWarnings("unchecked")
List<User> users = new ArrayList<>();

public User findUser(@Param("id") String userId) { ... }

public record Rectangle(
   @PrintAsJson Point topLeft,
   int width,
   int height
) { }
```

### Annotating Type Parameters

You can even annotate a type parameter in a generic class or method:

```java
public class Cache<@NotNull V> { ... }
```

This means that the generic type `V` should never be `null`.

### Annotating Packages

To annotate an entire `package`, you create a special file called `package-info.java`.
That file contains only the `package` statement and the annotations before it.

```java
/**
 * Package-level documentation
 */
@Version("1.2.0")
package com.example.myapp;
import com.example.annotations.Version;
```

Notice that the import statement comes after the `package` declaration.
Annotations placed on local variables or packages are not stored in the compiled class file, so they can only be used by source-level tools, not at runtime.

### Annotating for loop

Starting with Java 8, the language allows annotations on local variables, including those declared inside a for loop

This means you can write code like this:

```java
for (@MyAnnotation int i = 0; i < 5; i++) {
    System.out.println(i);
}
```

Here, `@MyAnnotation` is applied directly to the loop variable `i`.
This works because `i` is a local variable declaration, and Java 8 expanded the annotation system to support such cases.

> To make this possible, your annotation type must target `ElementType.LOCAL_VARIABLE` or `ElementType.TYPE_USE`.

You can only annotate the variable declaration inside the loop — not the loop’s control expressions.

In other words, these are not allowed:

```java
for (int i = 0; @MyAnnotation i < 5; i++) // Invalid
for (int i = 0; i < 5; @MyAnnotation i++) // Invalid
```

Annotations apply only to **declarations**, not to **statements** or **expressions**.

You can also annotate variables in enhanced for loops (the ones used with collections or arrays):

```java
for (@NonNull String name : names) {
    System.out.println(name);
}
```

This is particularly useful for static analysis tools such as the Checker Framework or SpotBugs, 
which use annotations like `@NonNull` or `@Nullable` to verify code correctness.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)