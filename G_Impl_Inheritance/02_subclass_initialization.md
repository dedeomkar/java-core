# Subclass Initialization

## Table of Contents
- [1. Fundamentals of Initialization](#1-fundamentals-of-initialization)
- [2. Constructor Delegation](#2-constructor-delegation)
  - [2.1 The `super()` and `this()` Keywords](#21-the-super-and-this-keywords)
  - [2.2 Delegation Rules and Constraints](#22-delegation-rules-and-constraints)
- [3. Detailed Initialization Order](#3-detailed-initialization-order)
  - [3.1 Execution Flow](#31-execution-flow)
  - [3.2 Example: Analyzing the Trace](#32-example-analyzing-the-trace)
- [4. Best Practices & Critical Pitfalls](#4-best-practices--critical-pitfalls)

---

## 1. Fundamentals of Initialization

Java enforces a strict hierarchical initialization protocol to ensure that an object's foundations are fully constructed before its specific features are added.

- **Foundational Hierarchy**: Every class is built upon its parent. Initialization always flows from the top of the hierarchy (`java.lang.Object`) down to the specific subclass.
- **Foundations-First Principle**: You cannot build the roof of a house before the foundation and walls are set. Similarly, Java ensures that the parent parts of an object are initialized before the child parts.

---

## 2. Constructor Delegation

Constructors are the primary mechanism for coordinating initialization. They don't just set fields; they manage the delegatory chain up the hierarchy.

### 2.1 The `super()` and `this()` Keywords

- **`super()`**: Invokes a constructor of the immediate parent class.
- **`this()`**: Invokes another constructor within the *same* class (constructor overloading).
- **Implicit `super()`**: If a constructor does not explicitly call `this()` or `super()`, the compiler automatically inserts `super()` (the no-argument call) as the very first line.

### 2.2 Delegation Rules and Constraints

- **Mutual Exclusion**: A constructor can call *either* `this()` or `super()`, but never both.
- **The First-Line Rule**: Any call to `this()` or `super()` MUST be the first statement in the constructor.
- **No Circularity**: Constructors cannot call each other in a circle (e.g., A calls B, and B calls A). This results in a compiler error.
- **No Recursion**: A constructor cannot call itself.

```java
public class Base {
    public Base() {
        System.out.println("Base Initialized");
    }
}

public class Sub extends Base {
    public Sub() {
        this(10); // Delegates to the other constructor in Sub
    }

    public Sub(int x) {
        super(); // Explicitly calls Base constructor (optional, implicit by default)
        System.out.println("Sub Initialized with " + x);
    }
}
```

[Back to Top](#table-of-contents)

---

## 3. Detailed Initialization Order

The sequence of events during `new Subclass()` is deterministic and follows a three-stage process.

### 3.1 Execution Flow

1.  **Stage 1: Parent Delegation**: The constructor delegates up the chain via `super()` until it reaches `java.lang.Object`.
2.  **Stage 2: Instance Initialization**: Upon returning from the parent constructor, the JVM performs "instance initialization" for the current class:
    -   Initializing instance fields (e.g., `int x = 10;`).
    -   Executing instance initializer blocks (`{ ... }`).
3.  **Stage 3: Constructor Body**: Finally, the code within the constructor's body is executed.

### 3.2 Example: Analyzing the Trace

Consider the following hierarchy:

```java
class Y {
    { System.out.println("Initializing Y"); }
}

class X extends Y {
    { System.out.println("Initializing X"); }
    public X() { System.out.println("X()"); }
    public X(int val) {
        this();
        System.out.println("X(" + val + ")");
    }
}

// Execution: new X(1)
```

**Trace Results:**
1.  Enter `X(1)` -> calls `this()`.
2.  Enter `X()` -> calls implicit `super()`.
3.  Run `Y`'s instance initializer -> **"Initializing Y"**.
4.  Return to `X()` -> Run `X`'s instance initializer -> **"Initializing X"**.
5.  Run body of `X()` -> **"X()"**.
6.  Return to `X(1)` -> Run body of `X(1)` -> **"X(1)"**.

[Back to Top](#table-of-contents)

---

## 4. Best Practices & Critical Pitfalls

> [!WARNING]
> **Avoid Overridable Methods in Constructors**: Never invoke a method in a constructor that can be overridden by a subclass. The subclass version will execute *before* the subclass fields have been initialized, leading to `NullPointerException` or inconsistent state.

```java
abstract class Y {
    int x = 100;
    Y(int v) {
        x = v;
        announce();
    }
    abstract void announce();
}

public class X extends Y {
    int x = 300;
    X() { super(200); }
    void announce() { 
        System.out.println("x is " + x); 
    }
    public static void main(String[] args) {
        new X(); 
    }
}
```

### Step-by-Step Execution Trace

1.  **Start `new X()`**: The JVM begins the allocation process for a new `X` object.
2.  **Parent Delegation**: `X()` constructor is called, which immediately delegates to `super(200)`.
3.  **Parent Initialization (`Y`)**: 
    -   `Y`'s instance fields are initialized: `Y.x` becomes `100`.
    -   `Y(int v)` body executes: `Y.x` is set to `200`.
4.  **Polymorphic Call**: `Y` calls `announce()`. Because the actual object is of type `X`, it invokes `X.announce()`.
5.  **Field Access Paradox**: `X.announce()` prints `"x is " + x`. At this precise moment, the execution is still inside the `super(200)` call. This means **`X`'s own instance initialization (where `x = 300` happens) has not yet occurred**.
6.  **Default Value**: Since `X.x` has not been initialized yet, it holds its default value for an `int`, which is `0`.
7.  **Result**: The code prints **`x is 0`**.
8.  **Completion**: After `super(200)` returns, `X.x` is finally initialized to `300`, and the body of `X()` executes.

> **Initialization Safety**: Ensure that all critical fields are initialized either in their declaration or in an instance initializer block to ensure they are ready before the constructor body runs.  
> **Constructor Accessibility**: A subclass must have access to at least one parent constructor. If the parent only provides `private` constructors, it cannot be subclassed (except by nested types).

[Back to Top](#table-of-contents)
