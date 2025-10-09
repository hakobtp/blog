## Transform, Combine, and Reduce Data

```info
Author      Ter-Petrosyan Hakob
```

---

Streams are a powerful way to work with data in Java. You can take parts of a stream, combine streams, remove duplicates, sort elements, or calculate results. Let’s see how.


### Extracting Parts of a Stream


If you only need a certain number of elements from a stream, use limit.
This is very useful with streams that could go on forever.

```java
Stream<Integer> numbers = Stream.iterate(1, n -> n + 1).limit(50);
```

This stream now contains numbers from 1 to 50.

Sometimes, you want to ignore the first few elements.

```java
Stream<String> words = Stream.of("","apple","banana","cherry").skip(1);
words.forEach(System.out::println);

```

Output:

```
apple
banana
cherry
```

You can take elements until a rule becomes `false`.

```java
Stream<Integer> digits = Stream.of(1,2,3,0,4,5).takeWhile(n -> n != 0);
digits.forEach(System.out::println);
```

Output:

```
1
2
3
```

Opposite of `takeWhile`: skip elements until the condition is `false`.

```java
Stream<String> fruits = Stream.of(""," ","apple","banana");
Stream<String> clean = fruits.dropWhile(s -> s.isBlank());
clean.forEach(System.out::println);
```

Output:

```
apple
banana
```

You can merge two streams into one.

```java
Stream<String> letters = Stream.concat(Stream.of("A","B"), Stream.of("C","D"));

letters.forEach(System.out::println);
```

Output:

```
A
B
C
D
```

---

- `java.util.stream.Stream`
   - `Stream<T> limit(long maxSize)` Takes the first `maxSize` elements from the stream.
   - `Stream<T> skip(long n)` Ignores the first `n` elements and keeps the rest.
   - `Stream<T> takeWhile(Predicate<? super T> predicate)` Takes elements from the start while the condition is `true`.
   - `Stream<T> dropWhile(Predicate<? super T> predicate)` Skips elements from the start while the condition is `true`, then keeps the rest.
   - `static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)` Joins two streams: first `a`, then `b`.



### Other Transformations

`distinct()` – Remove duplicate elements

```java
Stream<String> unique = Stream.of("apple","apple","banana").distinct();
unique.forEach(System.out::println);
```

Output:

```
apple
banana
```

You can sort elements alphabetically or with custom rules.

```java
List<String> words = List.of("cat","elephant","dog");
Stream<String> longestFirst = words.stream()
   .sorted(Comparator.comparing(String::length)
   .reversed());

longestFirst.forEach(System.out::println);
```

Output:

```
elephant
cat
dog
```

Peek lets you see elements while the stream is processed.

```java
Stream<Integer> powers = Stream.iterate(1, n -> n * 2)
   .peek(n -> System.out.println("Processing " + n))
   .limit(5);
powers.forEach(n -> {});

```

Output:

```
Processing 1
Processing 2
Processing 4
Processing 8
Processing 16
```
---

- `java.util.stream.Stream`
   - `Stream<T> distinct()` Keeps only unique elements, removing duplicates.
   - `Stream<T> sorted()` Sorts elements in natural order. Important: the elements must implement `Comparable`, 
      meaning they can be compared to each other (like numbers or strings).
   - `Stream<T> sorted(Comparator<? super T> comparator)` Sorts the elements using a custom rule you provide.
   - `Stream<T> peek(Consumer<? super T> action)` Looks at each element while the stream is processed, useful for debugging.


### Simple Reductions

Reductions are terminal operations that return a single value from a stream.

`count()` – Count elements

```java
long total = Stream.of("apple","banana","cherry").count();
```
---

`max()` and `min()` – Find largest or smallest

These return an `Optional<T>`, which is safer than returning `null`.

```java
Optional<String> largest = Stream.of("apple","banana","cherry").max(String::compareToIgnoreCase);
System.out.println(largest.orElse("No words"));
```

---

`findFirst()` and `findAny()` – Find elements

```java
Optional<String> first = Stream.of("apple","banana","cherry")
                               .filter(s -> s.startsWith("b"))
                               .findFirst();
System.out.println(first.orElse("No match"));
```

`findAny()` is useful in parallel streams when you don’t care which match is returned.

---

Match checks: `anyMatch`, `allMatch`, `noneMatch`

```java
Stream<String> words = Stream.of("apple","banana","cherry");

// Check if any word starts with "b"
boolean anyB = words.anyMatch(s -> s.startsWith("b"));
System.out.println(anyB); // true
```

- `allMatch` → `true` if all elements match a condition.
- `noneMatch` → `true` if no elements match a condition.

---

- `java.util.stream.Stream` 
   - `Optional<T> max(Comparator<? super T> comparator)` Finds the largest element in the stream using the comparator. Returns an `Optional` (empty if the stream has no elements).
   - `Optional<T> min(Comparator<? super T> comparator)` Finds the smallest element in the stream using the comparator. Returns an `Optional`.
   - `Optional<T> findFirst()` Returns the first element of the stream, or empty if the stream is empty.
   - `Optional<T> findAny()` Returns any element of the stream, useful in parallel streams.
   - `boolean anyMatch(Predicate<? super T> predicate)` Returns `true` if any element matches the condition.
   - `boolean allMatch(Predicate<? super T> predicate)` Returns `true` if all elements match the condition.
   - `boolean noneMatch(Predicate<? super T> predicate)` Returns `true` if no element matches the condition.
   
---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [filter, map, and flatMap](./3_filter_map,_and_flatMap.md)
- [How to Convert Optional Values into Streams](./5_How_to_Convert_Optional_Values_into_Streams.md)