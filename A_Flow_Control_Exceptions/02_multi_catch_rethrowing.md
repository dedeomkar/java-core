# Multi-Catch Behavior and Rethrowing

## Table of Contents
- [1. Multi-Catch Syntax and Rules](#1-multi-catch-syntax-and-rules)
  - [1.1 Purpose of Multi-Catch](#11-purpose-of-multi-catch)
  - [1.2 Usage and Constraints](#12-usage-and-constraints)
- [2. Rethrowing Checked Exceptions](#2-rethrowing-checked-exceptions)
  - [2.1 Precise Rethrowing with Multi-Catch](#21-precise-rethrowing-with-multi-catch)
  - [2.2 Reassignment Restrictions](#22-reassignment-restrictions)

---

## 1. Multi-Catch Syntax and Rules

### 1.1 Purpose of Multi-Catch

#### Concept Definition
- When different exceptions require identical handling, duplicating `catch` blocks or catching a highly generic parent class (like `Exception`) are poor solutions.
- Catching a common ancestor can accidentally catch unrelated sibling exceptions that you didn't intend to handle.
- The **multi-catch** syntax solves this by allowing a single `catch` block to handle a specific, exhaustive list of exception types without code duplication.

#### Java-Specific Implementation
```java
try {
    // Code that might throw unrelated exceptions
    processMediaFiles();
} catch (FileNotFoundException | MidiUnavailableException ex) {
    // Both exceptions are caught here.
    // The vertical bar '|' acts as an "either/or".
    System.out.println("Could not process file: " + ex.getMessage());
}
```

### 1.2 Usage and Constraints

#### Concept Definition
- There is no limit to the number of exceptions you can chain using the vertical bar `|`.
- **Constraint**: The exception types listed in a multi-catch MUST NOT have a parent-child relationship. Attempting to catch both a parent and its child in the same multi-catch results in a compiler error.
- The compiler assigns the reference variable (`ex` in the example above) the type of the **nearest common ancestor** of the listed exceptions. 

#### Best Practice / Consideration
> **Important Consideration**: If you feel handicapped because the reference variable only exposes methods of the common ancestor, it implies you're trying to use specific features of the subtypes. In that case, you probably shouldn't be combining their handling, as the handling is no longer identical!

---

## 2. Rethrowing Checked Exceptions

### 2.1 Precise Rethrowing with Multi-Catch

#### Concept Definition
- Generally, if a checked exception is rethrown from a `catch` block, the method signature must declare that it `throws` that exception.
- When catching exceptions via a broad parent type or a multi-catch block, modern Java uses **precise rethrow** analysis.
- If the catch parameter variable is **final** or **effectively final** (unmodified inside the catch block) and you throw that exact parameter, you can declare the specific exception types in the `throws` clause rather than the generic common ancestor.

#### Java-Specific Implementation
```java
// Method only needs to declare the specific underlying exceptions, 
// even though 'e' statically resolves to Exception.
public void preciseRethrow() throws SQLException, FileNotFoundException {
    try {
        if (Math.random() > 0.5) throw new SQLException();
        else throw new FileNotFoundException();
    } catch (SQLException | FileNotFoundException e) {
        // Log it, then rethrow
        System.out.println("Logging error before throwing");
        throw e; // 'e' is effectively final and directly thrown
    }
}
```

### 2.2 Reassignment Restrictions

#### Concept Definition
- If you copy the caught exception into a new variable or modify the catch parameter before rethrowing it, the compiler loses the ability to trace the precise specific types being thrown.
- In that scenario, you lose the precise rethrow feature and are forced to declare the broader common ancestor (e.g., `throws Exception`).

#### Java-Specific Implementation
```java
public void brokenPreciseRethrow() throws Exception { 
    // Must throw Exception because the precise type is lost during reassignment
    try {
        if (Math.random() > 0.5) throw new SQLException();
        else throw new FileNotFoundException();
    } catch (SQLException | FileNotFoundException e) {
        Exception newEx = e; // Assignment breaks precise rethrow
        throw newEx;
    }
}
```

#### Best Practice / Consideration
> **Important Consideration**: Precise rethrowing is a relatively modern Java feature. If you read old code or tutorials, you might see them claiming that catching and rethrowing `Exception` forces the method signature to `throws Exception`. In modern Java, this is not required as long as the exception variable remains effectively final and is exactly the one thrown.

---
[Back to Top](#table-of-contents) | [Back to Master TOC](./TOC.md)
