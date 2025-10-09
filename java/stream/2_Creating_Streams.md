# Creating Streams

```info
Author      Ter-Petrosyan Hakob
```

---

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

- `java.util.Arrays`
    - `static <T> Stream<T> stream(T[] array, int startInclusive, int endExclusive)` Creates a stream from a part of an array.
    The range starts at `startInclusive` (included) and ends at `endExclusive` (not included).

---

- `java.lang.String`
    - `static Stream<String> lines()` Splits a text into lines and returns them as a stream of strings.
    Each line (separated by `\n`) becomes one element in the stream.

--- 

- `java.util.regex.Pattern`    
    - `Stream<String> splitAsStream(CharSequence input)` Splits a text using a regular expression (regex) and gives the parts as a stream of strings.
    Each match of the pattern is used as a separator.
    ```java
        String text = "apple,banana;cherry orange";

        // Split by comma, semicolon, or space
        Pattern pattern = Pattern.compile("[,; ]+");

        Stream<String> words = pattern.splitAsStream(text);

        words.forEach(System.out::println);
    ```            

---

- `java.nio.file.Files`
    - `static Stream<String> lines(Path path)` 
    - `static Stream<String> lines(Path path, Charset cs)` Read a text file and return a stream of lines.
    Each line in the file becomes one element in the stream.
    By default, it uses UTF-8, or you can provide a different charset.

---

- `java.util.stream.StreamSupport`
    - `static <T> Stream<T> stream(Spliterator<T> spliterator, boolean parallel)` Creates a stream from a `Spliterator`.
    You can choose `parallel = true` to process the elements in parallel or `false` for normal sequential processing.

---

- `java.lang.Iterable`
    - `Spliterator<T> spliterator()` Creates a `Spliterator` from any `Iterable` (like a `List` or `Set`).
    A `Spliterator` can be used to make a stream or for parallel processing.
    The default version does not split the elements and does not know the size.

---

- `java.util.Scanner`   
    - `public Stream<String> tokens()` Creates a stream of words (or tokens) from a `Scanner`.
    Each token is what the scanner would normally return using `next()`. 

    ```java
    String text = "apple banana cherry";
    Scanner scanner = new Scanner(text);

    Stream<String> words = scanner.tokens();

    words.forEach(System.out::println);
    scanner.close();
    ```

    `tokens()` is a simple way to turn scanner input into a stream for easy processing.

---

- `java.util.function.Supplier<T>`
    - `T get()` Produces or “supplies” a value when called. It does not take any input, it just returns something

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Introduction to Stream](./1_Introduction_to_stream.md)
- [filter, map, and flatMap](./3_filter_map,_and_flatMap.md)
