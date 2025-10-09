## How to Convert Optional Values into Streams

```info
Author      Ter-Petrosyan Hakob
```

---

In Java, an `Optional<T>` can either contain a value or be empty. Sometimes, we want to turn an `Optional` into a `Stream` so we can work with it more easily. The `stream()` method does exactly that: it creates a stream with one element if the `Optional` has a value, or no elements if it is empty.

But why would we want to do this? Let’s see a practical example.

## Using Optional with Lists of IDs

Suppose you have a list of customer IDs and a method that finds a customer by ID:

```java
Optional<Customer> findCustomer(String id)
```

You want to create a stream of valid customers, skipping IDs that do not exist in your database. 
You could first filter and then get the value like this:

```java
Stream<String> ids = ...;
Stream<Customer> customers = ids.map(CustomerService::findCustomer)
                                .filter(Optional::isPresent)
                                .map(Optional::get);
```

This works, but it is a bit clunky because it uses `isPresent()` and `get()`. A cleaner way is:

```java
Stream<Customer> customers = ids.map(CustomerService::findCustomer)
                                .flatMap(Optional::stream);
```

Here’s what happens:
- Each `Optional` becomes a stream with zero or one element.
- `flatMap` combines all these small streams into one stream of customers.
- Any missing customers are automatically skipped.

This is shorter, safer, and easier to read.


## Handling Methods That Return Null

Not all methods return `Optional`. Some older methods return `null` when a value is missing. For example:

```java
Customer CustomerService.oldFind(String id) // returns null if no customer
```

You could filter out `null` values like this:

```java
Stream<Customer> customers = ids.map(CustomerService::oldFind)
                                .filter(Objects::nonNull);
```

Or, using `Stream.ofNullable()` with `flatMap`, you can write it more elegantly:

```java
Stream<Customer> customers = ids.flatMap(id -> Stream.ofNullable(CustomerService.oldFind(id)));
```

Or even:

```java
Stream<Customer> customers = ids.map(CustomerService::oldFind)
                                .flatMap(Stream::ofNullable);
```

`Stream.ofNullable(obj)` creates:
- An empty stream if obj is `null`.
- A stream containing the object if it is not `null`.

This method avoids manual `null` checks and makes the code cleaner.

## A Fresh Example

Imagine you have a list of book ISBNs and a method that finds books in your system:

```java
Optional<Book> findBook(String isbn)
```

You can get a stream of available books like this:

```java
List<String> bookIds = List.of("B101", "B102", "B103");
Stream<Book> books = bookIds.stream()
                            .map(BookService::findBook)
                            .flatMap(Optional::stream);
```

If `B102` does not exist, it will be skipped. You will get a stream with only `B101` and `B103`.

Why This Is Useful

- Simplifies code: You don’t need if statements to check if the value exists.
- Handles empty values automatically: Missing elements are skipped.
- Works with modern Java streams: Combines well with `map`, `filter`, and other stream operations.

This approach is very helpful when processing lists of IDs, products, users, or any items that may be missing.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Transform, Combine, and Reduce Data](./4_Transform_Combine_and_Reduce_Data.md)
- [Collecting Results from a Stream](./6_Collecting_Results_from_a_Stream.md)