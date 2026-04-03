# Java Comparison and Sorting

## Table of Contents
- [1. Fundamental Comparison Interfaces](#1-fundamental-comparison-interfaces)
  - [1.1 Comparable: The Natural Order](#11-comparable-the-natural-order)
  - [1.2 Comparator: External Ordering](#12-comparator-external-ordering)
- [2. Implementation Mechanics and Pitfalls](#2-implementation-mechanics-and-pitfalls)
  - [2.1 The Subtraction Analogy](#21-the-subtraction-analogy)
  - [2.2 Numeric Overflows and Underflows](#22-numeric-overflows-and-underflows)
- [3. Sorting APIs](#3-sorting-apis)
  - [3.1 Arrays.sort()](#31-arrayssort)
  - [3.2 List.sort() and Collections.sort()](#32-listsort-and-collectionssort)
  - [3.3 Mutating Collections](#33-mutating-collections)
- [4. Comparator Factories and Composition](#4-comparator-factories-and-composition)
  - [4.1 Comparison Chain Pattern](#41-comparison-chain-pattern)
  - [4.2 Handling Null Values](#42-handling-null-values)
  - [4.3 Higher-Order Decorators](#43-higher-order-decorators)

---

## 1. Fundamental Comparison Interfaces

In Java, ordering is determined by two primary interfaces. Understanding when to use each is critical for clean architecture.

### 1.1 Comparable: The Natural Order

- **Definition**: Implemented by the class itself to define its "default" or "natural" ordering.
- **Method**: `int compareTo(T o)`
- **Scope**: Best suited for mathematical or chronological concepts (e.g., `Integer`, `String`, `LocalDate`).

> [!NOTE]
> Think of `Comparable` like a person's height. It is an intrinsic property of the person that they can report themselves.

```java
public class FinancialInstrument implements Comparable<FinancialInstrument> {
    private String ticker;
    
    @Override
    public int compareTo(FinancialInstrument other) {
        return this.ticker.compareTo(other.ticker);
    }
}
```

### 1.2 Comparator: External Ordering

- **Definition**: Implemented by a separate class or lambda to provide an "ad-hoc" or specific ordering logic.
- **Method**: `int compare(T o1, T o2)`
- **Scope**: Ideal for business domain objects where multiple ordering strategies are required (e.g., by Price, by Volatility, by Expiry).

> [!NOTE]
> Think of `Comparator` like a tailor measuring people. The tailor is external to the person and can decide to sort them by arm length, waist size, or shoulder width depending on the task.

```java
// Comparator defined as a separate strategy
Comparator<Order> priceComparator = (o1, o2) -> 
    Double.compare(o1.getPrice(), o2.getPrice());
```

[Back to Top](#java-comparison-and-sorting)

---

## 2. Implementation Mechanics and Pitfalls

### 2.1 The Subtraction Analogy

The result of a comparison is effectively a subtraction: `A - B`.
- **Negative result**: `A` comes before `B` (A is smaller).
- **Zero**: `A` and `B` are equivalent in order.
- **Positive result**: `A` comes after `B` (A is larger).

### 2.2 Numeric Overflows and Underflows

It is tempting to implement `compare` using direct subtraction: `(o1, o2) -> o1.val - o2.val`. **Never do this in production.**

> [!WARNING]
> Integer subtraction can overflow or underflow. For instance, `Integer.MIN_VALUE - 1` results in `Integer.MAX_VALUE`, flipping the comparison logic entirely. Similarly, floating-point subtraction can lose precision or return non-zero fractions that get truncated to zero when cast to `int`.

**The Right Way**: Use the static helper methods in primitive wrapper classes.

```java
public int compare(Trade t1, Trade t2) {
    // AVOID: return (int)(t1.getQuantity() - t2.getQuantity());
    
    // PREFER: Accurate and safe
    return Long.compare(t1.getQuantity(), t2.getQuantity());
}
```

[Back to Top](#java-comparison-and-sorting)

---

## 3. Sorting APIs

Java provides robust utilities for sorting arrays and collections, leveraging the interfaces discussed above.

### 3.1 Arrays.sort()

Found in `java.util.Arrays`, this utility provides overloads for:
- All primitive types (uses Dual-Pivot Quicksort).
- Object arrays (uses Timsort) using natural order.
- Object arrays using a custom `Comparator`.
- Sub-ranges (start to end index).

### 3.2 List.sort() and Collections.sort()

- **`list.sort(Comparator c)`**: The modern (Java 8+) default method on the `List` interface.
- **`Collections.sort(List l)`**: Traditional method for natural ordering.
- **`Collections.sort(List l, Comparator c)`**: Traditional method delegating to `list.sort()`.

> [!IMPORTANT]
> `Collections.sort(list)` will throw a `ClassCastException` at runtime if the elements in the list do not implement `Comparable`.

### 3.3 Mutating Collections

Beyond sorting, the `Collections` utility provides several low-latency friendly mutations:
- **`reverse(List<?> list)`**: Reverses the order of elements.
- **`rotate(List<?> list, int distance)`**: Shifts elements by a distance, with wrap-around.
- **`shuffle(List<?> list)`**: Randomizes order (useful for Monte Carlo simulations).
- **`swap(List<?> list, int i, int j)`**: Exchanges elements at specific indices.

[Back to Top](#java-comparison-and-sorting)

---

## 4. Comparator Factories and Composition

Modern Java allows for declarative comparator building using static factory methods.

### 4.1 Comparison Chain Pattern

Instead of nested `if-else` blocks, use `comparing()` and `thenComparing()`.

```java
Comparator<Student> multiSort = Comparator
    .comparing(Student::getName) // Primary sort
    .thenComparing(Student::getGrade, Comparator.reverseOrder()); // Secondary sort (descending)
```

### 4.2 Handling Null Values

Passing a `null` to a standard comparator usually triggers a `NullPointerException`. Java provides "Null-safe" decorators.

> [!TIP]
> Use `nullsFirst` or `nullsLast` to explicitly define how `null` values should be positioned in the sorted list.

```java
Comparator<String> safeComparator = Comparator.nullsFirst(String.CASE_INSENSITIVE_ORDER);
```

### 4.3 Higher-Order Decorators

Comparators can be transformed using higher-order functions:
- **`reversed()`**: Flips the current comparator's logic.
- **`comparingInt()` / `comparingDouble()`**: Specialized factories that avoid boxing/unboxing overhead.

```java
// Efficient sorting for low-latency systems
List<MarketUpdate> updates = ...;
updates.sort(Comparator.comparingLong(MarketUpdate::getTimestamp));
```

[Back to Top](#java-comparison-and-sorting)
