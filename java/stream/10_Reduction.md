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
(x \mathop{\mathrm{op}} y) \mathop{\mathrm{op}} z = x \mathop{\mathrm{op}} (y \mathop{\mathrm{op}}z)
$$

For example, addition is associative:

$$
(2+3)+4=2+(3+4)
$$

But subtraction is not associative:

$$
(6−3)−2 \not= 6−(3−2)
$$

## Using an Identity Value

Many operations have an identity value, which is a starting point that doesn’t change the result.

- For addition, the identity is `0`
- For multiplication, the identity is `1`
- For string concatenation, the identity is `""` (empty string)

Using an identity lets you avoid dealing with `Optional`:

```java
List<Integer> numbers = List.of(2, 5, 7);
int sum = numbers.stream().reduce(0, (a, b) -> a + b);
// Computes 0 + 2 + 5 + 7 = 14
```

If the list is empty, the identity (`0`) is returned.

## Reducing Different Types

Sometimes, the elements in the stream are different from the result type. For example, suppose we want the total length of words:

```java
List<String> words = List.of("apple", "banana", "kiwi");
int totalLength = words.stream()
.reduce(0, 
    (sum, word) -> sum + word.length(), 
    (sum1, sum2) -> sum1 + sum2
);
```

Here:
- The first function adds the length of each word to the running total
- The second function combines results if the stream is parallelized

## When reduce Might Not Be Enough

Sometimes you want to collect results into complex objects, like a set or a map. `reduce` is not ideal for this because it allows only one identity value, and objects like sets are not thread-safe in parallel streams.

In these cases, use collect:

```java
BitSet bits = numbers.stream().collect(
    BitSet::new,        // creates a new BitSet
    BitSet::set,        // adds an element
    BitSet::or          // combines two BitSets
);
```

--- 

**java.util.Stream**

```java

 //Adds up all elements in the stream using the accumulator function. If identity is given, 
 //it is the first value. If combiner is given, it can join results from different parts of the stream

Optional<T> reduce(BinaryOperator<T> accumulator)

T reduce(
    T identity, 
    BinaryOperator<T> accumulator)

<U> U reduce(
    U identity, 
    BiFunction<U,? super T,U> accumulator, 
    BinaryOperator<U> combiner)

 // Puts all elements into a result of type R. For each part of the stream, supplier gives a starting result, 
 // accumulator adds elements to it, and combiner joins two results together.
<R> R collect(
    Supplier<R> supplier, 
    BiConsumer<R,? super T> accumulator, 
    BiConsumer<R,R> combiner)    

```

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Downstream Collectors in Streams](./9_Downstream_Collectors_in_Streams.md)