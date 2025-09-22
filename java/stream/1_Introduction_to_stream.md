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


## Creating Streams

Streams in Java let you work with data in a flexible way. Instead of writing loops again and again, you can describe what you want to do with the data, and the stream takes care of the rest.

If you already have a collection (like a `List` or `Set`), you can turn it into a stream easily:

```java
List<String> colors = List.of("red", "blue", "yellow");
Stream<String> colorStream = colors.stream();
```

For arrays, use `Stream.of` or `Arrays.stream`:

```java
String[] colors = {"red", "blue", "yellow"};
Stream<String> colorStream = Stream.of(colors);
```

If you only want part of an array, say the first three numbers:

```java
int[] numbers = {1, 2, 3, 4, 5};
IntStream firstThree = Arrays.stream(numbers, 0, 3);
```

Sometimes you want an empty stream, for example as a default value. Use:

```java
Stream<String> emptyStream = Stream.empty();
```

Streams don’t have to be finite! You can make streams that keep producing data.
There are two common ways:

`generate` takes a function with no input and keeps calling it.

```java
Stream<String> echoes = Stream.generate(() -> "Hello!");
Stream<Double> randomNumbers = Stream.generate(Math::random);
```

The first one repeats `"Hello!"` forever, the second one gives endless random numbers.

`iterate` starts with a seed (first value) and applies a function to create the next ones.

```java
Stream<Integer> counting = Stream.iterate(0, n -> n + 1);
```

This creates numbers: `0, 1, 2, 3, ...`. 

To stop it, use a condition:
```java
Stream<Integer> upToFive = Stream.iterate(0, n -> n < 5, n -> n + 1);
```

This produces `0, 1, 2, 3, 4`.


- Special Factory Methods
    - `Stream.ofNullable(obj)` → makes a stream of 1 element if obj is not null, otherwise empty.
    - `String.lines()` → turns a text block into a stream of its lines.

```java
String text = "Hello\nHola\nCiao";
Stream<String> greetings = text.lines();
```

If you want to process a file line by line:

```java
try (Stream<String> lines = Files.lines(Path.of("data.txt"))) {
    lines.forEach(System.out::println);
}
```

The `try` block makes sure the file closes properly.

If you need to build a stream step by step, use `Stream.Builder`:

```java
Stream.Builder<Integer> builder = Stream.builder();
builder.add(10);
builder.add(20);
builder.add(30);
Stream<Integer> numbers = builder.build();

```

If you only have an `Iterator` or `Iterable` (not a collection), you can still make a stream:

```java
Stream<String> fromIterator = StreamSupport.stream(Spliterators
        .spliteratorUnknownSize(iterator, Spliterator.ORDERED), false);
```

If you have an Iterable that is not a collection, you can turn it into a stream by callin

```java
StreamSupport.stream(iterable.spliterator(), false);
````


Streams work on top of collections. If you change the collection while the stream is running, strange errors may happen.

**Safe (though unusual):**
```java
List<String> items = new ArrayList<>(List.of("A", "B"));
Stream<String> s = items.stream();
items.add("C"); 
long count = s.count(); // Works, but not recommended
```

**Unsafe:**
```java
List<String> items = new ArrayList<>(List.of("A", "B"));
Stream<String> s = items.stream();
s.forEach(x -> items.remove(x)); // Error
```
---

- `java.util.stream.Stream` 
    - `static <T> Stream<T> of(T... values)` Creates a stream from the given values..
    - `static <T> Stream<T> empty()` Creates a stream with no elements.
    - `static <T> Stream<T> generate(Supplier<T> s)` Creates an infinite stream. Each element is produced by calling the given function again and again.
    - `static <T> Stream<T> iterate(T seed, UnaryOperator<T> f)` Creates an infinite stream. Starts with `seed`, then applies the function to get the next values.
    - `static <T> Stream<T> iterate(T seed, Predicate<? super T> hasNext, UnaryOperator<T> f)` Like the previous method, but stops when 
        the condition (`hasNext`) is no longer true.
    - `static <T> Stream<T> ofNullable(T t)` If `t` is null, returns an empty stream. Otherwise, returns a stream with that single value.

---

- `java.util.Spliterators`
    - `static <T> Spliterator<T> spliteratorUnknownSize(Iterator<? extends T> iterator, int characteristics)` Turns an Iterator into a `Spliterator` 
        (an iterator that can split work for parallel processing). The characteristics value describes special properties of the data, 
        for example `Spliterator.ORDERED` if the elements have a fixed order. Key idea `spliteratorUnknownSize` helps you reuse old-style Iterator code with modern streams.
        ```java
          Iterator<String> iterator = List.of("apple", "banana", "cherry").iterator();
            Stream<String> stream = StreamSupport.stream(
            Spliterators.spliteratorUnknownSize(iterator, Spliterator.ORDERED),
            false
        );
        ```    

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
