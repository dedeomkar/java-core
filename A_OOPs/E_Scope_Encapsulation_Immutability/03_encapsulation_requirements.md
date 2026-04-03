# 03. Encapsulation Requirements in Java

## Table of Contents
- [1. Principles of Encapsulation](#1-principles-of-encapsulation)
- [2. Validation Strategies](#2-validation-strategies)
- [3. Defensive Copying](#3-defensive-copying)
- [4. Protecting Against Subclasses](#4-protecting-against-subclasses)

---

## 1. Principles of Encapsulation

- **Goal**: Prevent external code from putting an object into an **invalid state**.
- **Analogy**: A "Vending Machine"—you can only interact through buttons (methods). You can't reach inside and grab the snacks directly (private fields).

### The Six Golden Rules
1. **Valid Initialization**: Validate in constructors.
2. **Validated Mutation**: Validate in setters.
3. **Private Fields**: All mutable data must be private.
4. **Defensive Input**: Use a copy of caller-provided mutable objects.
5. **Defensive Output**: Return a copy/unmodifiable view of internal mutable objects.
6. **Control Subclassing**: Prevent unauthorized overrides using `final`.

---

## 2. Validation Strategies

### 2.1 Constructor Guarding
Objects should never be allowed to "wake up" in a broken state.
- **Action**: Throw exceptions for invalid inputs during construction.

### 2.2 Mutator Guarding
Setters must check the **new value** against the **current state** before applying it.

```java
public void setDay(int day) {
    if (isValidDate(day, this.month, this.year)) {
        this.day = day;
    } else {
        throw new IllegalArgumentException("Invalid state change rejected.");
    }
}
```

---

## 3. Defensive Copying

### 3.1 Handling Mutable Inputs
If a caller passes an array or a list, **clone** it before storing it.
```java
public DateRecord(int[] dateValues) {
    this.values = dateValues.clone(); // Caller cannot modify our internal array later
}
```

### 3.2 Protecting Return Values
Never return a direct reference to internal mutable structures.
```java
public int[] getValues() {
    return this.values.clone(); // Caller gets a mirror, not the original
}
```

---

## 4. Protecting Against Subclasses

- **The Problem**: A subclass can override your validated methods and "short-circuit" your logic.
- **The Fix**: Mark the class as **`final`** or make critical methods final.


---
