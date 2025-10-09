# Grouping and Partitioning

```info
Author      Ter-Petrosyan Hakob
```

---


When working with collections in Java, you often need to group elements that share the same property. 
For example, you might want to group students by their grade or products by category. 
Doing this manually can be tedious—you would need to create a new list for each group and carefully merge new items into it. 
Luckily, Java provides a simple way to handle this using the groupingBy method.

## Grouping Elements by a Property

Suppose you have a collection of books, and each book has a genre. You want to group the books by genre. Here's how you can do it:

```java
public record Book(String title, String genre) {}

List<Book> books = List.of(
    new Book("Journey to the West", "Adventure"),
    new Book("Sherlock Holmes", "Mystery"),
    new Book("Murder on the Orient Express", "Mystery"),
    new Book("The Hobbit", "Fantasy")
);

Map<String, List<Book>> booksByGenre = books.stream()
    .collect(Collectors.groupingBy(Book::genre));

```

Now, if you want all mystery books:

```java
List<Book> mysteryBooks = booksByGenre.get("Mystery");
// Contains "Sherlock Holmes" and "Murder on the Orient Express"
```

The `Book::genre` function is called the classifier. It tells Java how to group the elements. 
The result is a map where each `key` is a `genre`, and each `value` is a `list of books` in that `genre`.


## Quick Note on Concurrent Grouping

If you use a parallel stream, you can group elements safely at the same time using `groupingByConcurrent`:

```java
ConcurrentMap<String, List<Book>> booksByGenreConcurrent = books.parallelStream()
    .collect(Collectors.groupingByConcurrent(Book::genre));
```

This works just like groupingBy, but it is thread-safe for parallel operations.

## Partitioning Elements by a Condition

Sometimes, you don’t want many groups, but just two categories: elements that meet a condition and elements that don’t. This is called partitioning, and Java provides `partitioningBy` for this.

For example, let’s split books into those that are Fantasy and all others:

```java
Map<Boolean, List<Book>> fantasyAndOtherBooks = books.stream()
    .collect(Collectors.partitioningBy(b -> b.genre().equals("Fantasy")));
    
List<Book> fantasyBooks = fantasyAndOtherBooks.get(true);
// Contains "The Hobbit"

List<Book> nonFantasyBooks = fantasyAndOtherBooks.get(false);
// Contains the other three books
```

Here, the predicate `(b -> b.genre().equals("Fantasy"))` returns `true` or `false`. 
Java automatically creates a map with `true` and `false` keys and groups the elements accordingly.

Summary

- `groupingBy`: Groups elements into lists by a property. Great for multiple categories.
- `groupingByConcurrent`: Same as `groupingBy`, but safe for parallel streams.
- `partitioningBy`: Splits elements into two groups based on a condition.

This approach makes your code cleaner and easier to read than manually building maps or lists.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Collecting Stream Elements into Maps](./7_Collecting_Stream_Elements_into_Maps.md)
- [Downstream Collectors in Streams](9_Downstream_Collectors_in_Streams.md)
