# Reduction

```info
Author      Ter-Petrosyan Hakob
```

---

The reduce method in Java is a powerful tool for combining all elements in a stream into a single result. 
It works by repeatedly applying a function that combines two values.

The most basic use of reduce is summing numbers. Suppose we have a list of numbers:

```java
List<Integer> numbers = List.of(2, 5, 7);
Optional<Integer> sum = numbers.stream().reduce((a, b) -> a + b);
```

Here, reduce calculates $$2 + 5 + 7$$. We use `Optional` because the stream might be empty, in which case there would be no sum.

You can also write the sum in a simpler way:

```java
Optional<Integer> sum = numbers.stream().reduce(Integer::sum);
```

This does the same thing but is shorter and more readable.

## Using Any Binary Operation

You don’t have to sum numbers. Any operation that combines a previous result with the next element works. 

For example:
- Multiplying numbers
- Finding the largest number
- Combining strings into a single sentence

For instance, to join words:

```java
List<String> words = List.of("Java", "is", "fun");
Optional<String> sentence = words.stream().reduce((a, b) -> a + " " + b);
```

The result would be `"Java is fun"`.

## Associative Operations and Parallel Streams

If you want to use reduce with parallel streams, the operation must be associative. This means it doesn’t matter how the numbers are grouped:

$$
(x \mathop{\mathrm{op}} y) \mathrm{op}z = x \mathop{\mathrm{op}} (y \mathrm{op}z)
$$



---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Downstream Collectors in Streams](./9_Downstream_Collectors_in_Streams.md)