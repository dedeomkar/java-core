# Collection and reduction, part 2

## Table of Contents
- [1. The Need for Mutable Reduction](#1-the-need-for-mutable-reduction)
  - [1.1 Limitations of Pure Functions](#11-limitations-of-pure-functions)
  - [1.2 Safe Mutation](#12-safe-mutation)
- [2. The Collection Mechanism](#2-the-collection-mechanism)
  - [2.1 Building the Pipeline](#21-building-the-pipeline)
  - [2.2 The Collect Method](#22-the-collect-method)
- [3. Collect vs Reduce](#3-collect-vs-reduce)
  - [3.1 Key Differences](#31-key-differences)

---

## 1. The Need for Mutable Reduction

### 1.1 Limitations of Pure Functions

- Using a pure function for reduction creates a **new** result object for every execution. 
- In operations involving complex intermediate states, memory allocation, initialization, and Garbage Collection (GC) can become prohibitively expensive.

> [!NOTE] 
> Imagine pouring a glass of water into a new, slightly larger glass every time you want to add a drop. This is what pure functions do. Mutable reduction is like keeping the same glass and just adding drops to it—far more efficient.

### 1.2 Safe Mutation

- As an alternative to pure functions, we can mutate an intermediate result object.
- **Thread Safety**: This is not unsafe or error-prone if the mutated object is kept within a tight scope and not shared across threads.
- **Grouping/Parallelism**: If using associativity to group operations (e.g., across multiple CPUs), each group **must** have its own distinct result object to prevent race conditions.
- A final mutating aggregation is required at the end to merge these independent result objects.

---

## 2. The Collection Mechanism

### 2.1 Building the Pipeline

To execute a mutable reduction (a collection), we need to teach the system how to manage our intermediate states:

1. **Empty Result Provider**: We need to instruct the system how to build empty result objects (e.g., an object tracking sum=0, count=0). This requires a `Supplier`.
2. **Accumulation Operation**: The operation that updates the state using incoming stream data. This is a `BiConsumer` of the result type and stream type. It returns `void` because it updates in-place.
3. **Merging Operation**: To combine grouped intermediate results (e.g., combining CPU 1's tally with CPU 2's tally). This is a `BiConsumer` taking two result objects. 

> [!IMPORTANT]
> The target of the merging operation is critical. Data from the second argument must be poured into the first argument. Since both arguments are of the same type, the compiler cannot enforce correct ordering—it is entirely up to the developer to avoid logic bugs.

### 2.2 The Collect Method

The Stream API provides a `collect` method that implements this architecture verbatim.

- **Type Parameters**: Works on a stream of type `T` to produce a result of type `R`.
- **Arguments**:
  1. `Supplier<R>` (Supplier)
  2. `BiConsumer<R, ? super T>` (Accumulator)
  3. `BiConsumer<R, R>` (Combiner)

```java
public class AverageTracker {
    private double sum = 0;
    private int count = 0;

    public void accept(double value) {
        this.sum += value;
        this.count++;
    }

    public void combine(AverageTracker other) {
        this.sum += other.sum;
        this.count += other.count;
    }

    public double getAverage() {
        return count == 0 ? 0 : sum / count;
    }
}

// ... inside a method ...
List<Double> numbers = List.of(1.0, 2.0, 3.0, 4.0);

AverageTracker result = numbers.stream()
    .collect(
        AverageTracker::new,          // 1. Supplier
        AverageTracker::accept,       // 2. Accumulator 
        AverageTracker::combine       // 3. Combiner
    );

System.out.println("Average: " + result.getAverage()); // Average: 2.5
```

---

## 3. Collect vs Reduce

### 3.1 Key Differences

While both `collect` and `reduce` can take three arguments, their operational paradigms are fundamentally different:

| Paradigm | `reduce` (3 arguments) | `collect` (3 arguments) |
| :--- | :--- | :--- |
| **First Argument** | An **instance** of the identity object. | A **`Supplier`** of the result type. Allows system to create as many intermediate objects as needed. |
| **Second Argument** | `BiFunction<U, ? super T, U>` that returns a **new** result object. | `BiConsumer<R, ? super T>` returning `void`. Mutates existing result in-place. |
| **Third Argument** | `BinaryOperator<U>` returning a **new** merged object. | `BiConsumer<R, R>` returning `void`. Modifies the first object by including data from the second. |

```java
// Example: String Concatenation 
Stream<String> stream = Stream.of("A", "B", "C");

// REDUCE (Immutable, creates new strings)
String reduced = stream.reduce(
    "",
    (partialString, element) -> partialString + element,
    (str1, str2) -> str1 + str2
);

// COLLECT (Mutable, re-uses StringBuilder)
StringBuilder collected = stream.collect(
    StringBuilder::new,
    StringBuilder::append,
    StringBuilder::append
);
```
