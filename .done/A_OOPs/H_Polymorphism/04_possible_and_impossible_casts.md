# Possible and Impossible Casts

## Table of Contents
- [1. Reference Casting Rules](#1-reference-casting-rules)
  - [1.1 Casting Down the Hierarchy](#11-casting-down-the-hierarchy)
  - [1.2 Casting Up the Hierarchy](#12-casting-up-the-hierarchy)
  - [1.3 Casting Across Class Hierarchies](#13-casting-across-class-hierarchies)
- [2. Interface Casting Dynamics](#2-interface-casting-dynamics)
  - [2.1 Open-Ended Interface Relationships](#21-open-ended-interface-relationships)
  - [2.2 The "Future Subclass" Rationale](#22-the-future-subclass-rationale)
- [3. Final Class and Enum Constraints](#3-final-class-and-enum-constraints)
  - [3.1 Final Class Restrictions](#31-final-class-restrictions)
  - [3.2 Enums as Implicitly Final](#32-enums-as-implicitly-final)
- [4. Verification and Safety](#4-verification-and-safety)
  - [4.1 Compile-time vs Runtime Failures](#41-compile-time-vs-runtime-failures)

---

## 1. Reference Casting Rules

The Java compiler enforces specific rules when shifting the "lens" through which we view an object.

### 1.1 Casting Down the Hierarchy

- **Rule**: Casting from a superclass to a subclass.
- **Compilation**: Always permitted by the compiler.
- **Runtime**: Fails with a `ClassCastException` if the object is not actually an instance of the target type.

> [!CAUTION]
> Downcasting is a speculative operation. You are telling the compiler, "I know better than you what this object actually is."

```java
Object obj = "Hello Java";
String s = (String) obj; // Valid downcast
// Integer i = (Integer) obj; // Compiles, but throws ClassCastException at runtime
```

### 1.2 Casting Up the Hierarchy

- **Rule**: Casting from a subclass to a superclass.
- **Compilation**: Always valid and often redundant.
- **Effect**: Widens the reference; the compiler now only permits access to superclass members.

### 1.3 Casting Across Class Hierarchies

- **Rule**: Casting between two classes that share no parent-child relationship (e.g., `String` to `Integer`).
- **Result**: Rejected at **compile-time**.
- **Reason**: There is no possible "is-a" relationship; a `String` can never be an `Integer`.


---

## 2. Interface Casting Dynamics

Casting involving interfaces is significantly more flexible than class-to-class casting.

### 2.1 Open-Ended Interface Relationships

- **Rule**: In many cases, the compiler permits casting between a class and an interface even if no direct relationship exists at compile-time.

### 2.2 The "Future Subclass" Rationale

- The compiler assumes that a future subclass of the current class might implement the interface.
- **Example**: `ArrayList` does not implement `Runnable`, but the compiler allows the cast because a developer could create `MyRunnableArrayList extends ArrayList implements Runnable`.

```java
ArrayList list = new ArrayList();
// Permitted: A subclass of ArrayList might implement Runnable
Runnable r = (Runnable) list; 
```


---

## 3. Final Class and Enum Constraints

The flexibility of interface casting disappears when the class hierarchy is "closed."

### 3.1 Final Class Restrictions

- If a class is marked `final`, it cannot have subclasses.
- If a `final` class doesn't implement an interface, the compiler knows it is **impossible** for any instance of that class to ever satisfy the interface.
- **Result**: Casting is rejected at compile-time.

```java
String s = "Finality";
// COMPILE ERROR: String is final and doesn't implement Runnable
// Runnable r = (Runnable) s; 
```

### 3.2 Enums as Implicitly Final

- Enums are implicitly `final` in Java.
- They follow the same rigid casting rules as final classes regarding interfaces they do not implement.

> [!IMPORTANT]
> The `final` modifier acts as a lock on the hierarchy, allowing the compiler to perform much stricter static analysis on casting possibilities.


---

## 4. Verification and Safety

### 4.1 Compile-time vs Runtime Failures

- **Compile-time Failure**: Occurs when the cast is logically impossible (across class hierarchies or involving final classes).
- **Runtime Failure (`ClassCastException`)**: Occurs when the cast is logically possible but the actual object at the memory address is not compatible.

> [!TIP]
> Use the `instanceof` operator to verify assignment compatibility before performing a downcast to avoid runtime crashes.

