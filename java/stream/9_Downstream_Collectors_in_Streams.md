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

## Transforming Collected Results

The `collectingAndThen()` collector allows a final step after collecting:

```java
Map<Character, Integer> uniqueFirstLetters = Stream
    .of("apple", "apricot", "banana")
    .collect(groupingBy(s -> s.charAt(0), collectingAndThen(toSet(), Set::size)));
```

This first groups by the first letter, then counts how many unique words start with each letter.

## Mapping Values

The `mapping()` collector applies a function to each element before collecting:

```java
Map<Character, Set<Integer>> wordLengthsByFirstLetter = Stream
    .of("apple", "apricot", "banana")
    .collect(groupingBy(s -> s.charAt(0), mapping(String::length, toSet())));
```

Here, words are grouped by first letter, and inside each group, we collect their lengths.

## Filtering Elements

You can also filter items inside groups using `filtering()`:

```java
Map<String, Set<City>> largeCitiesByCountry = Stream.of(
    new City("Paris", "FR", 2200000),
    new City("Lyon", "FR", 500000),
    new City("Marseille", "FR", 860000))
.collect(groupingBy(City::getCountry, filtering(c -> c.getPopulation() > 1000000, toSet())));
```

Only cities with populations over 1 million are included.

## Teeing Collector for Multiple Results

Sometimes you need two results at once. The `teeing()` collector helps:

```java
record Pair<S, T>(S first, T second) {}

Pair<List<String>, Double> result = Stream.of(
    new City("Paris", "FR", 2200000),
    new City("Lyon", "FR", 500000))
.collect(teeing(
    mapping(City::getName, toList()),
    averagingDouble(City::getPopulation), 
    (names, avg) -> new Pair<>(names, avg)
));
```

Here, we get a list of city names and the average population together.


## Key Advice

- Use downstream collectors mainly with `groupingBy` or `partitioningBy`.
- For simpler tasks, `map()`, `reduce()`, `count()`, `max()`, or `min()` are often enough.
- Combining collectors is powerful but can make code complex, so use them wisely.


## Full Example

```java
package collecting;

import static java.util.stream.Collectors.*;

import java.util.*;
import java.util.stream.*;

/**
 * Example of using downstream collectors in Java Streams.
 * Shows grouping, mapping, counting, averaging, and more.
 */
public class DownstreamCollectorsExample {

    // A simple City class with name, country, and population
    public record City(String name, String country, int population) {}

    public static void main(String[] args) {

        // Example cities
        List<City> cities = List.of(
            new City("Paris", "FR", 2200000),
            new City("Lyon", "FR", 500000),
            new City("Marseille", "FR", 860000),
            new City("Berlin", "DE", 3600000),
            new City("Munich", "DE", 1500000),
            new City("Hamburg", "DE", 1800000)
        );

        // 1. Group cities by country and collect names into a set
        Map<String, Set<String>> cityNamesByCountry = cities.stream()
            .collect(groupingBy(City::country,
                    mapping(City::name, toSet())));
        System.out.println("City names by country: " + cityNamesByCountry);

        // 2. Count how many cities are in each country
        Map<String, Long> cityCountsByCountry = cities.stream()
            .collect(groupingBy(City::country, counting()));
        System.out.println("Number of cities by country: " + cityCountsByCountry);

        // 3. Compute average population per country
        Map<String, Double> averagePopulationByCountry = cities.stream()
            .collect(groupingBy(City::country, averagingInt(City::population)));
        System.out.println("Average population by country: " + averagePopulationByCountry);

        // 4. Find the largest city in each country
        Map<String, Optional<City>> largestCityByCountry = cities.stream()
            .collect(groupingBy(City::country,
                    maxBy(Comparator.comparing(City::population))));
        System.out.println("Largest city by country: " + largestCityByCountry);

        // 5. Filter only large cities (>1 million) and group by country
        Map<String, Set<City>> largeCitiesByCountry = cities.stream()
            .collect(groupingBy(City::country,
                    filtering(c -> c.population() > 1000000, toSet())));
        System.out.println("Large cities by country: " + largeCitiesByCountry);

        // 6. Using teeing to collect both city names and average population
        record Pair<S, T>(S first, T second) {}

        Pair<List<String>, Double> countrySummary = cities.stream()
            .filter(c -> c.country.equals("DE"))
            .collect(teeing(
                mapping(City::name, toList()),            // Collect names
                averagingDouble(City::population),        // Compute average population
                (names, avg) -> new Pair<>(names, avg)    // Combine results
            ));
        System.out.println("Germany city summary: " + countrySummary);

        // 7. Nested grouping: group cities by country, then by population category
        Map<String, Map<String, List<City>>> citiesByCountryAndSize = cities.stream()
            .collect(groupingBy(City::country,
                    groupingBy(c -> c.population > 1000000 ? "Large" : "Small")));
        System.out.println("Cities grouped by country and size: " + citiesByCountryAndSize);
    }
}
````




---

- [Home](./../../README.md)
- [Java Tutorials](./../tutorials.md)
- [Grouping and Partitioning](./8_Grouping_and_Partitioning.md)