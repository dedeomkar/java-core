# Grouping and Partitioning with Collectors

## Table of Contents
- [1. Core Grouping Concepts](#1-core-grouping-concepts)
  - [1.1 Understanding the Collectors API](#11-understanding-the-collectors-api)
- [2. Grouping Data](#2-grouping-data)
  - [2.1 The `groupingBy` Collector](#21-the-groupingby-collector)
  - [2.2 Object Property Grouping](#22-object-property-grouping)
- [3. Partitioning Data](#3-partitioning-data)
  - [3.1 The `partitioningBy` Collector](#31-the-partitioningby-collector)

---

## 1. Core Grouping Concepts

### 1.1 Understanding the Collectors API

While the core `collect` operation in the Stream API requires three discrete arguments (supplier, accumulator, combiner), managing these manually is often cumbersome. The `Collector` interface encapsulates these into a single abstraction, and the `java.util.stream.Collectors` utility class provides factories for many typical use cases.

> [!NOTE]
> Think of the base `collect` method as buying individual parts to build a car engine, whereas the `Collectors` factory methods are pre-assembled engines ready to be dropped into your vehicle.

- **`groupingBy`**: Organizes data into a `Map` based on a programmer-defined key extracted from elements.
- **`partitioningBy`**: A specialized form of grouping that divides data into precisely two buckets (true/false) using a `Predicate`.

---

## 2. Grouping Data

### 2.1 The `groupingBy` Collector

The `groupingBy` collector accepts a **classifier** function (the key extractor), which determines the key under which each stream element will be filed. By default, the grouped elements are accumulated into a `List`.

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class BasicGrouping {
    public static void main(String[] args) {
        Stream<String> names = Stream.of("Fred", "Jim", "Sheila", "Bill", "Joe", "John");

        // Groups names by their string length
        Map<Integer, List<String>> byLength = names.collect(
            Collectors.groupingBy(String::length)
        );
        
        // Output might be: {3=[Jim, Joe], 4=[Fred, Bill, John], 6=[Sheila]}
        System.out.println(byLength);
    }
}
```

### 2.2 Object Property Grouping

Grouping is incredibly powerful when applied to structured domain objects. You can dynamically construct maps of grouped data strictly from object properties.

> [!TIP]
> This pattern replaces traditional iterative loops that check for key existence, initialize new lists, and add elements sequentially—saving lines of code and reducing error surfaces.

```java
record Customer(String name, String region) {}

public class ObjectGrouping {
    public static void main(String[] args) {
        Stream<Customer> customers = Stream.of(
            new Customer("Alice", "North"),
            new Customer("Bob", "East"),
            new Customer("Charlie", "North")
        );

        // Group Customers by Region
        Map<String, List<Customer>> byRegion = customers.collect(
            Collectors.groupingBy(Customer::region)
        );

        /* Result:
         * North=[Customer[name=Alice, region=North], Customer[name=Charlie, region=North]]
         * East=[Customer[name=Bob, region=East]]
         */
    }
}
```

---

## 3. Partitioning Data

### 3.1 The `partitioningBy` Collector

`partitioningBy` behaves like `groupingBy`, but is restricted to Boolean keys. It accepts a **Predicate** instead of a general classifier function. It is highly optimized for splitting a stream into passing and failing groups.

- **Guaranteed Keys**: The resulting Map will *always* contain keys for both `true` and `false`, even if one or both buckets are empty.

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class PartitioningExample {
    public static void main(String[] args) {
        Stream<String> names = Stream.of(
            "Fred", "Jim", "Bill", "Joe", "John", "Sheila", "Sally", "Tommy"
        );

        // Partition by whether the name length is greater than 4
        Map<Boolean, List<String>> longNames = names.collect(
            Collectors.partitioningBy(name -> name.length() > 4)
        );

        System.out.println("Long names (true): " + longNames.get(true));
        // Output: [Sheila, Sally, Tommy]
        
        System.out.println("Short names (false): " + longNames.get(false));
        // Output: [Fred, Jim, Bill, Joe, John]
    }
}
```

> [!WARNING]
> Because `partitioningBy` explicitly creates `Boolean` keys, the map signature must be `Map<Boolean, List<T>>` (using the wrapper class `Boolean`, not the primitive `boolean`).
