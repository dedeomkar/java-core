# Defining Records

## Table of Contents
- [1. Record Fundamentals](#1-record-fundamentals)
  - [1.1 Purpose as Data Carriers](#11-purpose-as-data-carriers)
  - [1.2 Syntax and Structure](#12-syntax-and-structure)
- [2. Architectural Constraints](#2-architectural-constraints)
  - [2.1 Inheritance Limitations](#21-inheritance-limitations)
  - [2.2 Field Constraints](#22-field-constraints)
- [3. Auto-Generated Functionality](#3-auto-generated-functionality)
  - [3.1 Native Accessors](#31-native-accessors)
  - [3.2 Standard Object Methods](#32-standard-object-methods)
- [4. The Immutability Caveat](#4-the-immutability-caveat)

---

## 1. Record Fundamentals

### 1.1 Purpose as Data Carriers
- A **Record** provides a concise syntax for defining aggregate data types or "data carriers" in Java.
- They group related values together purely for data transport or static state management, heavily minimizing standard boilerplate code.
- Records are architecturally designed to be unmodifiable, which means they inherently lack all setter methods.

### 1.2 Syntax and Structure
- Creating a record utilizes the reserved `record` keyword.
- The elements intended for storage are declared explicitly inside strict parentheses `()`, and are formally referred to as **components**.
- Mandatory curly braces `{}` must always follow the component list, even if completely empty.

```java
// Formal declaration of a record
public record Customer(String name, int creditLimit) { }
```

---

## 2. Architectural Constraints

### 2.1 Inheritance Limitations
- **Implicitly Final**: Record types are implicitly `final`. You cannot subclass a record. (Appending `final` explicitly is permitted but functionally redundant).
- **Implicit Parent**: All records automatically structurally extend `java.lang.Record`. Because Java prohibits multiple inheritance of classes, you **cannot use an `extends` clause**.
- **Interfaces**: Records uniquely are fully permitted to natively implement one or more interfaces.

### 2.2 Field Constraints
- You are **not permitted to explicitly declare custom instance fields**. Only the components named directly in the header parentheses exist as instance state.
- Component names must avoid colliding with standard `java.lang.Object` parameterless methods (e.g., defining a component named `wait` triggers an instant compilation failure).
- **Member Records**: Inherently, if a record is declared nested inside another parent class, it is automatically marked `static` to strictly prevent unintended references back to the enclosing parent class's instance state.

---

## 3. Auto-Generated Functionality

### 3.1 Native Accessors
- Records automatically generate native accessor methods for every predefined component.
- Deviating from traditional JavaBeans styling (e.g., `getName()`), these generated accessors are named **exactly identical to the component itself** (e.g., `name()`).
- These native methods are universally `public`, take precisely zero arguments, declare zero exceptions, and return the component's exact explicit type securely.

```java
Customer c1 = new Customer("Fred", 1000);

// Extract components invoking the exact component name cleanly:
String fetchedName = c1.name(); 
```

### 3.2 Standard Object Methods
The compiler natively synthesizes the boilerplate code servicing standard object behaviors:
- `equals()` and `hashCode()` are logically mapped safely evaluating identically against all internal components collectively.
- A highly functional `toString()` is generated formatting visually the record's specific type alongside the contents of each initialized underlying component.

---

## 4. The Immutability Caveat

> **Important Consideration**: Records automatically synthesize securely `private final` mapping fields anchoring their components natively, but physically routing a strictly explicitly mutable object (like a `StringBuilder` or standard `ArrayList`) into a record creates an architectural vulnerability. **Java fundamentally does not natively possess deep immutability mechanics**. 

Functionally, if a record accurately stores a literal reference pointer mapping to a mutable object natively, external operational code retaining that exact identical exterior pointer can independently manipulate the deep internal state of that object, entirely bypassing the logical "unmodifiable" premise of the parent record.

```java
// A formal record structurally wrapping a fundamentally explicitly mutable type
record Named(StringBuilder nameBuilder) { }

StringBuilder sb = new StringBuilder("Initial");
Named n1 = new Named(sb);

// Mutating the externally held object explicitly alters the record's payload secretly!
sb.append("... Compromised");
System.out.println(n1.nameBuilder()); // Output reliably exposes: Initial... Compromised
```

---

[Back to Top](#table-of-contents)
