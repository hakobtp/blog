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

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)