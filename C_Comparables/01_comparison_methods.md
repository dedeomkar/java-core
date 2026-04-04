# Comparison Methods and Interfaces

## Table of Contents
- [1. The Subtraction Analogy](#1-the-subtraction-analogy)
- [2. Comparable Interface: Natural Ordering](#2-comparable-interface-natural-ordering)
- [3. Comparator Interface: External Ordering](#3-comparator-interface-external-ordering)
- [4. Implementation Best Practices](#4-implementation-best-practices)
  - [4.1 Avoiding Arithmetic Subtraction](#41-avoiding-arithmetic-subtraction)
  - [4.2 The Wrapper Advantage](#42-the-wrapper-advantage)
- [5. String and Specialized Comparisons](#5-string-and-specialized-comparisons)
  - [5.1 Case Insensitivity](#51-case-insensitivity)
  - [5.2 Locales and Collators](#52-locales-and-collators)

---

## 1. The Subtraction Analogy

In Java, comparison results follow a mathematical convention where the sign of the returned integer dictates the relationship between two objects.

- **Formula**: `arg1 - arg2 = result`
- **Negative result**: `arg1` comes **before** `arg2` (smaller).
- **Positive result**: `arg1` comes **after** `arg2` (larger).
- **Zero**: `arg1` and `arg2` are equivalent in the sort order.

> [!NOTE]
> Think of a number line. If you subtract $7$ from $3$, you move $4$ units to the left $(-4)$. If you subtract $2$ from $8$, you move $6$ units to the right $(+6)$. The direction of movement on the line is exactly what Java uses to determine order.

---

## 2. Comparable Interface: Natural Ordering

The `Comparable<T>` interface provides a "natural order" for a class.

- **Definition**: Implemented internally by the class itself.
- **Scope**: Exactly one natural order per class.
- **Method**: `public int compareTo(T other)`

> [!TIP]
> Use `Comparable` for "math-like" concepts where the ordering is intrinsic and obvious, such as dates, weights, or numeric values.

```java
public class Temperature implements Comparable<Temperature> {
    private double celsius;

    @Override
    public int compareTo(Temperature other) {
        // Natural order: colder comes before warmer
        return Double.compare(this.celsius, other.celsius);
    }
}
```

---

## 3. Comparator Interface: External Ordering

The `Comparator<T>` interface defines an ordering external to the class being compared.

- **Definition**: Implemented by an independent class or lambda.
- **Scope**: Allows multiple different orderings for the same type.
- **Method**: `public int compare(T o1, T o2)`

> [!IMPORTANT]
> `Comparator` is preferred for business domain objects where ordering depends on the context (e.g., sorting employees by salary vs. by start date).

```java
// Ordering People by height (external strategy)
Comparator<Person> heightComparator = (p1, p2) -> 
    Integer.compare(p1.getHeightCm(), p2.getHeightCm());

// Ordering People by distance from office
Comparator<Person> distanceComparator = (p1, p2) ->
    Double.compare(p1.getDistance(), p2.getDistance());
```

---

## 4. Implementation Best Practices

### 4.1 Avoiding Arithmetic Subtraction

It is tempting to implement `compare` using simple subtraction: `return o1.id - o2.id`. This is dangerous.

- **Underflow/Overflow**: Subtracting large integers (e.g., `Integer.MIN_VALUE` from a positive number) can result in arithmetic wrapping, flipping the sign and breaking the sort.
- **Precision Loss**: Casting `double` or `float` subtraction results to `int` can truncate fractional values, making non-equal numbers appear equal (yielding `0`).

### 4.2 The Wrapper Advantage

Always use the static `compare` methods provided by primitive wrapper classes.

```java
// INCORRECT: Risk of overflow/underflow
public int compare(long a, long b) { return (int)(a - b); } 

// CORRECT: Safe and idiomatic
public int compare(long a, long b) { return Long.compare(a, b); }
```

---

## 5. String and Specialized Comparisons

### 5.1 Case Insensitivity

While `String` implements `Comparable` (Unicode order), it provides specialized tools for text matching:

- `compareToIgnoreCase(String str)`: A method for direct comparison.
- `String.CASE_INSENSITIVE_ORDER`: A pre-built `Comparator` object.

### 5.2 Locales and Collators

Standard string comparison is unaware of local linguistic rules (e.g., how accented characters are treated in German vs. French).

- **Collator**: Use `java.text.Collator` for locale-sensitive sorting.
- **CollationKeys**: Faster for repeated comparisons of the same strings in large datasets.

> [!NOTE]
> For standard certification exams, `Collator` depth is usually out of scope, but it is critical for production-grade internationalized applications.
