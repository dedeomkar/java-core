# Instance and static methods, part 1

## Table of Contents
- [1. Method Declarations](#1-method-declarations)
  - [1.1 General Form](#11-general-form)
  - [1.2 Arrays in Method Signatures](#12-arrays-in-method-signatures)
- [2. Formal Parameters and Arguments](#2-formal-parameters-and-arguments)
  - [2.1 Parameter Rules](#21-parameter-rules)
  - [2.2 Positional Argument Passing](#22-positional-argument-passing)
- [3. Exception Handling (Throws Clause)](#3-exception-handling-throws-clause)
  - [3.1 Checked Exceptions](#31-checked-exceptions)
  - [3.2 Unchecked Exceptions](#32-unchecked-exceptions)
- [4. Method Body Returns](#4-method-body-returns)
- [5. Rules for Static Methods](#5-rules-for-static-methods)
  - [5.1 Absolute Lack of `this`](#51-absolute-lack-of-this)
  - [5.2 Interaction with Instance Fields](#52-interaction-with-instance-fields)
  - [5.3 Static Binding](#53-static-binding)

---

## 1. Method Declarations

### 1.1 General Form
Methods strictly observe a structured declaration architecture:
*   **Modifiers**: Access controls (`public`, `protected`, `private`) or lifecycle modifiers (`abstract`, `static`, `final`, `synchronized`, `native`, `strictfp`).
*   **Generic Declarations**: e.g., `<T>` positioned preceding the return type.
*   **Return Type**: The resulting data type output, or strictly `void`.
*   **Method Name**: The identifier label.
*   **Formal Parameter List**: Located universally inside `()`.
*   **Throws Clause**: An optional trailing declaration listing potential exceptions.

### 1.2 Arrays in Method Signatures
There exist two valid syntaxes for designating array bracket locations when generating a method returning an array:
1.  **Preferred Form**: Standardized, readable Java implementation.
```java
public int[] getNumbers() { ... }
```
2.  **Deprecated Form**: C-style bracket placement fixed to the end of the signature.
```java
public int getNumbers()[] { ... } // Legal, though strongly discouraged
```

---

## 2. Formal Parameters and Arguments

### 2.1 Parameter Rules
- Parameters mandate comma-separated sequences within formal parentheses.
- The boundary parentheses are enforced syntactically even containing zero elements.
- Local parameters are inherently mutable internally but can optionally be cemented via `final`.
- Parameter types accept annotations cleanly (e.g., `@NonNull String name`).

### 2.2 Positional Argument Passing
- Java implements explicit **positional argument passing** locally. The caller submits arguments matching exactly the declared sequence architecture.
- **No Default Values**: Unlike some functional languages, Java lacks structural support for default or unassigned argument values directly modeled into method signatures.

---

## 3. Exception Handling (Throws Clause)

### 3.1 Checked Exceptions
- Methods harboring the capacity to propagate **checked exceptions** must formally outline them trailing a `throws` clause.
- Expanding exceptions follows a clean, comma-separated format.
- A declared method isn't strictly mandated to instantiate specific declared exceptions; a `throws` clause can act cleanly as an expansion placeholder for future implementations.

### 3.2 Unchecked Exceptions
- The Java compiler functionally disregards **unchecked exceptions** (e.g., `RuntimeException` or `NullPointerException`) declared inside a throws clause.
- While you can manually state them for Javadoc or architecture documentation, the compile engine enforces nothing surrounding unchecked definitions.

---

## 4. Method Body Returns
Excluding implementations bearing a `void` format, **every continuous execution path** structurally leaving a block (excluding those yielding thrown exceptions) must ultimately culminate delivering a value assignment-compatible corresponding with the stated return type. 

```java
// Compilation fails: 'int' fails direct assignment-compatibility to 'String'
public String testIt(int parameter) {
    final int x = 100;
    return x; // ERROR: Incompatible output types
}
```

---

## 5. Rules for Static Methods

### 5.1 Absolute Lack of `this`
Driven from static methods being innately woven to the class architectural template, they harbor completely zero implied `this` context acting to signify a local invocation object.

### 5.2 Interaction with Instance Fields
> **Important Consideration**: Often misattributed widely across engineering blogs, **static methods *are structurally capable* of interacting with instance fields**. However, they strictly require explicitly mapped reference points (an instantiated object) to address. Unqualified execution attempting to address isolated fields inherently fails.

```java
class Scratch {
    int x;

    // This static method needs an injected structural reference to access 'x'
    public static void doStuff(Scratch sc) {
        sc.x = 100; // Perfectly legal alteration to an instance field using reference
    }
}
```

### 5.3 Static Binding
- Static method resolution formally binds solely at direct compile time (Static Binding). Activation paths utilize identically the type-structure context that the compilation engine detects naturally, completely avoiding dynamic architectural polymorphism. 
- You may activate a static class configuration universally through an instance reference strictly as syntactic sugar, though most IDEs flag loud warnings. Importantly, this loose behavior is forbidden entirely addressing static interface methods.

---

[Back to Top](#table-of-contents)
