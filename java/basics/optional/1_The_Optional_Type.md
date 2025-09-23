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

- `java.util.Optional`
    - `<U> Optional<U> map(Function<? super T,? extends U> mapper)` If a value is present, applies the function and wraps the result in a new `Optional`.
    - `Optional<T> filter(Predicate<? super T> predicate)` Keeps the value only if it matches the condition (predicate). If it doesn’t, returns an empty `Optional`.
    - `Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)` If a value is present, returns this `Optional`. If not, calls the supplier to provide an alternative `Optional`.
---

- [Home](./../../../README.md)
- [Java Tutorials](./../../tutorials.md)
