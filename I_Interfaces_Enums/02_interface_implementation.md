# Interface Implementation

## Table of Contents
- [1. Core Implementation Rules](#1-core-implementation-rules)
  - [1.1 Concrete Class requirements](#11-concrete-class-requirements)
  - [1.2 Abstract Classes and Interfaces](#12-abstract-classes-and-interfaces)
- [2. Annotation and Visibility](#2-annotation-and-visibility)
  - [2.1 The @Override Annotation](#21-the-override-annotation)
  - [2.2 Visibility of Inherited Abstract Methods](#22-visibility-of-inherited-abstract-methods)
- [3. Case Study: Interface Hierarchy](#3-case-study-interface-hierarchy)
  - [3.1 Scenario Definition](#31-scenario-definition)
  - [3.2 Implementation Comparison](#32-implementation-comparison)

---

## 1. Core Implementation Rules

### 1.1 Concrete Class requirements

- **Full Implementation**: When a concrete class declares that it `implements` an interface, it **must** provide concrete bodies for all abstract methods.
- **Transitive Requirement**: This includes all abstract methods inherited from parent interfaces (interfaces that the target interface `extends`).

> [!NOTE]
> Think of an interface as a **Contract**. A concrete class is like a **Contractor** who signs it. If the contract says "Build a house" and "Install plumbing," the contractor cannot skip the plumbing and still claim the job is done.

```java
interface Shape { void draw(); }
interface ColorableShape extends Shape { void color(); }

// Concrete class MUST implement both draw() and color()
public class Circle implements ColorableShape {
    @Override
    public void draw() { /* implementation */ }
    
    @Override
    public void color() { /* implementation */ }
}
```

### 1.2 Abstract Classes and Interfaces

- **Deferred Implementation**: Abstract classes and interfaces are not required to implement abstract methods they inherit.
- **Abstract Nature**: The resulting type remains abstract and can continue to contain abstract methods.
- **Subclass Responsibility**: Any concrete class that eventually derives from these abstract types must provide the missing implementations.

[Back to Top](#table-of-contents)

---

## 2. Annotation and Visibility

### 2.1 The @Override Annotation

- **Verification**: The `@Override` annotation can be used when:
    - Providing a concrete implementation for an abstract method from a parent type.
    - A default method provides an implementation for an abstract or default method in a parent interface.

### 2.2 Visibility of Inherited Abstract Methods

- **Implicit Existence**: A derived type (abstract class or interface) can simply ignore methods it leaves abstract in its source code.
- **Hidden Presence**: These methods still exist in the type's contract, even if they aren't explicitly redeclared or visible in the derived type's source file.

> [!TIP]
> Always use `@Override`. It's your **Safety Net**. If you accidentally change a method signature (e.g., `doStuff(int x)` to `doStuff(long x)`), the compiler will immediately warn you that you're no longer overriding the intended method.

[Back to Top](#table-of-contents)

---

## 3. Case Study: Interface Hierarchy

### 3.1 Scenario Definition

Consider the following hierarchy of interfaces:

- **Interface T**: Declares two abstract methods: `doSomething()` and `doSomethingElse()`.
- **Interface U**: Extends **T**.
    - Provides a `default` implementation for `doSomething()`.
    - `doSomethingElse()` remains abstract.
- **Interface V**: Extends **U**.
    - Redeclares `doSomething()` as **abstract**.
    - Both methods are now abstract again for any implementer of **V**.

### 3.2 Implementation Comparison

| Type | Target | Status | Reason |
| :--- | :--- | :--- | :--- |
| **Class A** | `implements T` | **FAIL** | No implementations provided for `doSomething()` or `doSomethingElse()`. |
| **Abstract B** | `implements T` | **VALID** | Abstract classes can leave methods unimplemented. |
| **Class C** | `implements V` | **FAIL** | Missing `doSomething()`. Even though **U** provided a default, **V** made it abstract again. |
| **Class D** | `implements U` | **VALID** | Provides `doSomethingElse()`. Uses the `default` implementation of `doSomething()` from **U**. |

```java
interface T {
    void doSomething();
    void doSomethingElse();
}

interface U extends T {
    default void doSomething() { /* fallback */ }
}

interface V extends U {
    void doSomething(); // Redeclared abstract, hiding U's default
}

// VALID: Class D only needs to implement doSomethingElse
class D implements U {
    public void doSomethingElse() { 
        System.out.println("Doing something else...");
    }
}
```

[Back to Top](#table-of-contents)
