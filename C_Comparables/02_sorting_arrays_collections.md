# Sorting Arrays and Collections

## Table of Contents
- [1. Array Sorting with `Arrays.sort()`](#1-array-sorting-with-arrayssort)
  - [1.1 Primitives vs. Objects](#11-primitives-vs-objects)
  - [1.2 Sub-range Sorting](#12-sub-range-sorting)
- [2. List and Collections Sorting](#2-list-and-collections-sorting)
  - [2.1 The `List.sort()` Default Method](#21-the-listsort-default-method)
  - [2.2 The `Collections` Utility](#22-the-collections-utility)
- [3. Structural List Utilities](#3-structural-list-utilities)
  - [3.1 Reversing and Swapping](#31-reversing-and-swapping)
  - [3.2 Rotating: The Conveyor Belt](#32-rotating-the-conveyor-belt)
  - [3.3 Shuffling: Randomizing Elements](#33-shuffling-randomizing-elements)
- [4. Verification Checklist](#4-verification-checklist)

---

## 1. Array Sorting with `Arrays.sort()`

The `java.util.Arrays` class provides a suite of overloaded `sort()` methods that handle everything from primitive values to complex object types.

### 1.1 Primitives vs. Objects

- **Primitives**: Uses a **Dual-Pivot Quicksort** which is $O(n \log n)$ and highly optimized for CPU cache efficiency.
- **Objects**: Uses **TimSort**, a hybrid of Merge Sort and Insertion Sort that is stable (preserves the relative order of equal elements).

> [!NOTE]
> Think of sorting integers like arranging a set of labeled boxes by weight. You move them quickly based on a pivot point. Sorting objects, however, is more like organizing a library; stability matters because you want books with the same title to stay in their original relative order.

```java
// Sorting primitives (Natural Order)
int[] numbers = {45, 12, 89, 7};
Arrays.sort(numbers); // {7, 12, 45, 89}

// Sorting objects with a Comparator
String[] names = {"Zane", "Alice", "Bob"};
Arrays.sort(names, String.CASE_INSENSITIVE_ORDER);
```

### 1.2 Sub-range Sorting

You can sort a specific portion of an array by specifying the `fromIndex` (inclusive) and `toIndex` (exclusive).

```java
int[] data = {10, 9, 8, 7, 6, 5, 4, 3, 2, 1};
// Sort only the elements at indices 2, 3, 4, and 5
Arrays.sort(data, 2, 6); 
// Results in: {10, 9, [5, 6, 7, 8], 4, 3, 2, 1}
```

---

## 2. List and Collections Sorting

While arrays have `Arrays.sort()`, lists have multiple paths for sorting depending on whether you want to use the list's own API or a utility class.

### 2.1 The `List.sort()` Default Method

In modern Java (Java 8+), the `List` interface provides a default method `sort(Comparator<? super E> c)`. This is the **preferred** way to sort a list.

```java
List<String> list = new ArrayList<>(List.of("Sheila", "Alice", "Bob"));
list.sort(String.CASE_INSENSITIVE_ORDER);
```

### 2.2 The `Collections` Utility

The legacy `java.util.Collections` class provides two `sort()` overloads:
- `sort(List<T> list)`: Sorts based on the **Natural Order** (requires elements to implement `Comparable`).
- `sort(List<T> list, Comparator<? super T> c)`: Sorts using the provided strategy.

> [!IMPORTANT]
> If you use the single-argument `Collections.sort()` on a list whose elements do **not** implement `Comparable`, the code will result in a **Compilation Error**.

---

## 3. Structural List Utilities

Beyond basic sorting, the `Collections` class provides tools to manipulate the structure of a list without necessarily "sorting" it by value.

### 3.1 Reversing and Swapping

- **`reverse(List<?> list)`**: Inverts the current order.
- **`swap(List<?> list, int i, int j)`**: Exchanges elements at the specified indices.

### 3.2 Rotating: The Conveyor Belt

`Collections.rotate(List<?> list, int distance)` moves elements circularly.

- **Positive distance**: Pushes elements to the right; those falling off the end reappear at the front.
- **Negative distance**: Pulls elements to the left.

> [!TIP]
> **Analogy**: Imagine a sushi conveyor belt. Rotating by $+2$ means every plate moves two seats to the right. The two plates that were at the end of the belt are carried around to the first two seats.

```java
List<Integer> belt = new ArrayList<>(List.of(1, 2, 3, 4, 5));
Collections.rotate(belt, 2); // Result: [4, 5, 1, 2, 3]
```

### 3.3 Shuffling: Randomizing Elements

`Collections.shuffle(List<?> list)` reorders elements randomly. This is useful for simulations, games, or testing.

- **Overload**: `shuffle(list, Random rnd)` allows you to provide a specific seed for deterministic randomness.

---

## 4. Verification Checklist

To ensure your list is correctly ordered alphabetically (Unicode/Case-Sensitive), you can use any of these three approaches:

| Method | Syntax |
| :--- | :--- |
| **List.sort** | `myList.sort(Comparator.naturalOrder())` |
| **Natural Sort** | `Collections.sort(myList)` |
| **Comparator Sort** | `Collections.sort(myList, Comparator.naturalOrder())` |

> [!WARNING]
> Always ensure your list is **mutable** (e.g., `ArrayList`) before calling these methods. Methods like `List.of()` or `Arrays.asList()` produce immutable or fixed-size lists that will throw an `UnsupportedOperationException`.
