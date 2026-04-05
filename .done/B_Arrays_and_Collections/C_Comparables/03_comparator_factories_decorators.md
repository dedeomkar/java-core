# Comparator Factories and Decorators

## Table of Contents
- [1. The Delegation Model](#1-the-delegation-model)
- [2. Core Factory Methods](#2-core-factory-methods)
- [3. Field Extraction with `comparing()`](#3-field-extraction-with-comparing)
- [4. Chaining and Decorators](#4-chaining-and-decorators)
- [5. Null-Safe Comparisons](#5-null-safe-comparisons)
- [6. Advanced Chaining: The Student Case Study](#6-advanced-chaining-the-student-case-study)

---

## 1. The Delegation Model

A `Comparator` implementation can delegate its decision-making logic to other `Comparators`. This pattern is a form of **Function Decorator** or **Higher-Order Function**, where a method takes a comparator and returns a transformed version of it.

> [!NOTE]
> **Analogy: The Assembly Line**
> Think of a basic comparator as a simple sorter. A decorator is like adding a specialized attachment to that sorter—one that flips items (reverse), one that handles broken items (nulls), or one that passes ties to a second sorter.

```java
// Conceptual Delegation: Manual Reversed Comparator
public static <E> Comparator<E> backwardComparator(Comparator<E> forward) {
    return (a, b) -> forward.compare(b, a); // Delegates to forward, but swaps arguments
}
```

---

## 2. Core Factory Methods

Java provides built-in factory methods to create standard orderings without manual implementation.

- **`Comparator.naturalOrder()`**: Uses the `Comparable` implementation of the class (e.g., Unicode for Strings, numerical for Integers).
- **`Comparator.reverseOrder()`**: The inverse of natural order.

> [!IMPORTANT]
> These methods require the objects being compared to implement `Comparable`. If they don't, a `ClassCastException` will occur at runtime.

---

## 3. Field Extraction with `comparing()`

The `comparing` family of methods allows you to create a comparator by simply specifying a "key extractor" function. These methods typically accept a `Function<T, U>` (where `T` is the object being compared and `U` is the sort key), which the comparator then uses to extract the comparison value from each object.

| Method | Use Case |
| :--- | :--- |
| `comparing(Function)` | Standard object extraction (e.g., `Student::getName`). |
| `comparingInt(ToIntFunction)` | Optimized for primitives to avoid auto-boxing. |
| `comparingLong(ToLongFunction)` | Optimized for `long` primitives. |
| `comparingDouble(ToDoubleFunction)` | Optimized for `double` primitives. |

```java
// T = Student (The object being sorted)
// Key Extractor = Student::getPercentGrade (Returns primitive int)
Comparator<Student> byGrade = Comparator.comparingInt(Student::getPercentGrade);

// Example Usage:
// Sorts students by numerical grade (e.g., 52 comes before 83)
students.sort(byGrade);
```

---

## 4. Chaining and Decorators

Real-world sorting often requires multiple criteria (e.g., sort by Last Name, then by First Name if Last Names are equal).

- **`.thenComparing()`**: Defines a secondary (or tertiary) sort order to be used if the previous comparator returns `0` (a "tie").
- **`.reversed()`**: An instance method that returns a comparator with the opposite ordering logic.

```java
// Sort by name, then by grade in reverse (highest to lowest)
Comparator<Student> complexSort = Comparator.comparing(Student::getName)
                                            .thenComparing(Comparator.comparingInt(Student::getPercentGrade).reversed());
```

---

## 5. Null-Safe Comparisons

By default, passing a `null` to a comparator usually results in a `NullPointerException`. The `nullsFirst` and `nullsLast` decorators handle this gracefully.

> [!WARNING]
> Always wrap your base comparator in a null-safe decorator if your dataset might contain null values.

```java
// Names starting with null will appear at the top
Comparator<Student> nullSafeName = Comparator.nullsFirst(
    Comparator.comparing(Student::getName, String.CASE_INSENSITIVE_ORDER)
);
```

---

## 6. Advanced Chaining: The Student Case Study

The following example demonstrates the power of the `Comparator` API in building complex business logic fluently.

### The Business Domain Object
```java
public class Student {
    private String name;
    private int percentGrade;

    public Student(String name, int percentGrade) {
        this.name = name;
        this.percentGrade = percentGrade;
    }

    public String getName() { return name; }
    public int getPercentGrade() { return percentGrade; }

    @Override
    public String toString() {
        return String.format("%s (%d%%)", name, percentGrade);
    }
}
```

### The Multi-Level Sort Implementation
```java
List<Student> students = new ArrayList<>(List.of(
    new Student("Fred", 83),
    new Student("Jim", 52),
    new Student("Fred", 68),
    new Student("Alice", 93)
));

// 1. Sort by name (Ascending)
// 2. If names are identical, sort by grade (Descending)
students.sort(
    Comparator.comparing(Student::getName)
              .thenComparing(
                  Comparator.comparingInt(Student::getPercentGrade).reversed()
              )
);

// Output:
// Alice (93%)
// Fred (83%)  <-- Higher grade first for identical names
// Fred (68%)
// Jim (52%)
```
