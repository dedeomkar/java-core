# Instance and static fields, part 2

## Table of Contents
- [1. Resolving Identifiers](#1-resolving-identifiers)
  - [1.1 Unqualified Identifiers](#11-unqualified-identifiers)
  - [1.2 Shadowing and Disambiguation](#12-shadowing-and-disambiguation)
- [2. Understanding Static vs Instance](#2-understanding-static-vs-instance)
  - [2.1 The Nature of Static](#21-the-nature-of-static)
  - [2.2 Static Fields Accessed via Instances](#22-static-fields-accessed-via-instances)
- [3. Rule Characteristics for Fields](#3-rule-characteristics-for-fields)
  - [3.1 Limitations on `var`](#31-limitations-on-var)
  - [3.2 No Dynamic Binding for Fields](#32-no-dynamic-binding-for-fields)
  - [3.3 Interface Fields](#33-interface-fields)
- [4. Advanced Field Access Examples](#4-advanced-field-access-examples)
  - [4.1 Evaluating Shadowed Fields](#41-evaluating-shadowed-fields)
  - [4.2 Prefixing Static Class Members](#42-prefixing-static-class-members)

---

## 1. Resolving Identifiers

### 1.1 Unqualified Identifiers
When an identifier lacks a prefix (e.g., `theField` instead of `myObj.theField`), Java resolves it using a precise hierarchy:
1.  **Method Locals**: The compiler first searches for a local variable matching the name.
2.  **Instance Context (implicit `this`)**: If not found locally and inside an instance context, it searches for an instance field via `this.theField`.
3.  **Static Context (implicit `Class`)**: Next, it searches for a shared static class field.

### 1.2 Shadowing and Disambiguation
- A local method variable will always take precedence over and "shadow" a class field.
- If multiple possibilities exist across different levels of the class hierarchy, the "closest" match wins.
- To disambiguate and explicitly target a shadowed field:
  - Provide a class-name prefix for static fields.
  - Provide the `this` prefix for current instance fields.
  - Provide the `super` prefix in an instance context to access a parent's shadowed instance field (Note: chaining super prefixes like `super.super` is strictly illegal).

---

## 2. Understanding Static vs Instance

### 2.1 The Nature of Static
- Static features are globally associated with the generalized type or class, acting universally. There is only one discrete shared variable among all instances of that type.
- **Poor Style Convention**: Due to early syntax design choices, accessing a static member using an instance reference is technically legal (e.g., `s1.staticVar`). This obscures intent and behaves identically as if invoked statically, causing confusion. 
> **Important Consideration**: Due to problems this caused historically, attempting this same instance-based invocation syntax with interface static methods was rightly prohibited in newer Java versions.

### 2.2 Static Fields Accessed via Instances
When multiple objects of a class modify a static field, they invariably manipulate the same singular memory location.
```java
// Assuming StaticClass initially has x = 99
StaticClass s1 = new StaticClass();
StaticClass s2 = new StaticClass();

s1.x++; // Legitimate syntax, but ugly. Increments static 'x' to 100
s2.x++; // Increments the globally shared static 'x' to 101

System.out.println(s1.x); // Prints 101
```

---

## 3. Rule Characteristics for Fields

### 3.1 Limitations on `var`
- The `var` keyword is completely illegal for field creation. It is fundamentally restricted to localized variable declarations within methods.

### 3.2 No Dynamic Binding for Fields
- In stark contrast to instance methods, class fields **do not exhibit dynamic virtual binding**.
- A field's interaction is firmly established by the **compile-time type of the reference variable**, ignoring whatever concrete object the reference may hold at runtime.

### 3.3 Interface Fields
- Any field specified inside an interface implicitly obtains `public static final` modifiers.
- While they act like constants, they don't uniformly demand literal constant declarations; they can store references to initialized mutable objects.
- Attempting to access an interface field making use of the `super` prefix is illegal unless standard inheritance weaves that exact interface's field directly into an overarching Parent class.

---

## 4. Advanced Field Access Examples

### 4.1 Evaluating Shadowed Fields
```java
class Parent { 
    int x = 100; 
}
class Child extends Parent { 
    int x = 200; 
}

Child c = new Child();
Parent p = c; // 'p' references the Child object strictly via a Parent lens

System.out.println(c.x); // Prints 200 (Resolves against 'Child' type)
System.out.println(p.x); // Prints 100 (Resolves against 'Parent' type)
```

### 4.2 Prefixing Static Class Members
If `J.value` represents a static integer internally located in class `J`, it can effectively be retrieved within an instance scope equivalently via:
- Standard class prefixing: `J.value`
- Subclass parent prefixing: `super.value` 
- Explicit scope casting: `((J)this).value`

---

[Back to Top](#table-of-contents)
