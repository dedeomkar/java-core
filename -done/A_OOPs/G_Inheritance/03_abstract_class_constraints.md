# Abstract Class Constraints

## Table of Contents
- [1. Fundamentals of Abstract Types](#1-fundamentals-of-abstract-types)
  - [1.1 Definition and Purpose](#11-definition-and-purpose)
  - [1.2 Instantiation Rules](#12-instantiation-rules)
- [2. Class-Level Constraints](#2-class-level-constraints)
  - [2.1 Finality and Abstractness](#21-finality-and-abstractness)
  - [2.2 Inheritance Limits](#22-inheritance-limits)
- [3. Members and Accessibility](#3-members-and-accessibility)
  - [3.1 Fields and Concrete Methods](#31-fields-and-concrete-methods)
  - [3.2 Constructors and Initialization](#32-constructors-and-initialization)
  - [3.3 Nested Types](#33-nested-types)
- [4. Abstract Method Rules](#4-abstract-method-rules)
  - [4.1 Syntax and Structure](#41-syntax-and-structure)
  - [4.2 Modifier Constraints](#42-modifier-constraints)
- [5. Low-Latency & Performance Impact](#5-low-latency--performance-impact)

---

## 1. Fundamentals of Abstract Types

### 1.1 Definition and Purpose

- **Abstract Types**: Encompass both `abstract` classes and `interfaces`.
- **Function**: They serve as a template or "contract" for specialized subtypes.
- **Abstract Methods**: Both types permit the declaration of methods without a body (implementation).

### 1.2 Instantiation Rules

- **Direct Instantiation**: Prohibited. You cannot use the `new` keyword on an abstract class.
- **Subtype Requirement**: An abstract class can only exist in memory as part of a concrete (non-abstract) subclass instance.

> [!NOTE]
> **The Blueprint Analogy**: Think of an abstract class as a blueprint for a "Generic Engine." You cannot drive a blueprint; you can only drive a physical engine (concrete class) built according to those specifications.


---

## 2. Class-Level Constraints

### 2.1 Finality and Abstractness

- **Mutual Exclusivity**: A class cannot be both `abstract` and `final`.
- **Reasoning**: `abstract` implies it *must* be subclassed to be useful, while `final` explicitly *forbids* subclassing.

### 2.2 Inheritance Limits

- **Single Parent**: Like all Java classes, an abstract class is restricted to a single parent class (even if the parent is also abstract).
- **Multiple Interfaces**: It can, however, implement multiple interfaces.

```java
public abstract class BaseProcessor extends RootHandler implements Runnable, AutoCloseable {
    // Valid: One parent, multiple interfaces
}
```


---

## 3. Members and Accessibility

### 3.1 Fields and Concrete Methods

Unlike interfaces (pre-Java 8), abstract classes offer full flexibility for state and logic:
- **Fields**: Can define `static` and `instance` fields with any accessibility (`private`, `protected`, `public`).
- **Concrete Methods**: Can implement full logic in `static` or `instance` methods.

### 3.2 Constructors and Initialization

- **Subclass Support**: Abstract classes provide constructors to facilitate the initialization of state by their concrete subclasses.
- **Implicit Constructor**: If none is defined, a default zero-argument constructor is provided.
- **Private Constructors**: If all constructors are `private`, only nested classes within that abstract class can extend it.

### 3.3 Nested Types

Abstract classes can contain:
- **Nested Classes**: Static or instance.
- **Nested Interfaces**: These are implicitly **static**, regardless of modifiers.

```java
public abstract class DatabaseConnector {
    private String connectionString; // Instance field

    public DatabaseConnector(String uri) { // Constructor for subclasses
        this.connectionString = uri;
    }

    public static void log(String msg) { // Static concrete method
        System.out.println("LOG: " + msg);
    }
}
```


---

## 4. Abstract Method Rules

### 4.1 Syntax and Structure

- **Termination**: Must end with a semicolon `;` instead of a curly-brace method body `{}`.
- **Placement**: Only permitted within abstract classes or interfaces.

### 4.2 Modifier Constraints

- **Non-Final**: Abstract methods cannot be `final` (they must be overridden).
- **Non-Private**: They cannot be `private` (subclasses must see them to implement them).
- **Instance Only**: Static methods cannot be abstract.
- **Default Keyword**: The `default` keyword is reserved for interfaces and cannot be used in abstract classes.

```java
public abstract class Shape {
    // Correct abstract method declaration
    public abstract double calculateArea(); 

    // COMPILATION ERROR:
    // private abstract void hideMe();
    // public final abstract void lockMe();
    // public static abstract void staticAbstract();
}
```


---

## 5. Low-Latency & Performance Impact

> [!IMPORTANT]
> **Virtual Method Invocations**: Invoking an abstract method involves a virtual table (`vtable`) lookup at runtime. In extreme low-latency environments, deep abstract hierarchies can introduce slight overhead. Favor "Flat" hierarchies where possible to reduce lookup depth.

- **State Locality**: Use protected fields in abstract classes to keep data close to the logic that operates on it, improving L1/L2 cache hits compared to accessing data via interface methods.

