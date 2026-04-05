# Parallel Stream Operation

## Table of Contents
- [1. Core Concepts](#1-core-concepts)
  - [1.1 Creating Parallel Streams](#11-creating-parallel-streams)
  - [1.2 Associativity and Thread Safety](#12-associativity-and-thread-safety)
- [2. Stream Ordering](#2-stream-ordering)
  - [2.1 Ordered vs Unordered Sources](#21-ordered-vs-unordered-sources)
  - [2.2 Memory Implications](#22-memory-implications)
- [3. Switching Execution Modes](#3-switching-execution-modes)
  - [3.1 The Last Invocation Rule](#31-the-last-invocation-rule)

---

## 1. Core Concepts

Parallel stream mode executes groups of computations on separate threads, significantly reducing execution time for heavy workloads, provided the underlying operations allow for logical partitioning.

> [!NOTE]
> Think of parallel streams like a team of workers assembling a car. Instead of one person building the entire car sequentially, multiple workers build different parts simultaneously (engine, doors, wheels) and assemble them at the end.

### 1.1 Creating Parallel Streams

You can activate parallel stream execution using two main APIs:

- `.parallelStream()` on any `Collection`.
- `.parallel()` on an existing stream (which extends `BaseStream`).

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);

// Option 1: From a collection
int sum1 = numbers.parallelStream()
                  .reduce(0, Integer::sum);

// Option 2: From an existing stream
int sum2 = numbers.stream()
                  .parallel()
                  .reduce(0, Integer::sum);
```

You can check if a stream is running in parallel using `isParallel()`, though it can be unreliable if the stream has finished or is already executing.

### 1.2 Associativity and Thread Safety

To run safely in parallel, the operations must be **associative** and generally pure functions.

- **Associativity**: The grouping of operations does not matter (e.g., `(1 + 2) + (3 + 4)` produces the same result regardless of grouping).
- **Thread Safety**: Operations should not mutate shared external objects. 

> [!CAUTION]
> Mutating external variables inside a parallel stream leads to race conditions. Each thread might attempt to update the same variable simultaneously, resulting in lost updates and unpredictable behavior.

```java
// DANGEROUS: Unsafe mutation in parallel stream
int[] counter = {0};
IntStream.range(0, 1000).parallel().forEach(i -> {
    counter[0]++; // Race condition! Thread-unsafe.
});

// SAFE: Using a thread-safe terminal operation
long count = IntStream.range(0, 1000).parallel().count();
```

The standard `collect()` operation handles thread safety internally by ensuring each thread receives exclusive ownership of its mutable result container (like a local accumulator), aggregating them safely at the end. Collectors like `Collectors.groupingByConcurrent()` are explicitly designed to maintain thread safety while optimizing parallel performance.

---

## 2. Stream Ordering

### 2.1 Ordered vs Unordered Sources

The ordering of a stream dictates whether the sequence of elements matters during execution.

- **Ordered Sources**: Collections like `List` or streams created via `Stream.iterate()` maintain order.
- **Unordered Sources**: Collections like `HashSet` or streams created via `Stream.generate()` do not guarantee any order.

If both the source and the terminal operation are ordered, the stream runs in **ordered mode by default**, even when running in parallel. This implies operations do not need to be commutative for the result to remain correct.

### 2.2 Memory Implications

> [!WARNING]
> Maintaining order in parallel streams incurs high memory overhead. If an item finishes processing before its preceding elements, the system must buffer it in memory to maintain the correct final sequence.

To disable ordering on a parallel stream, use the `.unordered()` method. This allows threads to yield elements immediately, skipping the expensive memory coordination.

```java
List<String> largeData = fetchLargeData();

// Maintains order, but could be memory-intensive in parallel
largeData.parallelStream()
         .map(String::toUpperCase)
         .toList();

// Disables ordering for maximum performance/memory efficiency
largeData.parallelStream()
         .unordered()
         .map(String::toUpperCase)
         .toList();
```

---

## 3. Switching Execution Modes

### 3.1 The Last Invocation Rule

When chained in a pipeline, calls to `.parallel()` and `.sequential()` dictate the execution mode for the entire sequence. **You cannot shard the execution into parallel and sequential phases.** 

The mode specified by the **last invocation** will dictate how the entire pipeline executes.

```java
// Entire stream runs sequentially due to the last call
IntStream.range(1, 100)
         .parallel()
         .map(x -> x * 2)
         .sequential() // Overrides the previous .parallel() call
         .forEach(System.out::println);
```
