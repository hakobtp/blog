# The Optional Type

An `Optional<T>` is a box (or wrapper) that can either hold a value of type `T` or be empty.
- If it has a value → we say the value is present.
- If it has no value → it is empty.

`Optional` is safer than using `null` because it makes you handle the “maybe empty” case in a clear way. But it is only safer if you use it correctly.

## Getting a Value from Optional

The main idea of `Optional` is: don’t take the value directly, but instead use methods that give you a safe result when the value is missing.

There are a few strategies:

**Give a default value**

```java
String result = optionalString.orElse("");
// Uses the value if present, otherwise ""
```

**Compute a default only when needed**

```java
String result = optionalString.orElseGet(() -> System.getProperty("myapp.default"));
// Calls the function only if the Optional is empty
```

**Throw an exception if empty**

```java
String result = optionalString.orElseThrow(IllegalStateException::new);
// Throws the exception when no value is present
```

---

- `java.util.Optional`
    - `T orElse(T other)` Returns the value if present, otherwise returns other.
    - `T orElseGet(Supplier<? extends T> other)` Returns the value if present, otherwise runs the supplier function and returns its result.
    - `<X extends Throwable> T orElseThrow(Supplier<? extends X> exceptionSupplier)` Returns the value if present, otherwise throws the exception created by the supplier.

## Using a Value If It Exists

In the last section, we looked at how to give a default value when an `Optional` is empty.
Another way is to use the value only if it exists;

- `ifPresent` → Runs some code only when the `Optional` has a value. If it’s empty, nothing happens.
    ```java
    optionalValue.ifPresent(v -> results.add(v));  
    // adds v to the set only if v is present
    ```

    This can also be written shorter with a method reference:
    ```java
    optionalValue.ifPresent(results::add);
    ```

- `ifPresentOrElse` → Runs one action if the value exists, and another action if it does not.
    ```java
    optionalValue.ifPresentOrElse(
        v -> System.out.println("Found " + v),
        () -> System.out.println("No value found")
    );
    ```
    This way, you don’t need to check for null yourself.


---

- `java.util.Optional` 
    - `void ifPresent(Consumer<? super T> action)` If the value exists, run the action with it. If not, do nothing.
    - `void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)` If the value exists, run the action. If it’s empty, run the emptyAction instead.

## Working with Optional in a Pipeline

So far, we looked at ways to get a value out of an `Optional`.
Another good strategy is to keep the `Optional` and transform it step by step.

- `map` → Changes the value inside the `Optional`. If the `Optional` is empty, it stays empty.
    ```java
    Optional<Path> transformed = optionalString.map(Path::of);
    // If optionalString is empty → transformed is also empty
    ```

- `filter` → Keeps the value only if it matches a condition. Otherwise, the result is empty.
    ```java
    Optional<Path> transformed = optionalString
        .filter(s -> s.endsWith(".txt"))  // only keep .txt files
        .map(Path::of);

    ```

- `or` → If the `Optional` has no value, use another `Optional` instead. The alternative is created only when needed.
    ```java
    Optional<String> result = optionalString.or(() ->
        alternatives.stream().findFirst()
    );
    ```

---

- `java.util.Optional`
    - `<U> Optional<U> map(Function<? super T,? extends U> mapper)` If a value is present, applies the function and wraps the result in a new `Optional`.
    - `Optional<T> filter(Predicate<? super T> predicate)` Keeps the value only if it matches the condition (predicate). If it doesn’t, returns an empty `Optional`.
    - `Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)` If a value is present, returns this `Optional`. If not, calls the supplier to provide an alternative `Optional`.


## How Not to Use Optional

Many developers like the `Optional` type in Java because it makes code safer and easier to read. But if you use it in the wrong way, it is no better than the old “value or null” style.

### Why get() Is a Problem


The method `get()` gives you the value inside an `Optional` if one is present. But if the `Optional` is empty, `get()` will `throw` an exception.

```java
Optional<User> maybeUser = ...;
maybeUser.get().sendEmail();
```

This code is just as dangerous as:

```java
User user = ...;
user.sendEmail();
```

In both cases, you risk an error if there is no user. Using `get()` directly makes your code no safer than working with `null`.

### Why isPresent() and isEmpty() Don’t Help Much

You might try:

```java
if (maybeUser.isPresent()) {
    maybeUser.get().sendEmail();
}
```

But this is almost the same as:

```java
if (user != null) {
    user.sendEmail();
}
```

The logic is clearer, but you have not really solved the problem. You are still checking manually instead of using better features of `Optional`.

### Better Approaches

Instead of calling `get()`, try these methods:

```java
maybeUser.ifPresent(user -> user.sendEmail());
```

The code runs only if a user is present.

```java
User user = maybeUser.orElse(new GuestUser());
```

You provide a default value when the `Optional` is empty.

```java
User user = maybeUser.orElseThrow();
```

This makes it clear that the program should fail if no user exists. Only use this when you are sure the value is there.

### Common Mistakes to Avoid

Here are some tips for writing cleaner code with `Optional`:
- Never assign null to an `Optional`. If you do, you create confusion: `Optional` is meant to replace `null`, not hide it.
- Avoid `Optional` as a field inside a class.
    For example:
    ```java
    class Order {
        private Optional<String> note; // Not recommended
    }
    ```

    Each `Optional` creates an extra object, and it adds no real benefit inside your own class. Use `null` internally if a field is absent.

- `Optional` as a method parameter is not recommended. It makes the method call harder when you already have a value. 
    Instead, write two versions of the method: one with the parameter and one without.
- Returning an `Optional` is fine. It shows that the result may be missing.    
- Don't put Optional objects in a set, and don't use them as keys for a map. Collect the values instead.

---

- `java.util.Optional` 
    - `T get()` Returns the value inside the `Optional`. If the `Optional` is empty, it throws a `NoSuchElementExcepti`
    - `T orElseThrow()` Works like `get()`. It gives the value, or throws an exception if there is no value.
    - `boolean isEmpty()` Returns `true` if there is no value inside the `Optional`.
    - `boolean isPresent()` Returns `true` if there is a value inside the `Optional`.

## Creating Optional Values in Java

Until now, we have mostly looked at how to use an `Optional` object created by someone else. But sometimes you need to create your own `Optional`. Java provides several static methods to do this.

- `java.util.Optional`
    - `static <T> Optional<T> of(T value)` Creates an `Optional` with the given value. Warning: If the value is `null`, it throws a `NullPointerException`.
    - `static <T> Optional<T> ofNullable(T value)`Creates an `Optional` containing the value if it is not `null`. If the value is `null`, it returns an empty Optional.
    - `static <T> Optional<T> empty()` Creates an empty `Optional` that contains no value.

## Combining Optional Values Using flatMap

Sometimes, we have a method that returns an `Optional<T>`, and the type `T` itself has a method that returns an `Optional<U>`. 
If these were normal methods, we might just write `s.f().g()`. But that doesn’t work here because `s.f()` gives an `Optional<T>`, not a `T`.

Instead, we can use `flatMap` to chain these calls safely:

```java
Optional<U> result = s.f().flatMap(T::g);
```

Here’s what happens:

- If `s.f()` contains a value, the method `g` is applied to it.
- If `s.f()` is empty, the result is an empty `Optional<U>` automatically.

This technique can be repeated for multiple methods or functions that return `Optional` values. By chaining `flatMap` calls, 
you can create a pipeline of operations that only produces a result if all steps succeed.

---

- `java.util.Optional`
    - `<U> Optional<U> flatMap(Function<? super T,? extends Optional<? extends U>> mapper)` This method takes a function and applies it to the value inside the Optional, if there is one.
        - If the Optional is empty, it returns an empty Optional.
        - If the Optional has a value, it returns the result of the function, which is also an Optional.

---

- [Home](./../../../README.md)
- [Java Tutorials](./../../tutorials.md)
