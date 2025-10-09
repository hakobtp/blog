## Collecting Stream Elements into Maps

```info
Author      Ter-Petrosyan Hakob
```

---

When you work with Java Streams, you often want to turn a stream of objects into a map — a structure that lets you find values by a key.
The `Collectors.toMap()` method is a powerful way to do this.

Imagine you have a `Stream<Student>` and you want a map where the key is each student’s `ID` and the value is the student’s `name`.

```java
public record Student(int id, String name) {}

Map<Integer, String> idToName = students
   .collect(Collectors.toMap(Student::id, Student::name));
```

The first function (`Student::id`) creates the map’s keys.
The second function (`Student::name`) creates the map’s values.

If you want the whole object as the value (not just one field), use `Function.identity()` — it simply returns the same object.

```java
Map<Integer, Student> idToStudent = students
   .collect( Collectors.toMap(Student::id, Function.identity()));
```

Now each `ID` maps to its full `Student` record.

## Handling Duplicate Keys

If two elements have the same key, Java doesn’t know which one to keep — so it throws an `IllegalStateException`.
To fix this, you can add a merge function (a third argument) to decide what happens when keys repeat.

**Example 1:** Suppose two students have the same `ID` (a mistake in data). You can choose to keep the first one, the last one, or merge them.

```java
Map<Integer, Student> idToStudent = students.collect(
    Collectors.toMap(
        Student::id,
        Function.identity(),
        (existing, replacement) -> existing // keep the first one
    )
);
```

You could also merge their names or scores if that makes sense for your case.

**Example 2:** Let’s say you have a list of products. You want a map where the key is the `category`, and the value is a set of product `names` in that category.

```java
record Product(String category, String name) {}

Map<String, Set<String>> productsByCategory = products.collect(
    Collectors.toMap(
        Product::category,
        p -> new HashSet<>(Set.of(p.name())),
        (set1, set2) -> {
            set1.addAll(set2);
            return set1;
        }
    )
);
```

If a category appears several times, the merge function combines the sets — so you end up with all products in that category.


## Choosing the Map Type

By default, `toMap()` uses a `HashMap`, but you can choose another type — for example, a `TreeMap`, which keeps keys sorted.

```java
Map<Integer, Student> sortedStudents = students.collect(
    Collectors.toMap(
        Student::id,
        Function.identity(),
        (a, b) -> a,
        TreeMap::new
    )
);
```

Now your map is automatically sorted by student `ID`.

## Concurrent Maps

When you process a parallel stream, multiple threads work at the same time.
To collect results safely and efficiently, use `Collectors.toConcurrentMap()` — it produces a thread-safe `ConcurrentHashMap`.

```java
ConcurrentMap<Integer, Student> concurrentMap = students.parallelStream().collect(
    Collectors.toConcurrentMap(Student::id, Function.identity())
);
```

This avoids merging multiple maps at the end and is faster for large data sets.

---------------------------------------------------------------------------

**java.util.stream.Collectors**

These methods return a collector that creates a map, an unmodifiable map, or a concurrent map.
The `keyMapper` and `valueMapper` functions are used for each element in the stream to produce a `key` and a `value` for the map.

If two elements have the same `key`, Java throws an `IllegalStateException` by default.
You can avoid this error by adding a `mergeFunction`, which decides how to combine the two `values` that share the same `key`.

Normally, the result is a `HashMap` or `ConcurrentHashMap`, but you can also give a `mapSupplier` to create the specific type of map you want.


```java
static <T,K,U> Collector<T,?,Map<K,U>> toMap(
   Function<? super T,? extends K> keyMapper, 
   Function<? super T,? extends U> valueMapper)

static <T,K,U> Collector<T,?,Map<K,U>> toMap(
   Function<? super T,? extends K> keyMapper, 
   Function<? super T,? extends U> valueMapper, 
   BinaryOperator<U> mergeFunction)

static <T,K,U,M extends Map<K,U>> Collector<T,?,M> toMap(
   Function<? super T,? extends K> keyMapper, 
   Function<? super T,? extends U> valueMapper, 
   BinaryOperator<U> mergeFunction, 
   Supplier<M> mapSupplier)

static <T,K,U> Collector<T,?,Map<K,U>> toUnmodifiableMap(
   Function<? super T,? extends K> keyMapper, 
   Function<? super T,? extends U> valueMapper) 

static <T,K,U> Collector<T,?,Map<K,U>> toUnmodifiableMap(
   Function<? super T,? extends K> keyMapper, 
   Function<? super T,? extends U> valueMapper, 
   BinaryOperator<U> mergeFunction) 10

static <T,K,U> Collector<T,?,ConcurrentMap<K,U>> toConcurrentMap(
   Function<? super T,? extends K> keyMapper, 
   Function<? super T,? extends U> valueMapper)

static <T,K,U> Collector<T,?,ConcurrentMap<K,U>> toConcurrentMap(
   Function<? super T,? extends K> keyMapper, 
   Function<? super T,? extends U> valueMapper, 
   BinaryOperator<U> mergeFunction)

static <T,K,U,M extends ConcurrentMap<K,U>> Collector<T,?,M> toConcurrentMap(
   Function<? super T,? extends K> keyMapper, 
   Function<? super T,? extends U> valueMapper,
   BinaryOperator<U> mergeFunction, 
   Supplier<M> mapSupplier)

```

---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Collecting Results from a Stream](./6_Collecting_Results_from_a_Stream.md)
- [Grouping and Partitioning](./8_Grouping_and_Partitioning.md)
