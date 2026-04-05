# Downstream Operations with Collectors

## Table of Contents
- [1. Core Concepts](#1-core-concepts)
  - [1.1 What is a Downstream Collector?](#11-what-is-a-downstream-collector)
  - [1.2 The Two-Argument `groupingBy`](#12-the-two-argument-groupingby)
- [2. Common Downstream Collectors](#2-common-downstream-collectors)
  - [2.1 Transforming Elements (`mapping`)](#21-transforming-elements-mapping)
  - [2.2 Aggregating Results (`joining` and `counting`)](#22-aggregating-results-joining-and-counting)
- [3. Advanced Downstream Operations](#3-advanced-downstream-operations)
  - [3.1 Primitive Averaging](#31-primitive-averaging)
  - [3.2 Filtering and FlatMapping](#32-filtering-and-flatmapping)

---

## 1. Core Concepts

### 1.1 What is a Downstream Collector?

When using collectors like `groupingBy`, the default behavior is to aggregate the items into a `List`. However, the values associated with those keys form a sequence of their own. A **downstream collector** allows you to process these intermediate groups in a stream-like fashion before yielding the final value for the map.

> [!NOTE]
> Think of `groupingBy` as a mailroom clerk who sorts packages into bins based on a destination ZIP code (the key). The downstream collector is the specialized worker at each bin who decides what to do with the packages inside it—whether to stack them in a list, count them, extract just the label (`mapping`), or combine them into one big box (`joining`).

### 1.2 The Two-Argument `groupingBy`

The standard `Collectors.groupingBy` takes a single classifier function and defaults to a `toList()` downstream collector. To apply custom downstream operations, you must use the overloaded version that accepts a second `Collector` argument.

```java
public record Student(String name, String letterGrade, double percent) {}

List<Student> students = List.of(
    new Student("Sheila", "A", 95.0),
    new Student("John", "A", 92.0),
    new Student("Fred", "B", 85.0)
);

// Standard 1-argument groupingBy
Map<String, List<Student>> groupedByGrade = students.stream()
    .collect(Collectors.groupingBy(Student::letterGrade));
```

---

## 2. Common Downstream Collectors

### 2.1 Transforming Elements (`mapping`)

The `Collectors.mapping` collector behaves similarly to the `Stream.map` operation. It transforms each element in a group before passing it to yet another downstream collector. Because `mapping` requires a downstream collector of its own, you are forced to specify how the mapped values are finally accumulated (e.g., `toList()`).

```java
// Extracting just the names of the students in each grade group
Map<String, List<String>> namesByGrade = students.stream()
    .collect(Collectors.groupingBy(
        Student::letterGrade,
        Collectors.mapping(Student::name, Collectors.toList())
    ));
```

> [!IMPORTANT]
> The default "collect to list" behavior of `groupingBy` is lost as soon as you provide a custom downstream collector. You must explicitly provide `Collectors.toList()` (or another terminal collector) as the final step of a `mapping` operation.

### 2.2 Aggregating Results (`joining` and `counting`)

If the elements in a group are `String` objects, you can use `Collectors.joining` to concatenate them. `joining` has three variants: without arguments, with a delimiter, or with a delimiter, prefix, and suffix.

```java
// Joining the extracted names into a single comma-separated string
Map<String, String> joinedNamesByGrade = students.stream()
    .collect(Collectors.groupingBy(
        Student::letterGrade,
        Collectors.mapping(Student::name, Collectors.joining(", "))
    ));

// Expected Output for "A": {A=Sheila, John}
```

The `Collectors.counting` collector simply counts the number of elements in each group, returning a `Map` with `Long` values.

```java
// Counting students per grade
Map<String, Long> countByGrade = students.stream()
    .collect(Collectors.groupingBy(
        Student::letterGrade, 
        Collectors.counting()
    ));
```

---

## 3. Advanced Downstream Operations

### 3.1 Primitive Averaging

While primitive streams (`IntStream`, `DoubleStream`) have built-in `average()` methods, standard object streams do not. When working inside a `groupingBy` operation, you cannot easily transition to a primitive stream. Instead, you use specialized downstream collectors like `averagingInt`, `averagingLong`, and `averagingDouble`.

```java
// Calculating the average score for each grade group
Map<String, Double> averageScoreByGrade = students.stream()
    .collect(Collectors.groupingBy(
        Student::letterGrade,
        Collectors.averagingDouble(Student::percent)
    ));
```

### 3.2 Filtering and FlatMapping

Just like `mapping`, the `Collectors.filtering` and `Collectors.flatMapping` collectors allow you to perform stream-like operations inside a reduction step. 

- **`filtering(Predicate, Collector)`**: Only passes elements matching the predicate downstream.
- **`flatMapping(Function, Collector)`**: Flattens nested collections or streams for each element before passing the individual items downstream.

> [!TIP]
> `filtering` as a downstream collector often yields different results than stream-level `filter`. A stream-level filter outright removes the element, so properties that do not match simply won't appear as keys in the resulting map. A downstream `filtering` still evaluates the key for the map but may associate it with an empty collection.
