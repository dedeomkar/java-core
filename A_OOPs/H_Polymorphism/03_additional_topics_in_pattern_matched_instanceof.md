# Additional Topics in Pattern-Matched instanceof

## Table of Contents
- [1. Scope of Pattern Variables](#1-scope-of-pattern-variables)
  - [1.1 Local Scope Rules](#11-local-scope-rules)
  - [1.2 Logical Inversion](#12-logical-inversion)
- [2. Complex Boolean Expressions](#2-complex-boolean-expressions)
  - [2.1 Short-Circuit AND (&&)](#21-short-circuit-and-&&)
  - [2.2 Short-Circuit OR (||) and Negation (!)](#22-short-circuit-or-||-and-negation-!)
- [3. Flow-Based Scoping (Smart Scoping)](#3-flow-based-scoping-smart-scoping)
  - [3.1 Loop Scoping](#31-loop-scoping)
  - [3.2 Exception-Based Scoping](#32-exception-based-scoping)
- [4. Generic Type Pattern Matching](#4-generic-type-pattern-matching)
  - [4.1 Compile-Time Generic Validation](#41-compile-time-generic-validation)

---

## 1. Scope of Pattern Variables

### 1.1 Local Scope Rules

- **Conditional Initialization**: Pattern variables are only initialized if the `instanceof` test evaluates to `true`.
- **Definite Assignment**: The compiler tracks exactly where the variable is "definitely assigned" and limits its scope accordingly.
- **Block-Level Scope**: In a simple `if` statement, the variable is in scope only within the `true` block.

> [!NOTE]
> Imagine a security badge that only works if you pass the fingerprint scanner. The badge (pattern variable) is only activated and usable once you're inside the high-security area (the `true` block).

```java
Object obj = "Hello World";

if (obj instanceof String s) {
    // 's' is in scope here
    System.out.println(s.length());
} else {
    // 's' is NOT in scope here
    // System.out.println(s.length()); // Compilation Error
}
```

### 1.2 Logical Inversion

- If the `instanceof` test is negated using `!`, the scope of the pattern variable shifts to the `else` block (or the "fall-through" area).
- This allows for "guard clauses" that simplify code by handling the negative case first.

```java
if (!(obj instanceof String s)) {
    // 's' is NOT in scope here
    return;
}
// 's' IS in scope here because the method didn't return
System.out.println(s.toLowerCase());
```


---

## 2. Complex Boolean Expressions

### 2.1 Short-Circuit AND (&&)

- Pattern variables can be used immediately within the same `if` condition using the `&&` operator.
- **Safety**: Since `&&` short-circuits, the right-hand side is only evaluated if the `instanceof` check on the left succeeded.

```java
if (obj instanceof String s && s.length() > 5) {
    System.out.println("Long string: " + s);
}
```

### 2.2 Short-Circuit OR (||) and Negation (!)

- **OR Limitation**: Declaring a pattern variable in a `||` expression is generally prohibited if it's used on the right side.
- **Reason**: If the left side is `false`, the right side evaluates, but the variable from the left side was never initialized.
- **Negated OR**: However, the compiler allows patterns in complex expressions if it can guarantee initialization through logical flow.

> [!WARNING]
> Using `||` with pattern variables often leads to "not definitely assigned" errors because the compiler cannot guarantee the variable will be ready for use in all branches of the expression.


---

## 3. Flow-Based Scoping (Smart Scoping)

### 3.1 Loop Scoping

- **While Loops**: The pattern variable is in scope within the loop body as long as the condition holds true.
- **For Loops**: Traditional C-style `for` loops can use pattern matching in their conditional expression.

```java
// 's' is available inside the loop body
while (obj instanceof String s && s.length() > 0) {
    System.out.println(s);
    obj = getNextObject(); // If obj changes, s is re-evaluated
}
```

### 3.2 Exception-Based Scoping

- If a block of code throws an exception when the `instanceof` check fails, the pattern variable remains in scope for the remainder of the block.

```java
public void process(Object obj) {
    if (!(obj instanceof String s)) {
        throw new IllegalArgumentException("Expected a string");
    }
    // 's' is definitely initialized from here until the end of the method
    System.out.println(s.toUpperCase());
}
```


---

## 4. Generic Type Pattern Matching

### 4.1 Compile-Time Generic Validation

- Prior to Java 11, `instanceof` could only test against raw types or wildcards (e.g., `List<?>`).
- Modern Java allows including generic type information (`List<String>`) if it can be proven valid at compile-time.
- **Constraint**: The compiler must be able to verify that if the object is of that base type, it *must* also be compatible with those generics.

```java
Collection<String> strings = new ArrayList<>();

// Valid: The compiler knows 'strings' can only contain Strings
if (strings instanceof List<String> ls) {
    ls.add("New Item");
}

// INVALID: String can never be a Number
// if ("test" instanceof Number n) { ... } // Compilation Error
```

