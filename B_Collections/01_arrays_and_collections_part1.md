# Arrays and Collection Utilities - Part 1

## Table of Contents
- [1. Array Fundamentals](#1-array-fundamentals)
  - [1.1 Covariant Behavior](#11-covariant-behavior)
  - [1.2 Initialization and Creation](#12-initialization-and-creation)
  - [1.3 Array Literals](#13-array-literals)
- [2. Resizing and Copying Arrays](#2-resizing-and-copying-arrays)
  - [2.1 Arrays.copyOf](#21-arrayscopyof)
  - [2.2 System.arraycopy](#22-systemarraycopy)
- [3. The java.util.Arrays Utility Class](#3-the-javautilarrays-utility-class)
  - [3.1 core Utility Methods](#31-core-utility-methods)
  - [3.2 Lexicographic Comparison](#32-lexicographic-comparison)
  - [3.3 Equality and Hashing](#33-equality-and-hashing)
- [4. Parallel Array Processing](#4-parallel-array-processing)
  - [4.1 Concurrent Operations](#41-concurrent-operations)
  - [4.2 Parallel Prefix Sum](#42-parallel-prefix-sum)
- [5. Searching Mechanics](#5-searching-mechanics)
  - [5.1 Binary Search Logic](#51-binary-search-logic)
  - [5.2 Insertion Point Calculation](#52-insertion-point-calculation)
- [6. Constraints and Edge Cases](#6-constraints-and-edge-cases)
  - [6.1 Generic Array Creation](#61-generic-array-creation)
  - [6.2 Var and Array Literals](#62-var-and-array-literals)

---

## 1. Array Fundamentals

### 1.1 Covariant Behavior

- Java arrays are **covariant**, meaning if `String` is a subtype of `Object`, then `String[]` is a subtype of `Object[]`.
- This allows assigning a specific child array to a generic parent array reference.
- **Runtime Risk**: Covariance can lead to `ArrayStoreException` at runtime if you attempt to store an incompatible type into the array through the generic reference.

> [!NOTE]
> Think of a "Fruit Basket" (Object array) that is actually a "Banana Crate" (String array). You can treat it as a fruit basket to look at the bananas, but if you try to force an Apple (Integer) into it, the crate will "break" (ArrayStoreException).

```java
String[] strings = {"Alpha", "Beta"};
Object[] objects = strings; // Valid due to covariance

try {
    objects[0] = 10; // Throws ArrayStoreException: Integer is not a String
} catch (ArrayStoreException e) {
    System.out.println("Invalid storage attempt!");
}
```

### 1.2 Initialization and Creation

- Arrays are objects and are initialized with **default zero-like values** (0, false, or null).
- **Fixed Size**: Once constructed, the size cannot be changed.
- **Reporting Size**: Use the read-only field `length` (not a method).

```java
// Single-dimensional
String[] s1 = new String[4]; // [null, null, null, null]

// Multi-dimensional (Array of Arrays)
String[][] s2 = new String[4][6]; // 4 elements, each an array of 6 strings

// Partial Creation
String[][] s3 = new String[4][]; // 4 elements, each initialized to null
```

> [!WARNING]
> While `String s[]` is currently legal (deprecated syntax), always prefer `String[] s` as it keeps the type definition together.

### 1.3 Array Literals

- Literals provide a shorthand for simultaneous declaration and initialization.
- **Inferred vs Explicit**: You can let the compiler infer the type or state it explicitly.

```java
// Inferred base type
String[] names = {"Alice", "Bob"};

// Explicit base type (Square brackets MUST be empty)
String[] roles = new String[]{"Admin", "User"};

// Multi-dimensional literal
int[][] matrix = { {1, 2}, {3, 4} };
```


---

## 2. Resizing and Copying Arrays

Since array sizes are immutable, "resizing" requires creating a new array and copying the elements.

### 2.1 Arrays.copyOf

- A simple utility that creates a new array of a specific length and copies elements from the start.
- If the new length is greater, the extra slots are zero-filled.

```java
int[] original = {1, 2, 3};
int[] resized = Arrays.copyOf(original, 5); // {1, 2, 3, 0, 0}
```

### 2.2 System.arraycopy

- A low-level, native method providing granular control.
- Allows specifying source/destination offsets and the number of elements to copy.
- **Overlapping Safety**: It correctly handles copies within the same array (shuffling elements up or down) by automatically determining the direction to avoid overwriting data being copied.

```java
int[] ia = {1, 2, 3, 4, 5};
int[] target = new int[10];

// Params: src, srcPos, dest, destPos, length
System.arraycopy(ia, 0, target, 2, 3); // target: {0, 0, 1, 2, 3, 0, ...}
```


---

## 3. The java.util.Arrays Utility Class

### 3.1 Core Utility Methods

The `Arrays` class provides several helper methods to manage array data efficiently.

- **asList**: Creates a **list-view** on the array. Changes to the array are reflected in the list and vice versa. You cannot add/remove elements (fixed size).
- **fill**: Writes a specific value to all or a sub-range of the array.
- **mismatch**: Returns the index of the first difference between two arrays (or -1 if identical).
- **sort**: Orders the array using natural order or a provided `Comparator`.

```java
String[] arr = {"Z", "A", "B"};
Arrays.sort(arr); // ["A", "B", "Z"]

List<String> listView = Arrays.asList(arr);
// listView.add("C"); // UnsupportedOperationException
```

### 3.2 Lexicographic Comparison

- **compare**: Performs lexicographic comparison (like words in a dictionary).
- For numbers, this can be counter-intuitive: `2001` comes before `501` because '2' comes before '5'.

> [!NOTE]
> Think of phone book sorting. "Aardvark" (even if long) comes before "Zoo" (short) because 'A' precedes 'Z'.

### 3.3 Equality and Hashing

- **equals**: Compares elements at equivalent indexes.
- **deepEquals**: Recursively tests contained arrays (essential for multidimensional arrays).
- **hashCode/deepHashCode**: Generates a hash code consistent with the equality logic.


---

## 4. Parallel Array Processing

Java provides utility implementations to leverage multi-core CPUs for array operations.

### 4.1 Concurrent Operations

- **parallelSort**: Uses multiple threads to sort large arrays faster.
- **parallelSetAll**: Sets every element based on a function that takes the index as input.

```java
int[] data = new int[1000];
Arrays.parallelSetAll(data, i -> i * 2); // Fills with 0, 2, 4, 6...
```

### 4.2 Parallel Prefix Sum

- **parallelPrefix**: Calculates an element based on the operation performed on all previous elements.
- **Requirement**: The operation must be **associative** and **side-effect free**.

> [!IMPORTANT]
> A running total is a classic example of a prefix operation. `[1, 2, 3]` becomes `[1, 1+2, 1+2+3] -> [1, 3, 6]`.

```java
int[] nums = {1, 2, 3, 4, 5};
Arrays.parallelPrefix(nums, (a, b) -> a + b); // {1, 3, 6, 10, 15}
```


---

## 5. Searching Mechanics

### 5.1 Binary Search Logic

- **Prerequisite**: The array MUST be sorted.
- The algorithm starts at the middle and halves the search range iteratively.

### 5.2 Insertion Point Calculation

- If an item is not found, `binarySearch` returns a negative number indicating where it **should** be.
- **Formula**: `-(insertion point) - 1`.
- The `-1` offset is necessary because `-0` is indistinguishable from `0`.

```java
String[] names = {"Alice", "Bob", "Dan"};
int index = Arrays.binarySearch(names, "Charlie"); 
// Charlie should be at index 2. Result: -2 - 1 = -3
```


---

## 6. Constraints and Edge Cases

### 6.1 Generic Array Creation

- You cannot instantiate an array with a generic type parameter (`new E[10]`).
- The base type must be known at runtime. You can create an `Object[]` and cast, but this triggers warnings and potential assignment issues.

```java
// public <E> E[] createArray() { return new E[10]; } // Compilation Error
```

### 6.2 Var and Array Literals

- `var` cannot be used with simple array literals (`{1, 2}`) because the literal requires a context to infer the type.
- However, if the type is explicit, `var` works perfectly.

```java
// var items = {1, 2, 3}; // Fails: cannot infer type
var items = new int[]{1, 2, 3}; // Works
```

