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


---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Downstream Collectors in Streams](./9_Downstream_Collectors_in_Streams.md)