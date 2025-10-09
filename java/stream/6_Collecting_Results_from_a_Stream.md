# Collecting Results from a Stream

```info
Author      Ter-Petrosyan Hakob
```

---

When you finish working with a Java Stream, you often want to see or save the results.
There are several ways to do this, depending on what you want to get — a printed list, an array, or a collection.

## Using forEach to Process Each Element

The simplest way to see what’s inside a stream is by using the `forEach` method. It applies a function to every element in the stream.

```java
numbersStream.forEach(System.out::println);
```

This line prints every number in the stream.

If you are using a parallel stream, elements may appear in any order. If you want to keep the stream’s natural order, use 
`forEachOrdered` instead — but doing so might make the stream slower.

## Converting a Stream into a List or Array

Most of the time, you want to store the results instead of just printing them.
You can turn the stream into a `List` or `Array`.

**Example 1 – List:**

```java
List<Integer> numbers = numbersStream.toList();
```

This gives you a list of elements from the stream.

**Example 2 – Array:**

```java
String[] names = nameStream.toArray(String[]::new);
```

Here we use `String[]::new` to tell Java to make a `String` array.
If you just write `stream.toArray()`, Java gives you an `Object[]`, which isn’t as useful.

## Using collect for More Control

The collect method is more flexible. It uses a `Collector` — an object that gathers the elements and produces the final result. 
The `Collectors` class provides many ready-to-use collectors.

**Example: collect into a List**

```java
List<String> names = stream.collect(Collectors.toList());
```

**Example: collect into a Set**
```java
Set<String> uniqueNames = stream.collect(Collectors.toSet());
```

These create a list or set, but Java doesn’t guarantee if it’s modifiable or thread-safe.
If you want a specific kind of collection, you can choose it explicitly:

```java
TreeSet<String> sortedNames = stream.collect(Collectors.toCollection(TreeSet::new));
```

## Joining Stream Elements

If you have a stream of strings and you want to combine them into one text, use `joining()`.

**Example – join without a separator**

```java
String combined = stream.collect(Collectors.joining());
```

**Example – join with a separator**

```java
String combined = stream.collect(Collectors.joining(" | "));
```

**Example with numbers:**

If your stream contains numbers or other objects, convert them to strings first:

```java
String report = stream
        .map(Object::toString)
        .collect(Collectors.joining(", "));

```

This will join all values like: `2, 4, 6, 8`.

## Summarizing Numbers in a Stream

If your stream holds numbers (or objects that can be turned into numbers), you can use the summarizing collectors.
They calculate several values at once — `total`, `average`, `min`, and `max`.

```java
IntSummaryStatistics stats = wordsStream.collect(Collectors.summarizingInt(String::length));
```

Now you can ask the statistics object for useful information:

```java
double averageLength = stats.getAverage();
int longestWord = stats.getMax();
int count = stats.getCount();
```

---

- `java.util.stream.BaseStream` 
  - `Iterator<T> iterator()` Returns an iterator that lets you go through each element of the stream one by one.
        This is a terminal operation, meaning that once you use it, the stream is finished.

---

- `java.util.stream.Stream` 
  - `List<T> toList()` Returns a list that contains all the elements from the stream.
  - `void forEach(Consumer<? super T> action)` Runs the given action (for example, `System.out::println`) on every element in the stream..
  - `Object[] toArray()` Returns an array with all stream elements. The array type is `Object[]`.
  - `<A> A[] toArray(IntFunction<A[]> generator)` Returns an array of a specific type. You must pass the constructor reference, such as `String[]::new`.
        Both `toArray()` methods are terminal operations.
  - `<R,A> R collect(Collector<? super T,A,R> collector)` Collects the elements of the stream into a result using a `Collector`.

---        

- `java.util.stream.Collectors` The Collectors class in Java provides many useful factory methods for collecting stream elements 
        into different forms such as lists, sets, strings, or summary statistics.
  - `static <T> Collector<T, ?, List<T>> toList()`
  - `static <T> Collector<T, ?, List<T>> toUnmodifiableList()`
  - `static <T> Collector<T, ?, Set<T>> toSet()`
  - `static <T> Collector<T, ?, Set<T>> toUnmodifiableSet()`
   - These methods return collectors that gather stream elements into a list or a set.
  - `static <T, C extends Collection<T>> Collector<T, ?, C> toCollection(Supplier<C> collectionFactory)` This method lets you decide what kind of collection to use.
        You can pass a constructor reference, such as `TreeSet::new` or `LinkedList::new`.
  - `static Collector<CharSequence, ?, String> joining()`
  - `static Collector<CharSequence, ?, String> joining(CharSequence delimiter)`
  - `static Collector<CharSequence, ?, String> joining(CharSequence delimiter, CharSequence prefix, CharSequence suffix)`
    - These methods create collectors that join strings together. The delimiter is placed between elements, while `prefix` and `suffix` are 
    added before and after the entire result. If they are not specified, they are empty.

---

**Summarizing Numbers:**

- `static <T> Collector<T, ?, IntSummaryStatistics> summarizingInt(ToIntFunction<? super T> mapper)`
- `static <T> Collector<T, ?, LongSummaryStatistics> summarizingLong(ToLongFunction<? super T> mapper)`
- `static <T> Collector<T, ?, DoubleSummaryStatistics> summarizingDouble(ToDoubleFunction<? super T> mapper)`

These methods return collectors that produce a summary statistics object — for integers, longs, or doubles.
This object contains values such as `count`, `sum`, `average`, `max`, and `min`.

**Common Methods:**

- `long getCount()` — returns how many elements were summarized.
- `(int|long|double) getSum()` — returns the total sum.
- `double getAverage()` — returns the average value, or 0 if there are no elements.
- `(int|long|double) getMax()` — returns the largest element, or the smallest possible value if the stream was empty.
- `(int|long|double) getMin()` — returns the smallest element, or the largest possible value if the stream was empty.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [How to Convert Optional Values into Streams](./5_How_to_Convert_Optional_Values_into_Streams.md)
- [Collecting Stream Elements into Maps](./7_Collecting_Stream_Elements_into_Maps.md)
