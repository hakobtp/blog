## Downstream Collectors in Streams

```info
Author      Ter-Petrosyan Hakob
```

---

When you group elements in a stream using `groupingBy`, the result is usually a map where the values are lists. But sometimes, you might want to process these lists differently. This is where downstream collectors come in. They let you define what to do with the values after grouping.

## Changing List to Set

By default, `groupingBy` creates a list for each key. But if you prefer a set (which removes duplicates), you can use the `toSet()` collector. 

**For example:**

```java
Map<String, Set<String>> cityNamesByCountry = Stream
    .of("Paris", "Lyon", "Marseille", "Berlin", "Munich")
    .collect(groupingBy(city -> city.substring(0, 1), toSet()));

System.out.println(cityNamesByCountry);
```

Here, we group city names by their first letter and collect them into sets.

## Nested Grouping

You can group elements more than once, creating nested maps. For example, imagine you have cities with their country and language information:

```java
Map<String, Map<String, List<String>>> citiesByCountryAndLanguage = Stream.of(
    new City("Paris", "FR", "French"),
    new City("Lyon", "FR", "French"),
    new City("Munich", "DE", "German"))
.collect(groupingBy(City::getCountry, groupingBy(City::getLanguage)));

```

Now you can get all French-speaking cities in France like this:

```java
citiesByCountryAndLanguage.get("FR").get("French");
```

## Counting Elements

If you want to know how many items are in each group, use the `counting()` collector:

```java
Map<String, Long> citiesPerCountry = Stream
    .of("Paris", "Lyon", "Berlin", "Munich", "Hamburg")
    .collect(groupingBy(city -> city.substring(0, 1), counting()));
```

This counts how many cities start with each letter.

## Summing and Averaging Numbers

Collectors like `summingInt()` or `averagingInt()` let you compute sums or averages of numbers in groups. Suppose you have a City class with population:

```java
Map<String, Double> averagePopulationByCountry = Stream.of(
    new City("Paris", "FR", 2200000),
    new City("Lyon", "FR", 500000),
    new City("Berlin", "DE", 3600000))
.collect(groupingBy(City::getCountry, averagingInt(City::getPopulation)));
```

This gives the average population for each country.

## Finding Maximum or Minimum

Use `maxBy()` and `minBy()` to find the largest or smallest element in each group:

```java
Map<String, Optional<City>> largestCityByCountry = Stream.of(
    new City("Paris", "FR", 2200000),
    new City("Lyon", "FR", 500000),
    new City("Berlin", "DE", 3600000))
.collect(groupingBy(City::getCountry, maxBy(Comparator.comparing(City::getPopulation))));
```

Transforming Collected Results

The collectingAndThen() collector allows a final step after collecting:

Map<Character, Integer> uniqueFirstLetters = Stream.of("apple", "apricot", "banana")
    .collect(groupingBy(s -> s.charAt(0),
           collectingAndThen(toSet(), Set::size)));


This first groups by the first letter, then counts how many unique words start with each letter.

Mapping Values

The mapping() collector applies a function to each element before collecting:

Map<Character, Set<Integer>> wordLengthsByFirstLetter = Stream.of("apple", "apricot", "banana")
    .collect(groupingBy(s -> s.charAt(0),
           mapping(String::length, toSet())));


Here, words are grouped by first letter, and inside each group, we collect their lengths.

Filtering Elements

You can also filter items inside groups using filtering():

Map<String, Set<City>> largeCitiesByCountry = Stream.of(
    new City("Paris", "FR", 2200000),
    new City("Lyon", "FR", 500000),
    new City("Marseille", "FR", 860000)
)
.collect(groupingBy(City::getCountry,
           filtering(c -> c.getPopulation() > 1000000, toSet())));


Only cities with populations over 1 million are included.

Teeing Collector for Multiple Results

Sometimes you need two results at once. The teeing() collector helps:

record Pair<S, T>(S first, T second) {}

Pair<List<String>, Double> result = Stream.of(
    new City("Paris", "FR", 2200000),
    new City("Lyon", "FR", 500000)
)
.collect(teeing(
    mapping(City::getName, toList()),
    averagingDouble(City::getPopulation),
    (names, avg) -> new Pair<>(names, avg)
));


Here, we get a list of city names and the average population together.
---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Grouping and Partitioning](./8_Grouping_and_Partitioning.md)