## Stream: Transform, Combine, and Reduce Data

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

------



## Simple Reductions
Now that you have seen how to create and transform streams, we will finally get to the most important point—getting answers from the stream data. The methods covered in this section are called reductions. Reductions are terminal operations. They reduce the stream to a nonstream value that can be used in your program.

You have already seen a simple reduction: the count method that returns the number of elements of a stream.

Other simple reductions are max and min that return the largest or smallest value. There is a twist—these methods return an Optional<T> value that either wraps the answer or indicates that there is none (because the stream happened to be empty). In the olden days, it was common to return null in such a situation. But that can lead to null pointer exceptions when it happens in an incompletely tested program. The Optional type is a better way of indicating a missing return value. We discuss the Optional type in detail in the next section. Here is how you can get the maximum of a stream:

Optional<String> largest = words.max(String::compareToIgnoreCase);
System.out.println("largest: " + largest.orElse(""));
The findFirst returns the first value in a nonempty collection. It is often useful when combined with filter. For example, here we find the first word that starts with the letter Q, if it exists:

Optional<String> startsWithQ
   = words.filter(s -> s.startsWith("Q")).findFirst();
If you are OK with any match, not just the first one, use the findAny method. This is effective when you parallelize the stream, since the stream can report any match that it finds instead of being constrained to the first one.

Optional<String> startsWithQ
   = words.parallel().filter(s -> s.startsWith("Q")).findAny();
If you just want to know if there is a match, use the terminal anyMatch operation with a predicate argument:

boolean aWordStartsWithQ
   = words.parallel().anyMatch(s -> s.startsWith("Q"));
There are methods allMatch and noneMatch that return true if all or no elements match a predicate. These methods also benefit from being run in parallel.