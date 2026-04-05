# 04. Immutability Requirements in Java

## Table of Contents
- [1. Principles of Immutability](#1-principles-of-immutability)
- [2. State Management Strategies](#2-state-management-strategies)
- [3. Code Examples: The "With" Pattern](#3-code-examples-the-with-pattern)
- [4. Handling Collections and Deep Copies](#4-handling-collections-and-deep-copies)

---

## 1. Principles of Immutability

- **Goal**: Create objects that **never change** after construction.
- **Workflow**: Instead of changing a field, you return a **new object** with the updated values.
- **Memory Efficiency**: New objects can share references to the original object's unmodified immutable fields (like Strings).

### Core Rules for Immutability
1. **No Setters**: Remove all mutator methods.
2. **Defensive Input**: Clone/Copy mutable data provided by the caller in the constructor.
3. **Defensive Output**: Return only immutable views or copies of internal mutable state.
4. **Final Class**: Mark the class `final` to prevent subclasses from adding mutable state.

---

## 2. State Management Strategies

### 2.1 Caller-Provided Data
- **Primitive/Immutable**: Safe to store directly (e.g., `int`, `String`).
- **Mutable**: (e.g., `List`, `Date`, `Arrays`) Must be copied immediately.

### 2.2 Shared Implementation
When "updating" an immutable object, reuse existing immutable fields.
- **Analogy**: A "New Edition" of a book. The story remains the same, but you might update the cover or the preface. You don't rewrite the whole thing from scratch; you preserve the original content.

---

## 3. Code Examples: The "With" Pattern

Instead of `setGrade(int g)`, use `withGrade(int g)` which returns a new instance.

```java
public final class Student {
    private final String name;
    private final int grade;

    public Student(String name, int grade) {
        this.name = name; // String is already immutable
        this.grade = grade;
    }

    public Student withGrade(int newGrade) {
        // Reuse 'this.name' reference since it's immutable
        return new Student(this.name, newGrade);
    }
}
```

---

## 4. Handling Collections and Deep Copies

### 4.1 Unmodifiable Collections
Use `List.copyOf()` or `Collections.unmodifiableList()` to protect internal lists.
- **Efficiency Note**: `List.copyOf()` is smart—if the input list is already unmodifiable, it returns the same reference instead of duplicating it.

### 4.2 Deep vs Shallow Copy
- **Shallow Copy**: Copying the list structure but keeping references to the same objects.
- **Deep Copy**: Required if the objects *inside* the list are mutable (e.g., a `List<StringBuilder>`). 
- **Recommendation**: Always prefer lists of immutable objects (e.g., `List<String>`) to avoid the overhead of deep copying.


---
