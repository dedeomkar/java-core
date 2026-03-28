# Custom Exceptions

## Table of Contents
- [1. Subclassing the Throwable Hierarchy](#1-subclassing-the-throwable-hierarchy)
  - [1.1 Purpose of Custom Exceptions](#11-purpose-of-custom-exceptions)
  - [1.2 Choosing the Right Parent Class](#12-choosing-the-right-parent-class)
- [2. Implementation Details](#2-implementation-details)
  - [2.1 Constructors and State](#21-constructors-and-state)
  - [2.2 Company Conventions](#22-company-conventions)

---

## 1. Subclassing the Throwable Hierarchy

### 1.1 Purpose of Custom Exceptions

#### Concept Definition
- Subclassing `Throwable` (or its descendants) allows you to create custom exceptions.
- Custom exceptions are used to represent problems that are highly specific to your application's domain or business logic.
- They act as a precise fit when none of the standard Java exceptions (like `IllegalArgumentException` or `IOException`) accurately describe the unique problem your code encountered.

### 1.2 Choosing the Right Parent Class

#### Concept Definition
- When creating a custom exception, you must decide where it belongs in the exception hierarchy based on the nature of the error:
  - **Program Bug**: Subclass something in the `RuntimeException` subtree.
  - **Catastrophe**: Subclass something in the `Error` subtree (environmental issues from which recovery is impossible).
  - **Recoverable Issue**: Subclass something in the `Exception` subtree that is *not* a `RuntimeException` (thereby creating a checked exception).

#### Java-Specific Implementation
```java
// Recoverable Issue: Checked Exception
public class OrderProcessingException extends Exception {
    public OrderProcessingException(String message) {
        super(message);
    }
}

// Program Bug: Unchecked Exception
public class InvalidAccountStateException extends RuntimeException {
    public InvalidAccountStateException(String message) {
        super(message);
    }
}
```

---
[Back to Top](#table-of-contents) | [Back to Master TOC](./TOC.md)

## 2. Implementation Details

### 2.1 Constructors and State

#### Concept Definition
- Custom exceptions typically need several constructor variations to properly initialize the exception's context.
- **Message**: A `String` passed to the constructor designed to describe what went wrong to a developer debugging the system.
- **Underlying Cause**: Another `Throwable` passed to the constructor if your custom exception was triggered by a lower-level exception (e.g., an `SQLException` wrapped inside a custom `DatabaseFailureException`).
- It is crucial to use the `super()` keyword to pass these arguments up to the parent class constructor rather than ignoring them or handling them locally in the subclass.

#### Java-Specific Implementation
```java
public class ConfigurationLoadException extends RuntimeException {

    // Default constructor (Not very helpful)
    public ConfigurationLoadException() {
        super();
    }

    // Message constructor (Helpful for debugging)
    public ConfigurationLoadException(String message) {
        super(message);
    }

    // Message and Cause constructor (Essential for exception chaining)
    public ConfigurationLoadException(String message, Throwable cause) {
        // Pass both to the parent class to ensure they are properly stored
        super(message, cause);
    }
}
```

#### Best Practice / Consideration
> **Important Consideration**: Failing to pass the `message` and `cause` to `super()` is a common pitfall. If you define a constructor with those arguments but ignore them or fail to pass them to `super(message, cause)`, the exception stack trace and error message will be lost, rendering your custom exception useless for debugging.

### 2.2 Company Conventions

#### Concept Definition
- While the standard advice applies to choosing checked vs. unchecked bases, individual organizations often enforce local conventions.
- Some modern architectures and organizations mandate the use of **only** `RuntimeException`s for all custom exceptions to avoid polluting method signatures with extensive `throws` clauses.

#### Best Practice / Consideration
> **Important Consideration**: Always be aware of and adhere to your local organizational guidelines regarding exception design. What is technically correct by language definition might be forbidden by company policy.

---
[Back to Top](#table-of-contents) | [Back to Master TOC](./TOC.md)
