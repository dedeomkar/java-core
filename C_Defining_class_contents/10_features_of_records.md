# Features of Records

## Table of Contents
- [1. Customizing Record Methods](#1-customizing-record-methods)
  - [1.1 Method Replacement vs Overriding](#11-method-replacement-vs-overriding)
  - [1.2 Custom Methods and Fields](#12-custom-methods-and-fields)
- [2. The Canonical Constructor](#2-the-canonical-constructor)
  - [2.1 Default Implementation](#21-default-implementation)
  - [2.2 Hand-Coded Replacements](#22-hand-coded-replacements)
- [3. The Compact Constructor](#3-the-compact-constructor)
  - [3.1 Structure and Purpose](#31-structure-and-purpose)
  - [3.2 The Field Constraint](#32-the-field-constraint)
- [4. Constructor Overloading and Chaining](#4-constructor-overloading-and-chaining)
  - [4.1 Delegation Rules](#41-delegation-rules)
- [5. Deserialization Behavior](#5-deserialization-behavior)

---

## 1. Customizing Record Methods

### 1.1 Method Replacement vs Overriding
- The compiler-provided automatic methods (`equals`, `hashCode`, `toString`, and the accessors) can be manually replaced to customize logic.
- They accept the `@Override` annotation, but this is a **replacement**, not a true traditional override. 
> **Important Consideration**: Because you are entirely replacing the method inside the record, there is no underlying parent implementation. Calling `super.toString()` or `super.name()` inside a record replacement perfectly fails compilation.

### 1.2 Custom Methods and Fields
- **Methods**: You can freely add brand new custom static or instance methods to a record layout. (Abstract and native methods are forbidden).
- **Static Fields**: You can define static fields and static initializers effortlessly (and they do not need to be `final`).
- **Instance Fields**: As previously covered, any custom instance fields or instance initialization blocks are fundamentally strictly forbidden.

---

## 2. The Canonical Constructor

### 2.1 Default Implementation
Records implicitly define a **Canonical Constructor** primarily assigned with securely mapping the input arguments cleanly into the `private final` mapping fields inherently matching the components. 
- By default, it generates automatically utilizing an argument sequence perfectly identically matching the exact mapping explicitly mirroring the record's components.

### 2.2 Hand-Coded Replacements
Developers often explicitly define the canonical constructor manually for **validation** (e.g., rejecting negative ranges) or **normalization** (e.g., formatting raw string inputs).
- **Rigid Parameter Mapping**: If you completely manually type the literal canonical constructor, the formal parameter names **must mathematically identically match** the exact explicit component identifiers.
- **Accessibility**: It cannot be constructed less accessible natively than the overarching record configuration.
- **Initialization**: As it represents the core master constructor, you must physically map validating initializing the target fields.

---

## 3. The Compact Constructor

### 3.1 Structure and Purpose
Because manually writing out the canonical constructor includes redundant mapping code targeting fields, Java enables a specific short-hand notation known as the **Compact Constructor**. 
- It operates uniquely lacking a formal bracketed parameter list natively `()`. 
- Functionally, any logic written here dynamically acts as a **preamble** securely preceding seamlessly feeding the entirely automatically-generated standard canonical constructor.

```java
public record Range(int low, int high) {
    // Compact constructor syntax (No parameter brackets!)
    public Range {
        // Perfect for validation logic before field mapping:
        if (low > high) {
            throw new IllegalArgumentException("Invalid layout");
        }
        // Logic falls through seamlessly completing automatic field assignments
    }
}
```

### 3.2 The Field Constraint
> **Important Consideration**: A compact constructor **must entirely avoid writing directly to the underlying `this.` fields**. If you assign them here, the seamlessly proceeding fully automatic canonical constructor will illegally attempt explicitly initializing those identically targeted `final` fields twice natively generating an error. 

You are inherently forbidden structurally designating both an explicit manual canonical constructor and uniquely formulating a compact constructor simultaneously.

---

## 4. Constructor Overloading and Chaining

### 4.1 Delegation Rules
You can implement entirely separate overloaded constructors to define fallback default arguments.
- Any overloaded extra constructor natively uniquely **must securely successfully delegate (via the `this(...)` execution explicitly securely identically chaining down)** terminating routing functionally arriving securely natively at the primary Canonical Constructor.
- Circular looping references entirely avoiding tracing natively down targeting exactly the Canonical Constructor explicitly triggers compilation errors natively securely identically.

---

## 5. Deserialization Behavior
In standard traditional Java classes, deserialization logic explicitly natively securely successfully structurally completely bypasses calling any functional constructors natively reconstructing the class identically.
- **For Records**: Deserializing a serializable Record natively diverges, actively systematically **invoking utilizing natively the active Canonical Constructor explicitly**. This prevents malicious payloads securely flawlessly natively validating cleanly thoroughly cleanly formatting inputs inherently.

---

[Back to Top](#table-of-contents)
