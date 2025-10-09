Collecting into Maps
Suppose you have a Stream<Person> and want to collect the elements into a map so that later you can look up people by their ID. Call Collectors.toMap with two functions that produce the map's keys and values. For example,

public record Person(int id, String name) {}
. . .
Map<Integer, String> idToName = people.collect(
   Collectors.toMap(Person::id, Person::name));
In the common case when the values should be the actual elements, use Function.identity() for the second function.

Map<Integer, Person> idToPerson = people.collect(
   Collectors.toMap(Person::id, Function.identity()));
If there is more than one element with the same key, there is a conflict, and the collector will throw an IllegalStateException. You can override that behavior by supplying a third function that resolves the conflict and determines the value for the key, given the existing and the new value. Your function could return the existing value, the new value, or a combination of them.

Here, we construct a map that contains, for each language in the available locales, as key its name in your default locale (such as "German"), and as value its localized name (such as "Deutsch").

Map<String, String> languageNames = Locale.availableLocales().collect(
   Collectors.toMap(
      Locale::getDisplayLanguage, 
      loc -> loc.getDisplayLanguage(loc),
      (existingValue, newValue) -> existingValue));
We don't care that the same language might occur twice (for example, German in Germany and in Switzerland), so we just keep the first entry.

In this chapter, I use the Locale class as a source of an interesting data set. See Chapter 7 for more information on working with locales.

Now suppose we want to know all languages in a given country. Then we need a Map<String, Set<String>>. For example, the value for "Switzerland" is the set [French, German, Italian]. At first, we store a singleton set for each language. Whenever a new language is found for a given country, we form the union of the existing and the new set.

Map<String, Set<String>> countryLanguageSets = Locale.availableLocales().collect(
   Collectors.toMap(
      Locale::getDisplayCountry,
      l -> Collections.singleton(l.getDisplayLanguage()),
      (a, b) ->
         { // Union of a and b
            var union = new HashSet<String>(a); 
            union.addAll(b);
            return union;
         }));
You will see a simpler way of obtaining this map in the next section.

If you want a TreeMap, supply the constructor as the fourth argument. You must provide a merge function. Here is one of the examples from the beginning of the section, now yielding a TreeMap:

Map<Integer, Person> idToPerson = people.collect(
   Collectors.toMap(
      Person::id,
      Function.identity(),
      (existingValue, newValue) -> { throw new IllegalStateException(); },
      TreeMap::new));
For each of the toMap methods, there is an equivalent toConcurrentMap method that yields a concurrent map. A single concurrent map is used in the parallel collection process. When used with a parallel stream, a shared map is more efficient than merging maps. Note that elements are no longer collected in stream order, but that doesn't usually make a difference.


java.util.stream.Collectors 8

static <T,K,U> Collector<T,?,Map<K,U>> toMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper)

static <T,K,U> Collector<T,?,Map<K,U>> toMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper, BinaryOperator<U> mergeFunction)

static <T,K,U,M extends Map<K,U>> Collector<T,?,M> toMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper, BinaryOperator<U> mergeFunction, Supplier<M> mapSupplier)

static <T,K,U> Collector<T,?,Map<K,U>> toUnmodifiableMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper) 10

static <T,K,U> Collector<T,?,Map<K,U>> toUnmodifiableMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper, BinaryOperator<U> mergeFunction) 10

static <T,K,U> Collector<T,?,ConcurrentMap<K,U>> toConcurrentMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper)

static <T,K,U> Collector<T,?,ConcurrentMap<K,U>> toConcurrentMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper, BinaryOperator<U> mergeFunction)

static <T,K,U,M extends ConcurrentMap<K,U>> Collector<T,?,M> toConcurrentMap(Function<? super T,? extends K> keyMapper, Function<? super T,? extends U> valueMapper, BinaryOperator<U> mergeFunction, Supplier<M> mapSupplier)

yield a collector that produces a map, unmodifiable map, or concurrent map. The keyMapper and valueMapper functions are applied to each collected element, yielding a key/value entry of the resulting map. By default, an IllegalStateException is thrown when two elements give rise to the same key. You can instead supply a mergeFunction that merges values with the same key. By default, the result is a HashMap or ConcurrentHashMap. You can instead supply a mapSupplier that yields the desired map instance.