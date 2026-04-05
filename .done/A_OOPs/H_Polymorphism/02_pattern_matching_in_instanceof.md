# Pattern Matching in instanceof

## Table of Contents
- [1. Evolution of instanceof](#1-evolution-of-instanceof)
  - [1.1 The Legacy Approach](#11-the-legacy-approach)
  - [1.2 Modern Pattern Matching (Java 16+)](#12-modern-pattern-matching-java-16)
- [2. Scope and Flow-Sensitive Typing](#2-scope-and-flow-sensitive-typing)
  - [2.1 Conditional Scope](#21-conditional-scope)
  - [2.2 Shadowing and Re-declaration](#22-shadowing-and-re-declaration)
- [3. Record Patterns (Java 21)](#3-record-patterns-java-21)
  - [3.1 Destructuring Records](#31-destructuring-records)
  - [3.2 Nested Record Extraction](#32-nested-record-extraction)
- [4. Logic and Constraint Rules](#4-logic-and-constraint-rules)
  - [4.1 Short-Circuiting Behavior](#41-short-circuiting-behavior)
  - [4.2 Plausibility and Null Checks](#42-plausibility-and-null-checks)

---

## 1. Evolution of instanceof

### 1.1 The Legacy Approach

- Traditionally, checking an object's type was a multi-step, ceremony-heavy process.
- **Ceremoy Steps**: 
    1. Test type using `instanceof`.
    2. Explicitly declare a new variable.
    3. Perform a manual cast.
    4. Assign the casted value.

> [!NOTE]
> It's like having a security guard who recognizes you, but still makes you show your ID and fill out a visitor form every time you enter the building.

```java
Object thing = "Hello World";

// Legacy: Multiple steps required
if (thing instanceof String) {
    String s = (String) thing; // Ceremony: Manual cast + assignment
    System.out.println(s.toLowerCase());
}
```

### 1.2 Modern Pattern Matching (Java 16+)

- Introduced in Java 16 to reduce boilerplate.
- **Single Expression**: Combines testing, declaration, and assignment into one logical step.
- **Type Pattern**: The right-hand side of `instanceof` now accepts a type followed by a variable name.

```java
Object thing = "Hello World";

// Modern: Concise conditional declaration
if (thing instanceof String s) {
    // 's' is already cast and ready to use
    System.out.println(s.toLowerCase());
}
```


---

## 2. Scope and Flow-Sensitive Typing

### 2.1 Conditional Scope

- The pattern variable exists only where it is **definitely initialized**.
- **Flow Sensitivity**: The compiler tracks the Boolean result to determine visibility.

> [!IMPORTANT]
> A pattern variable is like a temporary login session. It’s active only as long as the "authentication" (the `instanceof` check) remains valid.

```java
if (thing instanceof String s && s.length() > 5) {
    System.out.println("Long string: " + s); // s is in scope
} else {
    // System.out.println(s); // COMPILATION ERROR: s is not in scope here
}
```

### 2.2 Shadowing and Re-declaration

- **No Shadowing**: Pattern variables cannot have the same name as an existing local variable in the same scope.
- **Re-use**: Because the scope is restricted to the conditional block, the same name can be reused in subsequent, non-overlapping `instanceof` checks.

```java
if (obj instanceof String s) {
    System.out.println(s);
}
// s is no longer in scope here

if (obj instanceof Integer s) { // Valid: This is a new 's' for this block
    System.out.println(s + 10);
}
```


---

## 3. Record Patterns (Java 21)

### 3.1 Destructuring Records

- Java 21 extends pattern matching to **Records**, allowing for immediate extraction of component values.
- **Automatic Unpacking**: No need to call accessor methods (e.g., `person.name()`) after the check.

```java
record Point(int x, int y) {}

Object obj = new Point(10, 20);

if (obj instanceof Point(int x, int y)) {
    System.out.println("Coordinates: " + x + ", " + y);
}
```

### 3.2 Nested Record Extraction

- Patterns can be nested to drill down into complex data structures in a single line.
- **Var Support**: You can use `var` within the pattern to let the compiler infer component types.

```java
record Person(String name) {}
record Account(int id, int balance, Person owner) {}

Object obj = new Account(1, 5000, new Person("Inaya"));

// Destructuring nested record 'Person' inside 'Account'
if (obj instanceof Account(int id, int bal, Person(String name))) {
    System.out.println("Owner: " + name + " | Balance: " + bal);
}
```


---

## 4. Logic and Constraint Rules

### 4.1 Short-Circuiting Behavior

- Pattern variables work seamlessly with the short-circuiting `&&` operator.
- The right-hand side of `&&` only executes if the `instanceof` check is `true`, ensuring the variable is safely initialized.

> [!WARNING]
> You cannot use the pattern variable with the `||` (OR) operator in a way that risks using it uninitialized. If the first part is false, the second part might execute without the variable being set.

```java
// VALID: s is guaranteed if the second part runs
if (obj instanceof String s && s.startsWith("A")) { ... }

// INVALID: Compilation Error
// if (obj instanceof String s || s.length() > 0) { ... }
```

### 4.2 Plausibility and Null Checks

- **Type Plausibility**: The compiler rejects tests that are mathematically impossible (e.g., checking if an `Integer` is a `String`).
- **Null Safety**: `instanceof` always returns `false` for `null` references; no pattern variable is initialized in this case.

```java
Integer i = 10;
// if (i instanceof String s) { } // COMPILATION ERROR: Incompatible types

Object n = null;
if (n instanceof String s) {
    // This block will never execute
}
```

