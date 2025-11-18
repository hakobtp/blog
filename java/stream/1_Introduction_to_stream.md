# Introduction to Stream

```info
Author      Ter-Petrosyan Hakob
```

---

Streams let you work with collections more simply than loops. With a stream, you tell Java what you want to do, not how to do it. 
Java then chooses the best way to run your code. 

When you work with a collection, you often loop through its items and do something with each one. 
For example, imagine you have a list of words and you want to count how many have more than 8 letters. 

```java
List<String> words = List.of(
    "programming", 
    "java",
    "hibernate", 
    "code",
    "development",
    "test",
    "bug",
    "fix",
    "application",
    "data"
);

int count = 0;
for (String word : words) {
    if (word.length() > 8) {
        ++count;
    }
}

System.out.println("Words with more than 8 letters (using loop): " + count);

long streamCount = words.stream()
    .filter(word -> word.length() > 8)
    .count();

System.out.println("Words with more than 8 letters (using stream): " + streamCount);
```

With a stream, you do not need to read a long loop to see how filtering and counting work. 
The method names (for example, `filter()` and `count()`) 
show exactly what the code does. In a loop, you must write every step in order. 
A stream, however, can run those steps in any order it wants, as long as it gives the correct result.


At first glance, streams look like collections—you can change data and get results. But streams behave in different ways:

- **No storage:** A stream does not keep its items itself. The data may come from a list, a file, or be created only when needed.
- **No changes to the source:** When you use methods like `filter()`, the original data does not change. 
    Instead, the stream makes a new sequence of items that match your filter.
- **Operations run only when needed:** Streams wait to do work until you ask for a result. This is called lazy evaluation. For example, if you only want the first five long words, the stream stops after it finds five matches. Because of this lazy behavior, you can even have streams that never end (infinite streams), as long as you only take a limited number of items from them.

Let’s look at the example again. We use `stream()` or `parallelStream()` on the words list to make a stream. Then:

- **Filter:** `.filter(w -> w.length() > 8)` keeps only words longer than 8 letters.
- **Count:** `.count()` gives the total number of words left.

This pattern—making a stream, transforming it, and then getting a result—is very common. You can think of it as three steps:

- Create a stream from your data source.
- Transform the stream with one or more intermediate operations (like `filter`, `map`, or `sorted`).
- Finish with a terminal operation (like `count`, `findFirst`, or `collect`). 


---

- **`java.util.stream.Stream<T>`**
    - **`Stream<T> filter(Predicate<? super T> p)`** Creates a new stream that contains only those elements 
        from the original stream for which `predicate` returns `true`.
    - **`long count()`** Returns how many elements are in the stream. This is a terminal operation, which means 
        it runs all previous steps and produces a final result.

---

- **`java.util.Collection<E>`**
    - **`default Stream<E> stream()`** Creates a normal (sequential) stream of the collection’s elements.
    - **`default Stream<E> parallelStream()`** Creates a parallel stream, which may process elements at the same time on different threads.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Creating Streams](./2_Creating_Streams.md)