# Covariant Returns

## Table of Contents
- [1. Overriding Fundamentals](#1-overriding-fundamentals)
  - [1.1 Liskov Substitution Principle](#11-liskov-substitution-principle)
  - [1.2 Essential Overriding Rules](#12-essential-overriding-rules)
- [2. Covariant Return Types (Reference Types)](#2-covariant-return-types-reference-types)
  - [2.1 Definition and Mechanic](#21-definition-and-mechanic)
  - [2.2 Valid Covariant Examples](#22-valid-covariant-examples)
- [3. Primitive Return Type Constraints](#3-primitive-return-type-constraints)
  - [3.1 Identity Requirement](#31-identity-requirement)
  - [3.2 Common Pitfalls](#32-common-pitfalls)
- [4. Verification and Best Practices](#4-verification-and-best-practices)
  - [4.1 The @Override Safety Net](#41-the-override-safety-net)

---

## 1. Overriding Fundamentals

### 1.1 Liskov Substitution Principle

- When one method overrides another, it must not break the expectations created by the original method.
- This is formally known as the **Liskov Substitution Principle**.
- A subclass object should be usable anywhere its parent is expected without the caller needing to know the difference.

> [!NOTE]
> Imagine ordering a "hot beverage". You expect something warm to drink. If the vendor gives you a "Coffee" (subclass), they've fulfilled the expectation. If they give you a "Bowl of Soup" (different hierarchy), they've broken the contract.

### 1.2 Essential Overriding Rules

The Java compiler enforces several syntax rules to maintain this contract:
- **Accessibility**: The overriding method's accessibility must **not be less** than the parent's. (e.g., `protected` can become `public`, but not `private`).
- **Checked Exceptions**: An overriding method cannot declare **new or broader** checked exceptions than those in the parent method.

[Back to Top](#table-of-contents)

---

## 2. Covariant Return Types (Reference Types)

### 2.1 Definition and Mechanic

- Java allows an overriding method to return a **more specific type** (subtype) than the one declared in the parent method.
- This is called a **Covariant Return Type**.
- This is only applicable to **Reference Types**.

> [!TIP]
> This is extremely useful for avoiding unnecessary casting in the calling code when it is known that a specific implementation returns a specific type.

### 2.2 Valid Covariant Examples

If a parent method returns `Number`, any subclass of `Number` is a valid return type for the override.

```java
import java.math.BigDecimal;
import java.util.concurrent.atomic.AtomicInteger;

class Base {
    public Number getValue() { return 10; }
}

class Extended extends Base {
    @Override
    public Integer getValue() { return 20; } // Valid: Integer is a Number
}

class Specialized extends Base {
    @Override
    public BigDecimal getValue() { return new BigDecimal("3.14"); } // Valid
}

class Concurrent extends Base {
    @Override
    public AtomicInteger getValue() { return new AtomicInteger(100); } // Valid
}
```

> [!WARNING]
> You cannot return a completely unrelated type like `String` even if it's a reference type, as it doesn't satisfy the "is-a" relationship with `Number`.

[Back to Top](#table-of-contents)

---

## 3. Primitive Return Type Constraints

### 3.1 Identity Requirement

- Unlike reference types, **Primitive Return Types** must be **identical** in the overriding method.
- No covariance is allowed for primitives (e.g., an override cannot return `int` if the parent returns `long`).

### 3.2 Common Pitfalls

- **Primitive vs. Wrapper**: You cannot return a wrapper (e.g., `Long`) if the parent returns a primitive (e.g., `long`), as these are distinct types despite autoboxing.
- **Precision Changes**: Even if a `short` fits into a `long`, the compiler requires the exact primitive type declared in the parent.

```java
class PrimitiveBase {
    public long getCount() { return 1L; }
}

class PrimitiveChild extends PrimitiveBase {
    // COMPILATION ERROR: Return type must be 'long'
    // @Override
    // public int getCount() { return 5; } 

    @Override
    public long getCount() { return 10L; } // Correct: Identical primitive type
}
```

[Back to Top](#table-of-contents)

---

## 4. Verification and Best Practices

### 4.1 The @Override Safety Net

- Always use the `@Override` annotation when implementing covariant returns.
- It ensures the compiler verifies the signature and return type compatibility.
- If you accidentally change the signature (e.g., by changing an argument type), the `@Override` annotation will trigger a compilation error, preventing a subtle "overloading" bug.

[Back to Top](#table-of-contents)
