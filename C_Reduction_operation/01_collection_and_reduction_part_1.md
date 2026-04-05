# Collection and Reduction: Core Concepts

## Table of Contents
- [1. The Fundamentals of Reduction](#1-the-fundamentals-of-reduction)
  - [1.1 Binary Operators and Accumulation](#11-binary-operators-and-accumulation)
  - [1.2 Handling Empty Streams](#12-handling-empty-streams)
- [2. Complex Result Types](#2-complex-result-types)
  - [2.1 When Output Differs from Input](#21-when-output-differs-from-input)
  - [2.2 The Bifunction Alternative](#22-the-bifunction-alternative)
- [3. The Stream API Reduce Methods](#3-the-stream-api-reduce-methods)
  - [3.1 Single-Argument Reduce](#31-single-argument-reduce)
  - [3.2 Two-Argument Reduce](#32-two-argument-reduce)
  - [3.3 Three-Argument Reduce](#33-three-argument-reduce)
- [4. Associativity and Parallel Processing](#4-associativity-and-parallel-processing)
  - [4.1 Why Associativity Matters](#41-why-associativity-matters)

---

## 1. The Fundamentals of Reduction

### 1.1 Binary Operators and Accumulation

At its core, reduction takes a sequence of items and folds them down into a single item using a **binary operator**. A binary operator takes two inputs of a given type and produces one output of that same type. The result of one operation becomes the input to the next.

> [!NOTE]
> Think of reduction like rolling a snowball down a hill. The snowball (the accumulator or intermediate result) picks up more snow (the next element in the stream) as it rolls, growing larger until you reach the bottom of the hill with a single, massive snowball (the final result).

```java
List<Integer> numbers = List.of(1, 2, 3, 4);

// Repeated application of a binary operator (addition)
// 1 + 2 = 3
// 3 + 3 = 6
// 6 + 4 = 10
Integer sum = numbers.stream()
                     .reduce((subtotal, element) -> subtotal + element)
                     .orElse(0); 
```

### 1.2 Handling Empty Streams

When reducing a stream, you must consider the case where the stream contains no elements. If you attempt to sum zero items, naturally the result is zero. But for some operations (like finding the maximum value), there is no sensible default.

To represent the possibility of having no meaningful result, the standard reduction operation returns an `Optional`.

```java
List<String> emptyList = Collections.emptyList();

// Returns an empty Optional because there are no elements to reduce
Optional<String> concatenated = emptyList.stream()
                                         .reduce((s1, s2) -> s1 + s2);

System.out.println(concatenated.isPresent()); // false
```

---

## 2. Complex Result Types

### 2.1 When Output Differs from Input

Sometimes the data type you are accumulating into is different from the type of the elements in your stream. For example, when calculating an average, the input is a single number, but the intermediate state needs to track both the **sum** and the **count**.

> [!IMPORTANT]
> If a binary operator requires the output type to match the input type, you cannot directly use it to reduce data into a completely different format or structure (like a custom container holding multiple state variables).

To calculate an average, you can map the inputs into intermediate objects containing the sum and count, and reduce *those* objects instead.

```java
record AverageTracker(double sum, int count) {
    public AverageTracker merge(AverageTracker other) {
        return new AverageTracker(this.sum + other.sum, this.count + other.count);
    }
    
    public double getAverage() {
        return count == 0 ? 0 : sum / count;
    }
}

List<Double> values = List.of(1.0, 2.0, 3.0, 4.0);

// Map to our intermediate representation, then reduce
double average = values.stream()
    .map(v -> new AverageTracker(v, 1))
    .reduce(AverageTracker::merge)
    .map(AverageTracker::getAverage)
    .orElse(0.0);
```

### 2.2 The Bifunction Alternative

Instead of mapping every element to an intermediate object upfront, we can define an initial state (e.g., sum=0, count=0) and incrementally merge stream items into it. The operation that combine the accumulator (Result Type) with a new element (Stream Type) is a **BiFunction**, completely decoupling the output type from the input.

```java
// Using BiFunction conceptually (Note: this reflects the structure of the 3-argument reduce)
BiFunction<AverageTracker, Double, AverageTracker> accumulator = 
    (tracker, value) -> new AverageTracker(tracker.sum() + value, tracker.count() + 1);

// Applying the BiFunction directly in a stream reduction
AverageTracker result = values.stream()
    .reduce(
        new AverageTracker(0, 0),             // Identity
        accumulator,                          // The BiFunction
        AverageTracker::merge                 // Combiner for parallel streams
    );

System.out.println("Average: " + result.getAverage());
```

---

## 3. The Stream API Reduce Methods

The Stream API provides three distinct overloaded `reduce` methods matching the scenarios discussed.

### 3.1 Single-Argument Reduce

Used when the stream elements and the result are of the exact same type. Because the stream could be empty, it returns an `Optional`.

```java
// Optional<T> reduce(BinaryOperator<T> accumulator);
Optional<Integer> maxVal = Stream.of(5, 12, 3)
                                 .reduce(Integer::max);
```

### 3.2 Two-Argument Reduce

Used when you can provide a "zero-like" identity value. If the stream is empty, the identity value is returned, avoiding the need for an `Optional`.

```java
// T reduce(T identity, BinaryOperator<T> accumulator);
Integer product = Stream.of(1, 2, 3, 4)
                        .reduce(1, (a, b) -> a * b); // 1 is the identity for multiplication
```

### 3.3 Three-Argument Reduce

Used for complex reductions where the result type is different from the stream type. This is particularly relevant when parallelizing.

- **Identity**: The initial state (Result Type `U`).
- **Accumulator**: Merges a stream element (`T`) into the intermediate result (`U`), returning a new intermediate result (`U`).
- **Combiner**: Merges two intermediate results (`U`, `U` -> `U`).

```java
// <U> U reduce(U identity, BiFunction<U, ? super T, U> accumulator, BinaryOperator<U> combiner);
AverageTracker finalTracker = values.stream()
    .reduce(
        new AverageTracker(0, 0), // Identity
        (tracker, value) -> new AverageTracker(tracker.sum() + value, tracker.count() + 1), // Accumulator
        AverageTracker::merge // Combiner
    );
```

---

## 4. Associativity and Parallel Processing

### 4.1 Why Associativity Matters

When processing operations sequentially from left to right, we get a reliable result. But if we want to divide the work across multiple threads (parallel processing), we must group the elements differently.

> [!TIP]
> Think of associativity like splitting a deck of cards to be sorted by two different dealers. Dealer A sorts the first half, Dealer B sorts the second half, and finally, you merge their pre-sorted stacks together.

If the operation is **associative** (e.g., `(a + b) + c = a + (b + c)`), grouping operations like this will not affect the final outcome. Things like addition and string concatenation are associative, making them safe for parallel stream reductions.

However, non-associative operations, or operations that rely on side effects, can result in race conditions, corrupted data, or outright incorrect results when executed simultaneously across different threads.
