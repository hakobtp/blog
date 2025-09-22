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

------



The call stream.skip(n) does the exact opposite. It discards the first n elements. This is handy in our book reading example where, due to the way the split method works, the first element is an unwanted empty string. We can make it go away by calling skip:

Stream<String> words = Stream.of(contents.split("\\PL+")).skip(1);
The stream.takeWhile(predicate) call takes all elements from the stream while the predicate is true, and then stops.

For example, suppose we use the graphemeClusters method of the preceding section to split a string into characters, and we want to collect all initial digits. The takeWhile method can do this:

Stream<String> initialDigits = graphemeClusters(str).takeWhile(
   s -> "0123456789".contains(s));
The dropWhile method does the opposite, dropping elements while a condition is true and yielding a stream of all elements starting with the first one for which the condition was false. For example,

Stream<String> withoutInitialWhiteSpace = graphemeClusters(str).dropWhile(
   s -> s.strip().length() == 0);
You can concatenate two streams with the static concat method of the Stream class:

Stream<String> combined = Stream.concat(
   graphemeClusters("Hello"), graphemeClusters("World"));
   // Yields the stream ["H", "e", "l", "l", "o", "W", "o", "r", "l", "d"]
Of course, the first stream should not be infinite—otherwise the second one wouldn’t ever get a chance.


## Other Stream Transformations
The distinct method returns a stream that yields elements from the original stream, in the same order, except that duplicates are suppressed. The duplicates need not be adjacent.

Stream<String> uniqueWords
   = Stream.of("merrily", "merrily", "merrily", "gently").distinct();
   // Only one "merrily" is retained
For sorting a stream, there are several variations of the sorted method. One works for streams of Comparable elements, and another accepts a Comparator. Here, we sort strings so that the longest ones come first:

Stream<String> longestFirst
   = words.stream().sorted(Comparator.comparing(String::length).reversed());
As with all stream transformations, the sorted method yields a new stream whose elements are the elements of the original stream in sorted order.

Of course, you can sort a collection without using streams. The sorted method is useful when the sorting process is part of a stream pipeline.

Finally, the peek method yields another stream with the same elements as the original, but a function is invoked every time an element is retrieved. That is handy for debugging:

Object[] powers = Stream.iterate(1.0, p -> p * 2)
   .peek(e -> System.out.println("Fetching " + e))
   .limit(20).toArray();
When an element is actually accessed, a message is printed. This way you can verify that the infinite stream returned by iterate is processed lazily.

When you use a debugger to debug a stream computation, you can set a breakpoint in a method that is called from one of the transformations. With most IDEs, you can also set breakpoints in lambda expressions. If you just want to know what happens at a particular point in the stream pipeline, add

.peek(x ->
   {
      return;
   })
and set a breakpoint on the second line.


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