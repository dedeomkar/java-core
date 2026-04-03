# Collections Framework Fundamentals - Part 2

## Table of Contents  
- [1. The Collections Hierarchy](#1-the-collections-hierarchy)
  - [1.1 The Iterable Interface](#11-the-iterable-interface)
  - [1.2 The Collection Interface](#12-the-collection-interface)
- [2. List: Ordered Sequences](#2-list-ordered-sequences)
  - [2.1 Key List Characteristics](#21-key-list-characteristics)
  - [2.2 List Specific Methods](#22-list-specific-methods)
- [3. Set: Uniqueness and Organization](#3-set-uniqueness-and-organization)
  - [3.1 Rejection of Duplicates](#31-rejection-of-duplicates)
  - [3.2 Implementation Requirements](#32-implementation-requirements)
- [4. Core Collection Operations](#4-core-collection-operations)
  - [4.1 Bulk Modifications](#41-bulk-modifications)
  - [4.2 Stream and Array Conversion](#42-stream-and-array-conversion)
- [5. Unmodifiable Factory Methods](#5-unmodifiable-factory-methods)
  - [5.1 List.of and Set.of](#51-listof-and-setof)
  - [5.2 copyOf Mechanics](#52-copyof-mechanics)
- [6. Critical Pitfalls and Behavioral Edge Cases](#6-critical-pitfalls-and-behavioral-edge-cases)
  - [6.1 remove(int) vs remove(Object)](#61-removeint-vs-removeobject)
  - [6.2 Equality Across Implementations](#62-equality-across-implementations)
  - [6.3 HashSet without Equals/HashCode](#63-hashset-without-equalshashcode)

---

## 1. The Collections Hierarchy

### 1.1 The Iterable Interface

The `java.lang.Iterable` interface is the root of the collection hierarchy. It represents a "bag" of items that can be traversed.

- **Primary Goal**: To provide an `Iterator` for the enhanced `for-each` loop.
- **Key Methods**: `forEach()`, `iterator()`, and `spliterator()`.
- **Constraint**: It has no defined size and no rules regarding duplicates, order, or null values.

> [!NOTE]
> Think of `Iterable` as a **Pez dispenser**. You don't necessarily know how many candies are inside or if they are the same flavor; you just know you can keep clicking the head until no more come out.

### 1.2 The Collection Interface

Extending `Iterable`, the `Collection` interface adds fundamental management capabilities.

- **Size Awareness**: Provides `size()` and `isEmpty()`.
- **CRUD Operations**: Adds `add()`, `remove()`, and `clear()`.
- **Querying**: Adds `contains()` and `containsAll()`.

[Back to Top](#table-of-contents)

---

## 2. List: Ordered Sequences

### 2.1 Key List Characteristics

The `List` interface maintains a strictly **user-specified order**. It allows duplicates and offers high-precision control over element placement.

- **Bidirectional Traversal**: Provides `listIterator()` to move both forward and backward.
- **Index-Based Access**: Allows getting, setting, or removing elements by their numeric position.
- **Sub-Views**: `subList(start, end)` creates a view into a portion of the original list.

> [!NOTE]
> A `List` is like a **movie theater queue**. Every person has a specific rank (index 0, 1, 2) based on when they arrived. If someone leaves or joins, the ranks of everyone behind them shift accordingly.

### 2.2 List Specific Methods

```java
List<String> fruits = new ArrayList<>(List.of("Apple", "Banana", "Cherry"));

// Positional operations
fruits.add(1, "Dragonfruit"); // Inserts at index 1, shifts others
String second = fruits.get(2); // "Banana"

// Search operations
int lastIndex = fruits.lastIndexOf("Apple");

// Functional update
fruits.replaceAll(String::toUpperCase); // ["APPLE", "DRAGONFRUIT", ...]

// Sorting (In-place)
fruits.sort(Comparator.naturalOrder());
```

[Back to Top](#table-of-contents)

---

## 3. Set: Uniqueness and Organization

### 3.1 Rejection of Duplicates

A `Set` is a collection that cannot contain duplicate elements. Its primary design goal is **fast lookup**.

- **Internal Order**: Managed by the implementation, not the user. It may change as the set grows.
- **Return Value**: The `add()` method returns `false` if the item is already present (it does not replace the existing item).

> [!IMPORTANT]
> Sets use different organizational strategies. `HashSet` uses **hashing**, while `TreeSet` uses **natural or comparator-defined ordering**.

### 3.2 Implementation Requirements

For a `Set` to function correctly, the elements must be well-behaved:

1. **Hashing (HashSet/HashMap)**: Elements MUST implement meaningful `equals()` and `hashCode()` methods.
2. **Ordering (TreeSet)**: Elements MUST be `Comparable` or a `Comparator` must be provided.
3. **Immutability**: Mutating an object *after* it has been added to a set can corrupt the data structure, making the item unfindable.

```java
// Danger: Mutating keys in a set
Set<StringBuilder> sbSet = new HashSet<>();
StringBuilder sb = new StringBuilder("Data");
sbSet.add(sb);
sb.append(" Modified"); // The hashcode changed! 
System.out.println(sbSet.contains(sb)); // Likely FALSE
```

[Back to Top](#table-of-contents)

---

## 4. Core Collection Operations

### 4.1 Bulk Modifications

- **addAll(Collection)**: Adds all elements from the source.
- **removeAll(Collection)**: Removes all shared elements.
- **retainAll(Collection)**: Intersection operation – keeps only elements present in both.
- **removeIf(Predicate)**: Efficiently removes elements that meet a logical condition.

### 4.2 Stream and Array Conversion

```java
Collection<Integer> nums = Set.of(1, 2, 3, 4, 5);

// Stream API integration
long count = nums.parallelStream().filter(n -> n > 2).count();

// Conversion to Array
Integer[] array = nums.toArray(new Integer[0]);
```

[Back to Top](#table-of-contents)

---

## 5. Unmodifiable Factory Methods

### 5.1 List.of and Set.of

Since Java 9, factory methods allow for concise, **unmodifiable** collection creation.

- **Immutability**: Attempting to `add()` or `remove()` from these results in `UnsupportedOperationException`.
- **Null Safety**: These methods **disallow nulls** at the time of creation.

```java
List<String> fixed = List.of("A", "B", "C");
// fixed.add("D"); // CRASH: UnsupportedOperationException
```

### 5.2 copyOf Mechanics

`List.copyOf(Collection)` and `Set.copyOf(Collection)` guarantee an unmodifiable copy.

- **Optimization**: If the passed collection is *already* unmodifiable, `copyOf` may return the same reference instead of creating a new object.
- **Snapshot**: If the original is modifiable, it creates a disconnected snapshot of the current state.

[Back to Top](#table-of-contents)

---

## 6. Critical Pitfalls and Behavioral Edge Cases

### 6.1 remove(int) vs remove(Object)

In a `List<Integer>`, there is an ambiguity between the index-based remove and the object-based remove.

```java
List<Integer> list = new ArrayList<>(List.of(10, 20, 30));

// Ambiguity!
list.remove(1); // Removes element at INDEX 1 (the value 20)
list.remove(Integer.valueOf(10)); // Removes the OBJECT 10 (the value 10)
```

> [!WARNING]
> Always use `Integer.valueOf()` or a cast if you intend to remove a numeric object rather than a specific index.

### 6.2 Equality Across Implementations

- **Lists**: Two lists are equal if they contain the same elements in the **same order**, even if they are different classes (e.g., `ArrayList` vs `LinkedList`).
- **Sets**: Two sets are equal if they contain the **same set of elements**, regardless of order or implementation type (`HashSet` vs `TreeSet`).

### 6.3 HashSet without Equals/HashCode

An object that does not override `equals()` or `hashCode()` uses the default `Object` implementations (identity-based).

```java
class Student { String name; Student(String n){this.name=n;} }

Set<Student> students = new HashSet<>();
students.add(new Student("Fred"));
students.add(new Student("Fred")); 

System.out.println(students.size()); // 2! 
// Without overrides, these are two distinct objects in memory.
```

[Back to Top](#table-of-contents)
