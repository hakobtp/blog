# filter, map, and flatMap

```info
Author      Ter-Petrosyan Hakob
```

---

Streams let you transform data easily. Transformation means creating a new stream whose elements are based on another stream.

## filter — Keep only what you want

The `filter` method keeps elements that match a condition.
The condition is a `Predicate<T>`, which is a function returning `true` or `false`.

```java
List<String> words = List.of("apple", "banana", "strawberry", "kiwi");

// Keep only long words (length > 5)
Stream<String> longWords = words.stream()
    .filter(w -> w.length() > 5);

longWords.forEach(System.out::println);
```

## map — Transform each element

Use `map` when you want to change each element in a stream.
The function you provide returns a new value for each element.

```java
List<String> words = List.of("Apple", "Banana", "Cherry");

// Convert all words to lowercase
Stream<String> lowercaseWords = words.stream()
    .map(String::toLowerCase);

lowercaseWords.forEach(System.out::println);
```

Here, we used `map` with a method reference. You can also use a lambda to pick part of a string:

```java
Stream<Character> firstLetters = words.stream()
    .map(w -> w.charAt(0));

firstLetters.forEach(System.out::println);
```

## flatMap — Flatten nested streams

Sometimes a function returns a stream for each element. Using map here gives you a stream of streams, but often you want one flat stream. That’s what `flatMap` does.

Example: split words into letters:

```java
List<String> words = List.of("cat", "dog");

Stream<String> letters = words.stream()
    .flatMap(w -> Stream.of(w.split("")));

letters.forEach(System.out::println);
```

Here, `flatMap` “flattens” the small streams into one big stream.

## Using mapMulti — Efficient alternative

If creating many small streams is slow, `mapMulti` can help. It directly passes results to a collector instead of making new streams.

Example: get each character in a word:

```java
List<String> words = List.of("hi", "ok");

Stream<String> letters = words.stream()
    .mapMulti((word, collector) -> {
        for (char c : word.toCharArray()) {
            collector.accept(String.valueOf(c));
        }
    });

letters.forEach(System.out::println);
```

--- 

- `java.util.stream.Stream`
    - `Stream<T> filter(Predicate<? super T> predicate)` Keeps only the elements that match a condition.
    - `<R> Stream<R> map(Function<? super T,? extends R> mapper)` Changes each element using a function.
    - `<R> Stream<R> flatMap(Function<? super T,? extends Stream<? extends R>> mapper)` Flattens many small streams into one stream.
    - `<R> Stream<R> mapMulti(BiConsumer<? super T,? super Consumer<R>> mapper)` For each element, add multiple results directly to the stream using a collector.

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Creating Streams](./2_Creating_Streams.md)
- [Transform, Combine, and Reduce Data](./4_Transform_Combine_and_Reduce_Data.md)