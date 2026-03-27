# Suppressed Exceptions in Try-With-Resources

## Table of Contents
- [1. Primary vs. Suppressed Exceptions](#1-primary-vs-suppressed-exceptions)
  - [1.1 Exception Prioritization](#11-exception-prioritization)
  - [1.2 The Primary Exception](#12-the-primary-exception)
  - [1.3 Suppressed Exceptions](#13-suppressed-exceptions)
- [2. Handling and Retrieval](#2-handling-and-retrieval)
  - [2.1 Catch Block Mechanics](#21-catch-block-mechanics)
  - [2.2 Retrieving Suppressed Exceptions](#22-retrieving-suppressed-exceptions)

---

## 1. Primary vs. Suppressed Exceptions

### 1.1 Exception Prioritization

#### Concept Definition
- In a `try-with-resources` block, multiple exceptions can potentially be thrown sequentially: one from the `try` block itself, and several others from the `close()` methods of the various resources as they fail to close.
- The compiler-generated code catches all of these exceptions and stores them in the exact order that they arose.

### 1.2 The Primary Exception

#### Concept Definition
- After the closure process completes for all resources, the **first stored exception** becomes the "primary exception".
- If an exception occurred in the `try` block, it will always be the primary exception (since the code tries the `try` block before attempting to close anything).
- If the `try` block succeeded, but the first resource to close (the last one listed) failed, that closure exception becomes the primary exception.
- The primary exception is the one that is actually re-thrown to the surrounding code or `catch` blocks.

### 1.3 Suppressed Exceptions

#### Concept Definition
- All subsequent exceptions—everything except the first one—are officially termed **"suppressed exceptions"**.
- Suppressed exceptions do not get thrown individually. Instead, they are collected into an array and attached directly to the primary exception object.

#### Everyday Analogy
- Imagine you are carrying a stack of boxes (the `try` block) and trip, dropping the main box (the Primary Exception). As you scramble to pick it up, you accidentally knock over two more boxes (the Suppressed Exceptions). When you report the incident to your boss, you say "I tripped, and by the way, here is a list of other things I consequently knocked over." The trip is the primary issue, the others are suppressed details attached to the main report.

---

## 2. Handling and Retrieval

### 2.1 Catch Block Mechanics

#### Concept Definition
- A user-supplied `catch` block will **only** ever match against the primary exception.
- If the primary exception is a `FileNotFoundException` and a suppressed exception is an `SQLException`, a `catch (SQLException e)` block will **not** catch anything, because the primary exception dictates the flow. The `SQLException` is merely riding along inside the `FileNotFoundException`.

#### Java-Specific Implementation
```java
try (MyResource r1 = new MyResource();
     MyResource r2 = new MyResource()) {
     
    throw new FileNotFoundException("Failed in try"); // Becomes primary
    
} catch (SQLException e) {
    // This will NOT execute, even if r1 or r2 throw SQLException on close.
    // The primary exception is FileNotFoundException.
} catch (FileNotFoundException e) {
    // This WILL execute. 
    System.out.println("Primary exception caught: " + e.getMessage());
}
```

### 2.2 Retrieving Suppressed Exceptions

#### Concept Definition
- Once you catch the primary exception, you can interrogate it to find out if anything else went wrong during the closure process.
- The `Throwable` class provides a `getSuppressed()` method that returns an array of `Throwable` objects (`Throwable[]`).

#### Java-Specific Implementation
```java
try (MyResource r1 = new MyResource(1); 
     MyResource r2 = new MyResource(2)) {
    // Code that might succeed or throw
} catch (Throwable t) { // Catching the primary problem
    System.out.println("Primary Problem: " + t.getMessage());
    
    // Retrieve any suppressed exceptions
    Throwable[] suppressed = t.getSuppressed();
    for (Throwable supp : suppressed) {
        System.out.println("Suppressed Exception: " + supp.getMessage());
    }
}
```

#### Best Practice / Consideration
> **Important Consideration**: Always use `try-with-resources` when dealing with multiple resources that might throw exceptions on close. Before Java 7, manually writing nested `try-finally` blocks to close multiple resources without losing the original exception (exception masking) was notoriously complex and error-prone. This structure solves that elegantly by guaranteeing closure attempts and preserving all thrown exceptions via suppression.

---
[Back to Top](#table-of-contents) | [Back to Master TOC](./TOC.md)
