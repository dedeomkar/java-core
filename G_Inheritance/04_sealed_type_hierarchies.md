# Sealed Type Hierarchies

## Table of Contents
- [1. Fundamentals of Sealed Types](#1-fundamentals-of-sealed-types)
  - [1.1 Definition and Purpose](#11-definition-and-purpose)
  - [1.2 Controlled Hierarchies](#12-controlled-hierarchies)
- [2. Defining Sealed Types](#2-defining-sealed-types)
  - [2.1 The Sealed Keyword](#21-the-sealed-keyword)
  - [2.2 The Permits Clause](#22-the-permits-clause)
- [3. Controlling Subtype Extension](#3-controlling-subtype-extension)
  - [3.1 Final Subtypes](#31-final-subtypes)
  - [3.2 Sealed Subtypes](#32-sealed-subtypes)
  - [3.3 Non-Sealed Subtypes](#33-non-sealed-subtypes)
- [4. Structural Rules and Constraints](#4-structural-rules-and-constraints)
  - [4.1 Hierarchy Requirements](#41-hierarchy-requirements)
  - [4.2 Module and Package Restrictions](#42-module-and-package-restrictions)
- [5. Low-Latency & Performance Impact](#5-low-latency--performance-impact)

---

## 1. Fundamentals of Sealed Types

### 1.1 Definition and Purpose

- **Sealed Types**: A mechanism introduced in Java 17 to limit which classes or interfaces may extend or implement them.
- **Controlled Generalization**: Prevents arbitrary, unexpected new subtypes from being introduced into a carefully designed hierarchy.
- **External Behavior**: Particularly useful when behavior is external to the types (e.g., business logic using `instanceof` patterns).

### 1.2 Controlled Hierarchies

- **Traditional OOP**: Polymorphism allows any number of unknown subtypes to "do their thing."
- **Sealed Design**: Shifts the focus to a closed set of known types, enabling safer logic that doesn't "leak" when new classes are added.

> [!NOTE]
> **The Authorized Driver Analogy**: Barron and Olivia explain that a "Traditional" hierarchy is like a public road—anyone can drive any car they want. A **Sealed** hierarchy is like a high-security facility where only specific authorized drivers (permitted subtypes) are allowed entry.

[Back to Top](#table-of-contents)

---

## 2. Defining Sealed Types

### 2.1 The Sealed Keyword

- **Syntax**: Use the `sealed` modifier before `class` or `interface`.
- **Root Node**: The base type must indicate its authorized descendants.

### 2.2 The Permits Clause

- **Enumeration**: The `permits` keyword lists the direct subtypes allowed to extend the root.
- **Placement**: Must follow `extends` and `implements` clauses.

```java
public sealed interface Vehicle permits Car, Truck {
    default void add(String item) {
        System.out.println("Adding " + item + " to vehicle.");
    }
}
```

[Back to Top](#table-of-contents)

---

## 3. Controlling Subtype Extension

Every permitted subtype MUST explicitly state how it handles further inheritance using one of three modifiers:

### 3.1 Final Subtypes

- **No Further Extension**: The hierarchy stops here. Most common for "leaf" nodes.
- **Syntax**: `public final class Car implements Vehicle { ... }`

### 3.2 Sealed Subtypes

- **Nested Control**: The subtype itself is sealed and must provide its own `permits` clause.
- **Syntax**: `public sealed class Truck implements Vehicle permits SemiTruck { ... }`

### 3.3 Non-Sealed Subtypes

- **Open Extension**: Re-opens the hierarchy for arbitrary subclasses. 
- **Syntax**: `public non-sealed class SemiTruck extends Truck { ... }`
- **Context**: `non-sealed` is a hyphenated keyword (restricted identifier).

```java
// Leaf node - no more children
public final class Car implements Vehicle {
    private int seats;
    // ...
}

// Intermediate node - controlled children
public sealed class Truck implements Vehicle permits HeavyTruck {
    private int payload;
    // ...
}

// Open node - anyone can extend
public non-sealed class HeavyTruck extends Truck {
    // ...
}
```

[Back to Top](#table-of-contents)

---

## 4. Structural Rules and Constraints

### 4.1 Hierarchy Requirements

- **No Empty Sealed Types**: A `sealed` type must have at least one permitted subtype.
- **Implicit Rules**: Concrete methods and `instanceof` checks become safer because the compiler knows the full list of possibilities.
- **Keyword Order**: `extends` -> `implements` -> `permits`.

### 4.2 Module and Package Restrictions

- **JPMS (Modular)**: Root and subtypes can be in different packages but MUST be in the same module.
- **Non-Modular**: Root and all subtypes MUST be in the same package.

> [!WARNING]
> **Constraint Failure**: If you attempt to add a `Bicycle` class to the `Vehicle` hierarchy without listing it in the `permits` clause, the compiler will reject it, alerting you that the existing design (like weight calculations) wasn't built to handle it.

[Back to Top](#table-of-contents)

---

## 5. Low-Latency & Performance Impact

> [!IMPORTANT]
> **Exhaustiveness and Dispatch**: In low-latency Spring or Core Java applications, sealed types enable the compiler to perform **exhaustiveness checks**. When combined with pattern matching (available in newer Java versions), it can replace expensive "default" cases in business logic with direct, optimized jumps.

- **Reduced Runtime Errors**: By preventing unknown subtypes, you eliminate the risk of "Falling through" safely designed logic, which is critical for high-stakes financial or low-latency systems.
- **Static Analysis**: Tools can better optimize code when the entire inheritance tree is known at compile time.

[Back to Top](#table-of-contents)
