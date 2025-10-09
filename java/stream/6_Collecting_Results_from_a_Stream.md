# Collecting Results from a Stream

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
  - `List<T> toList()` yields a list of the elements in this stream.
  - `void forEach(Consumer<? super T> action)` invokes action on each element of the stream.
  - `Object[] toArray()`
  - `<A> A[] toArray(IntFunction<A[]> generator)`
        - yield an array of objects, or of type A when passed a constructor reference A[]::new. These are terminal operations.
  - `<R,A> R collect(Collector<? super T,A,R> collector)` collects the elements in this stream, using the given collector. The Collectors class has factory methods for many collectors.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [How to Convert Optional Values into Streams](./5_How_to_Convert_Optional_Values_into_Streams.md)
