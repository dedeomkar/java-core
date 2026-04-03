# 02. Access Control Modifiers in Java

## Table of Contents
- [1. Accessibility Layers](#1-accessibility-layers)
  - [1.1 Four Access Modifiers](#11-four-access-modifiers)
  - [1.2 Modern JVM & Modules](#12-modern-jvm--modules)
- [2. Private and Default](#2-private-and-default)
  - [2.1 Private: The Top-Level Rule](#21-private-the-top-level-rule)
  - [2.2 Default (Package-Private)](#22-default-package-private)
- [3. Protected and Public](#3-protected-and-public)
  - [3.1 Protected: The Subclass Nuance](#31-protected-the-subclass-nuance)
  - [3.2 Public: Global vs. Modular](#32-public-global-vs-modular)
- [4. Inheritance and Cross-Package Access](#4-inheritance-and-cross-package-access)
  - [4.1 Case Study: Protected Access Failure](#41-case-study-protected-access-failure)

---

## 1. Accessibility Layers

### 1.1 Four Access Modifiers
Java provides four levels of visibility for fields, methods, and types:
1. **Private**: Strictly internal.
2. **Default (no keyword)**: Package-internal.
3. **Protected**: Package-internal + Subclasses.
4. **Public**: Universally accessible (with module caveats).

### 1.2 Modern JVM & Modules
The introduction of the Java Module System added a layer of complexity to these levels:
- **Modular Public**: Only accessible outside the module if the package is explicitly **exported**.
- **Split Packages Prohibited**: You cannot define the same package name across two different modules.

---

## 2. Private and Default

### 2.1 Private: The Top-Level Rule
- **Definition**: Accessible anywhere within the **enclosing top-level curly braces**.
- **The "Myth"**: It is often stated that `private` is limited to the class. 
- **The Reality**: If you have nested classes, the outer class can access the inner class's private members, and vice-versa.

### 2.2 Default (Package-Private)
- **Definition**: Accessible only within the same package.
- **Context-Specific Behavior**:
  - **Interfaces**: Elements without a modifier are automatically `public`.
  - **Enum Constructors**: Default to `private`.

---

## 3. Protected and Public

### 3.1 Protected: The Subclass Nuance
- **Internal**: Accessible to anything in the same package (identical to default).
- **External**: Accessible to subclasses in **other packages**, but with a strict condition:
> Access from a subclass in a different package MUST be performed using a reference variable of the **subclass type**, not the parent type.

### 3.2 Public: Global vs. Modular
- **Traditional**: Accessible anywhere in the JVM.
- **Modular**: Only accessible to other modules if:
  1. The module **exports** the package.
  2. The other module **reads** the declaring module.

---

## 4. Inheritance and Cross-Package Access

### 4.1 Case Study: Protected Access Failure
Consider Class `P` (Package A) has a `protected` field `x`. Class `Q` (Package B) extends `P`.

```java
// Package B
public class Q extends P {
    void test() {
        Q q = new Q();
        System.out.println(q.x); // VALID: Access via Subclass reference

        P p = new P();
        // System.out.println(p.x); // COMPILER ERROR: Cannot access via Parent reference cross-package
    }
}
```

- **Reasoning**: If `protected` allowed access via a Parent reference (`P p`), anyone could bypass encapsulation simply by creating a new instance of the parent class.


---
