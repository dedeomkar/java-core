# Default Method Resolution

## Table of Contents
- [1. Fundamental Rules](#1-fundamental-rules)
  - [1.1 Object Class Restriction](#11-object-class-restriction)
  - [1.2 The Fallback Principle](#12-the-fallback-principle)
- [2. Conflict Resolution](#2-conflict-resolution)
  - [2.1 Class Precedence](#21-class-precedence)
  - [2.2 Qualified Super Syntax](#22-qualified-super-syntax)
  - [2.3 Implementation Hiding](#23-implementation-hiding)
- [3. Multiple Inheritance (Diamond Problem)](#3-multiple-inheritance-diamond-problem)
  - [3.1 Override-Equivalent Conflicts](#31-override-equivalent-conflicts)
  - [3.2 Resolving Ambiguity](#32-resolving-ambiguity)

---

## 1. Fundamental Rules

Default methods provide a way to add new functionality to interfaces while maintaining backward compatibility. However, they follow strict rules to avoid ambiguity.

### 1.1 Object Class Restriction

- **Constraint**: A default method must not match the signature of any method in `java.lang.Object`.
- **Reasoning**: Every class inherits from `Object`. Allowing a default method to override `Object` methods would lead to unpredictable behavior in the class hierarchy.

> [!CAUTION]
> Attempting to define `default boolean equals(Object obj)` in an interface will result in a compilation error.

```java
interface MyInterface {
    // ILLEGAL: Overrides java.lang.Object.equals
    // default boolean equals(Object o) { return true; } 
    
    // LEGAL: Abstract declaration of Object method (documentation only)
    boolean equals(Object o); 
}
```

### 1.2 The Fallback Principle

- **Definition**: Default methods are "fallbacks". They are only used if no other implementation is found in the class hierarchy.
- **Hierarchy Priority**: Any concrete implementation in a class (whether direct or inherited) always takes precedence over an interface's default method.

> [!NOTE]
> Imagine a default method as a **Safety Net**. If the class provides its own specialized equipment (implementation), the net isn't used.

[Back to Top](#table-of-contents)

---

## 2. Conflict Resolution

Java uses a "Class Always Wins" rule to resolve conflicts between class-provided methods and interface defaults.

### 2.1 Class Precedence

- If a class extends a parent class and implements an interface, and both provide a method with the same signature, the class's version (or its parent's version) is used.

```java
interface W {
    default void doStuff() { System.out.println("W default"); }
}

class B {
    public void doStuff() { System.out.println("B version"); }
}

class A extends B implements W {
    // B.doStuff() is used, ignoring W.default
}
```

### 2.2 Qualified Super Syntax

- To explicitly call a hidden default method from an interface, use the **Qualified Super** syntax.
- **Syntax**: `InterfaceName.super.methodName()`.

```java
class A implements W {
    @Override
    public void doStuff() {
        W.super.doStuff(); // Delegate to interface default
        System.out.println("Enhanced logic in A");
    }
}
```

> [!IMPORTANT]
> The prefix `W.super` is mandatory. Using just `super.doStuff()` looks for the method in the immediate parent class, not the interface.

### 2.3 Implementation Hiding

- Once a class provides a concrete implementation for a default method, that default method is "hidden" from subclasses of that class.
- Subclasses cannot bypass the parent class's implementation to reach the original interface default using `W.super`.

[Back to Top](#table-of-contents)

---

## 3. Multiple Inheritance (Diamond Problem)

When a class inherits from multiple interfaces that have conflicting method signatures, specific rules apply.

### 3.1 Override-Equivalent Conflicts

- If a class acquires two or more **override-equivalent** methods from interfaces, and at least one of them is `default`, the class **must** declare its own version of the method.
- This applies even if one is `default` and others are `abstract`.

> [!NOTE]
> Think of the Diamond Problem as receiving **two different sets of instructions** for the same assembly task. The worker (the class) cannot guess which one is correct and must explicitly decide which set to follow or combine them.

```java
interface IV { void doStuff(); }
interface IW { default void doStuff() { ... } }

// COMPILATION ERROR: Class must resolve the conflict
// class A implements IV, IW { } 
```

### 3.2 Resolving Ambiguity

A class can resolve the conflict by:
1.  Providing a completely new implementation.
2.  Choosing one of the interface implementations using the qualified super syntax.
3.  Stop making them defaults (change to abstract) and implement in the class.

```java
class A implements IV, IW {
    @Override
    public void doStuff() {
        IW.super.doStuff(); // Explicitly choose IW's version
    }
}
```

[Back to Top](#table-of-contents)
