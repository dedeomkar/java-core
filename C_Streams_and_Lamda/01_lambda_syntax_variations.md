# Lambda Expression Syntax Variations

## Table of Contents
- [1. Core Syntax and Functional Interfaces](#1-core-syntax-and-functional-interfaces)
  - [1.1 The Lambda Contract](#11-the-lambda-contract)
  - [1.2 Fundamental Structure](#12-fundamental-structure)
- [2. Parameter Syntax Variations](#2-parameter-syntax-variations)
  - [2.1 Type Inference and `var`](#21-type-inference-and-var)
  - [2.2 Parentheses Rules](#22-parentheses-rules)
- [3. Method Body Variations](#3-method-body-variations)
  - [3.1 Block vs. Expression Lambdas](#31-block-vs-expression-lambdas)
  - [3.2 Void Compatibility](#32-void-compatibility)
- [4. Type Consistency and Constraints](#4-type-consistency-and-constraints)
  - [4.1 The "All or Nothing" Rule](#41-the-all-or-nothing-rule)
  - [4.2 Annotations on Parameters](#42-annotations-on-parameters)
- [5. Expression Statements](#5-expression-statements)
  - [5.1 Definition and Categories](#51-definition-and-categories)
  - [5.2 Side Effects and Void Returns](#52-side-effects-and-void-returns)

---

## 1. Core Syntax and Functional Interfaces

### 1.1 The Lambda Contract

- **Functional Interface**: A lambda must implement an interface with exactly one abstract method.
- **Nameless Implementation**: Lambdas are instances of an automatically generated class.
- **No `this` Context**: Unlike anonymous inner classes, lambdas do not create their own `this` context.

> [!NOTE]
> Think of a Functional Interface as a **job description with exactly one task**. A lambda is a temporary worker hired specifically to perform that one task without needing a desk (`this` context) or a permanent name.

### 1.2 Fundamental Structure

The basic syntax consists of a formal parameter list, an arrow (`->`), and a method body.

```java
// Traditional Anonymous Inner Class
Predicate<String> longStringOld = new Predicate<String>() {
    @Override
    public boolean test(String s) {
        return s.length() > 3;
    }
};

// Standard Lambda Transition
Predicate<String> longString = (String s) -> { return s.length() > 3; };
```

---

## 2. Parameter Syntax Variations

### 2.1 Type Inference and `var`

- **Inferred Types**: If the compiler can determine the type from the context (e.g., `Predicate<String>`), the type name can be omitted.
- **Var Usage**: Since Java 11, `var` can be used to represent formal parameter types.

> [!TIP]
> Using `var` is like saying "**the candidate**" instead of "**the 18-year-old student named Alex**". If you're looking at a student enrollment list, "the candidate" is perfectly clear and less verbose.

```java
// Inferred parameter type
Predicate<String> p1 = (s) -> { return s.length() > 0; };

// Using var
Predicate<String> p2 = (var s) -> { return s.length() > 0; };
```

### 2.2 Parentheses Rules

- **Mandatory Parentheses**: Required if there are zero or multiple parameters, or if a type (including `var`) is specified.
- **Optional Parentheses**: Only allowed if there is **exactly one** parameter and **no type** is specified.

```java
// Single parameter, no type: Parentheses optional
Predicate<String> p3 = s -> { return s.length() > 0; };

// Multiple parameters: Parentheses mandatory
BiPredicate<String, Integer> bp = (s, i) -> s.length() > i;
```

---

## 3. Method Body Variations

### 3.1 Block vs. Expression Lambdas

- **Block Lambdas**: Use curly braces `{}` and require a `return` statement (if a result is expected).
- **Expression Lambdas**: A single expression follows the arrow. The result of the expression is implicitly returned.

```java
// Block Lambda
Function<Integer, Integer> squareBlock = (i) -> {
    int res = i * i;
    return res;
};

// Expression Lambda (Idiomatic)
Function<Integer, Integer> squareExpr = (i) -> i * i;
```

### 3.2 Void Compatibility

- Expression lambdas can be used for `void` return types if the expression is an **expression statement**.

---

## 4. Type Consistency and Constraints

### 4.1 The "All or Nothing" Rule

When specifying parameter types, you must be consistent across all parameters in the list.

> [!CAUTION]
> You cannot mix explicit types, `var`, and inferred types. It's an all-or-nothing commitment for that specific lambda.

```java
// INVALID: Mixing var and inferred
// (var ld, s) -> ... 

// INVALID: Mixing explicit and var
// (LocalDate ld, var s) -> ...

// VALID: All inferred
(ld, s) -> ...

// VALID: All var
(var ld, var s) -> ...

// VALID: All explicit
(LocalDate ld, String s) -> ...
```

### 4.2 Annotations on Parameters

Annotations can only be applied to a parameter if a type designator (`var` or explicit class name) is present.

```java
// VALID: Annotation with var
(@Nonnull var s) -> s.toLowerCase();

// INVALID: Annotation on inferred parameter
// (@Nonnull s) -> s.toLowerCase(); 
```

---

## 5. Expression Statements

### 5.1 Definition and Categories

Not all expressions are "expression statements". Only the following are valid when a `void` result is expected:
1. **Assignment** (e.g., `x = 10`)
2. **Increment/Decrement** (e.g., `i++`, `--j`)
3. **Method Invocation** (e.g., `list.add("item")`)
4. **Object Instantiation** (e.g., `new String()`)

### 5.2 Side Effects and Void Returns

A method that returns a value (like `Set.add()` which returns `boolean`) can be used as a `Consumer` (void return) because it is a method invocation.

> [!NOTE]
> This is a "**fire-and-forget**" mechanism. You trigger the action (the side effect) and ignore whatever receipt (return value) the system tries to give you.

```java
Set<String> names = new HashSet<>();

// Valid: names.add returns boolean, but Consumer ignores it
Consumer<String> addName = s -> names.add(s);

// Valid: instantiation is an expression statement
Consumer<String> dummy = s -> new String(s);
```
