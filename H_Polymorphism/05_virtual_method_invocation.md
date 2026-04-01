# Virtual Method Invocation

## Table of Contents
- [1. Core Concepts](#1-core-concepts)
  - [1.1 Static vs. Dynamic Binding](#11-static-vs-dynamic-binding)
  - [1.2 Applicability of Virtual Invocation](#12-applicability-of-virtual-invocation)
- [2. Method Overriding Mechanics](#2-method-overriding-mechanics)
  - [2.1 The @Override Annotation](#21-the-override-annotation)
  - [2.2 Overriding vs. Overloading Pitfalls](#22-overriding-vs-overloading-pitfalls)
- [3. Accessing Parent Behavior](#3-accessing-parent-behavior)
  - [3.1 The super Keyword](#31-the-super-keyword)
  - [3.2 Hierarchy Restrictions](#32-hierarchy-restrictions)

---

## 1. Core Concepts

### 1.1 Static vs. Dynamic Binding

- **Reference Type (Compile-Time)**: The type of the reference variable determines which methods are *directly accessible* to the compiler.
- **Object Type (Runtime)**: The actual type of the object being referred to determines which *behavior* (implementation) is executed.
- **Late Binding**: This mechanism is also known as "dynamic method invocation" or "late binding."

> [!NOTE]
> Imagine a "Universal Remote Control" (Reference). The remote has buttons for "Play" and "Stop". If you point it at a DVD player, it plays a disc. If you point it at a Streaming box, it plays a video stream. The "buttons" are defined by the remote, but the "action" depends on the device you're pointing at.

```java
class Parent {
    void someMethod() { System.out.println("Parent implementation"); }
}

class Child extends Parent {
    @Override
    void someMethod() { System.out.println("Child implementation"); }
}

public class Main {
    public static void main(String[] args) {
        Parent p = new Child(); // Reference: Parent, Object: Child
        p.someMethod();         // Result: "Child implementation"
    }
}
```

### 1.2 Applicability of Virtual Invocation

- **Dynamic Binding Applies To**:
    - Non-private instance methods.
    - Non-final instance methods.
- **Static Binding Applies To**:
    - Private methods (not visible to subclasses).
    - Final methods (cannot be overridden).
    - Static methods (associated with the class, not the instance).
    - Fields (always determined by the reference type).

[Back to Top](#table-of-contents)

---

## 2. Method Overriding Mechanics

### 2.1 The @Override Annotation

- **Purpose**: Strictly a verification tool for the compiler.
- **Behavior**: It requests that the compiler issue an error if the annotated method is not actually overriding a method from a supertype.
- **Scope**: Includes implementing abstract methods or replacing default methods from interfaces.

### 2.2 Overriding vs. Overloading Pitfalls

- **Overriding**: Identical name, identical argument type sequence.
- **Common Error**: Accidentally creating an *overload* instead of an *override* due to slight misspellings or different argument types (e.g., `doStuff(int)` vs `doStuff(long)`).
- **Solution**: Always use `@Override` to catch these errors at compile time.

```java
class Parent {
    void doStuff(int x) {}
}

class Child extends Parent {
    // @Override // Error: misspelled name
    void dostuff(int x) {} 

    // @Override // Error: different argument type (Overload)
    void doStuff(long x) {} 
}
```

[Back to Top](#table-of-contents)

---

## 3. Accessing Parent Behavior

### 3.1 The super Keyword

- **Explicit Invocation**: Use `super.methodName()` to bypass the subclass's overridden implementation and invoke the parent's version of the behavior.
- **Usage Context**: Valid anywhere the `this` reference exists (instance methods and constructors).

```java
class Parent {
    void show() { System.out.println("Parent View"); }
}

class Child extends Parent {
    @Override
    void show() {
        super.show(); // Invokes Parent's show()
        System.out.println("Child specialized View");
    }
}
```

### 3.2 Hierarchy Restrictions

- **Immediate Only**: `super` only refers to the immediate parent's implementation that *would* have been inherited.
- **No Grandparents**: You cannot skip levels (e.g., `super.super.method()` is invalid syntax).

> [!IMPORTANT]
> Virtual method invocation is the cornerstone of polymorphism in Core Java, enabling decoupled, extensible architectures where the caller doesn't need to know the specific subclass to get specialized behavior.

[Back to Top](#table-of-contents)
