# Comparison Methods and Interfaces

## Table of Contents
- [1. Comparison Logic and Conventions](#1-comparison-logic-and-conventions)
  - [1.1 The Subtraction Analogy](#11-the-subtraction-analogy)
  - [1.2 Natural Order vs. Contextual Order](#12-natural-order-vs-contextual-order)
- [2. Primary Comparison Interfaces](#2-primary-comparison-interfaces)
  - [2.1 Comparable Interface](#21-comparable-interface)
  - [2.2 Comparator Interface](#22-comparator-interface)
- [3. Comparing Primitives and Strings](#3-comparing-primitives-and-strings)
  - [3.1 Primitive Wrap and Underflow Safety](#31-primitive-wrap-and-underflow-safety)
  - [3.2 String and Locale-Agnostic Comparison](#32-string-and-locale-agnostic-comparison)
- [4. Sorting Arrays and Collections](#4-sorting-arrays-and-collections)
  - [4.1 Arrays.sort Utility](#41-arrayssort-utility)
  - [4.2 Sorting Lists and Collections](#42-sorting-lists-and-collections)
- [5. Advanced Comparator Factories and Chaining](#5-advanced-comparator-factories-and-chaining)
  - [5.1 Comparator Factories](#51-comparator-factories)
  - [5.2 Chaining and Decorators](#52-chaining-and-decorators)
  - [5.3 Null Handling](#53-null-handling)

---

## 1. Comparison Logic and Conventions

### 1.1 The Subtraction Analogy

Comparison in Java follows a mathematical sign convention. The result is always an `int` that indicates the relative position of two elements.

- **Formula**: `A - B`
- **Negative result (< 0)**: `A` comes BEFORE `B`.
- **Positive result (> 0)**: `A` comes AFTER `B`.
- **Zero (0)**: `A` and `B` are in the SAME position (effectively equal for sorting purposes).

> [!NOTE]
> Think of it like a tape measure. If you subtract the position of point B from point A (A - B), a negative number means A is "behind" B (smaller index/value), while a positive number means it's "ahead" (larger index/value).

### 1.2 Natural Order vs. Contextual Order

- **Natural Order**: The "obvious" way to sort something (e.g., 1, 2, 3... or A, B, C...).
- **Contextual Order**: A specific way to sort based on the situation (e.g., sort employees by salary, then by hire date).

---

## 2. Primary Comparison Interfaces

### 2.1 Comparable Interface

The `Comparable` interface is implemented by a class to give it a **natural order**.

- **Method**: `int compareTo(T other)`
- **Constraint**: Only one natural order per class.
- **Suitability**: Best for math-like concepts (Integers, Dates, Strings).

```java
// Implementing Comparable for a custom Price class
public class Price implements Comparable<Price> {
    private final double value;
    
    public Price(double value) { this.value = value; }

    @Override
    public int compareTo(Price other) {
        // Natural order: cheapest to most expensive
        return Double.compare(this.value, other.value);
    }
}
```

### 2.2 Comparator Interface

The `Comparator` interface is implemented by an independent class or lambda to provide **flexible/contextual order**.

- **Method**: `int compare(T o1, T o2)`
- **Flexibility**: Can have multiple comparators for the same class.
- **Suitability**: Best for business domain objects (e.g., ordering items by weight, price, or distance).

> [!TIP]
> Think of `Comparable` as an object's internal compass—it always knows its own "North." Think of `Comparator` as a GPS or a Map—you can change the "destination" or rules of the journey whenever you want without changing the object itself.

```java
// Comparator as a standalone strategy
Comparator<Product> heavyFirst = (p1, p2) -> Double.compare(p2.getWeight(), p1.getWeight());
```

---

## 3. Comparing Primitives and Strings

### 3.1 Primitive Wrap and Underflow Safety

Performing direct subtraction (e.g., `o1.id - o2.id`) is dangerous due to **integer underflow/overflow**.

- **The Danger**: If `o1.id` is a large positive number and `o2.id` is a large negative number, `o1 - o2` can wrap around and produce a negative result, even though `o1 > o2`.
- **The Solution**: Use static `compare` methods in primitive wrapper classes.

```java
public int compare(Item a, Item b) {
    // WRONG: return a.id - b.id; // Potential underflow
    
    // CORRECT:
    return Integer.compare(a.id(), b.id());
}
```

> [!WARNING]
> Subtraction with `long`, `float`, or `double` followed by casting to `int` is even more prone to failure due to loss of precision and overflow. Always use `Double.compare()` or `Long.compare()`.

### 3.2 String and Locale-Agnostic Comparison

String comparisons in Java are **case-sensitive** and **locale-blind** by default.

- `s1.compareTo(s2)`: Natural (Unicode) order.
- `s1.compareToIgnoreCase(s2)`: Case-insensitive order.
- `String.CASE_INSENSITIVE_ORDER`: A reusable static `Comparator` instance.

> [!IMPORTANT]
> Standard String comparisons do not handle accented characters correctly for all languages. If internationalization (i18n) is required, use `java.text.Collator`.

---

## 4. Sorting Arrays and Collections

### 4.1 Arrays.sort Utility

The `Arrays` utility class provides overloaded `sort()` methods for all primitives and object types.

- **Object sorting**: Requires `Comparable` implementation or an explicit `Comparator`.
- **Sub-ranges**: Supports sorting only a subset of an array (e.g., `Arrays.sort(arr, 1, 5)`).

```java
String[] names = {"Zoe", "Alice", "Bob"};
Arrays.sort(names); // Natural order
Arrays.sort(names, String.CASE_INSENSITIVE_ORDER); // Contextual order
```

### 4.2 Sorting Lists and Collections

There are three main ways to sort a `List`:

1.  `list.sort(comparator)`: Default method in the `List` interface.
2.  `Collections.sort(list)`: Legacy method (requires elements to be `Comparable`). That is compareTo() method must be implemented in elements. 
3.  `Collections.sort(list, comparator)`: Legacy method with explicit order.

```java
List<String> items = new ArrayList<>(List.of("C", "a", "B"));

// 1. Natural Order (Unicode): "B", "C", "a"
// Requires elements to implement Comparable (compareTo)
Collections.sort(items); 

// 2. Custom Order (Case-Insensitive): "a", "B", "C"
items.sort(String.CASE_INSENSITIVE_ORDER);

// 3. Custom Order (Reverse Natural): "a", "C", "B"
// Using a explicit Comparator
items.sort(Comparator.reverseOrder());
```


> [!TIP]
> Prefer `list.sort(comparator)` in modern (Java 8+) codebases. Pass `null` if you want to use the natural order of elements.

---

## 5. Advanced Comparator Factories and Chaining

### 5.1 Comparator Factories

The `Comparator` interface provides static factories to build comparators using **Method References** or **Lambdas**.

- **`Comparator.comparing(extractor)`**: 
  - **Parameter**: `Function<T, U>` (Key Extractor).
    - **T**: The type of objects being compared (e.g., `Student`).
    - **U**: The type of the sort key being extracted (e.g., `String`).
  - **Type Constraint**: The extracted key (`U`) must implement `Comparable` (Natural Order).
  - **Logic**: It extracts the key and compares items based on that key's natural order.
  ```java
  // T is Student, U is String (getName returns String)
  Comparator<Student> byName = Comparator.comparing(Student::getName);
  ```
- **`Comparator.comparing(extractor, comparator)`**:
  - **Parameters**: `Function<T, U>` (Extractor) and `Comparator<U>` (Key Comparator). Both are **mandatory**.
  - **Logic**: Extracts a key and uses the provided `Comparator` to order those keys.
- **`Comparator.comparingInt(extractor)`**: Optimized for primitives to avoid auto-boxing. **Mandatory** `ToIntFunction<T>`.

#### Deep Dive: The Method Reference (`::`)
The `::` operator is a shorthand syntax for a lambda expression that simply calls a method.

- **Syntax**: `ClassName::methodName`
- **Equivalent Lambda**: `Student::getPercentGrade` ↔ `(Student s) -> s.getPercentGrade()`

> [!TIP]
> Use Method References whenever your lambda is just a "pass-through" to a specific method. It makes the code cleaner and more declarative.

```java
// Sort students by their percentage grade (ascending)
// 'Student::getPercentGrade' is the key extractor
Comparator<Student> byGrade = Comparator.comparingInt(Student::getPercentGrade);
```

### 5.2 Chaining and Decorators

Comparators can be chained to handle "tie-breaker" scenarios using `thenComparing`.

- **`thenComparing(extractor)`**: Takes another key extractor to use if the previous comparator returns 0 (a tie).
- **`thenComparing(comparator)`**: Takes a full `Comparator` object for secondary sorting.

```java
// 1. Primary Sort: By Name (Natural)
// 2. Secondary Sort: By Grade (Descending)
Comparator<Student> complexOrder = Comparator.comparing(Student::getName)
    .thenComparing(
        Comparator.comparingInt(Student::getPercentGrade).reversed() 
    );
```

### 5.3 Null Handling

To prevent `NullPointerException` during sorting, use `nullsFirst` or `nullsLast` decorators.

```java
// Puts objects with null names at the very beginning of the list
Comparator<Student> safeOrder = Comparator.comparing(
    Student::getName, 
    Comparator.nullsFirst(String.CASE_INSENSITIVE_ORDER)
);
```

> [!NOTE]
> Functional delegation allows us to create complex "wrappers" (like `reversed()` or `nullsFirst()`) that transform the behavior of an existing comparator without modifying its internal logic.
