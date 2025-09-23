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
