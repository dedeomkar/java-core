# Overloaded and Overridden Methods, Part 2

## Table of Contents
- [1. Principles of Method Overriding](#1-principles-of-method-overriding)
  - [1.1 The Definition of Overriding](#11-the-definition-of-overriding)
  - [1.2 Method Hiding](#12-method-hiding)
- [2. Liskov Substitution Principle Enforcements](#2-liskov-substitution-principle-enforcements)
  - [2.1 Exception Compatibility](#21-exception-compatibility)
  - [2.2 Accessibility Upgrades](#22-accessibility-upgrades)
  - [2.3 Return Type Compliance](#23-return-type-compliance)
- [3. Dynamic Binding Mechanics](#3-dynamic-binding-mechanics)
- [4. Edge Cases: Equals and Generics](#4-edge-cases-equals-and-generics)
  - [4.1 The Equals Trap](#41-the-equals-trap)
  - [4.2 Generic Bridge Methods](#42-generic-bridge-methods)

---

## 1. Principles of Method Overriding

### 1.1 The Definition of Overriding
An **override** systematically transpires when a child class implements a specific instance method conveying the exact identical label name paired identically configuring an identical argument type sequence natively matching a visible instance method established firmly inside a parent class.
- **Failures**: Formatting identical identical parameters locally across identically identical single classes simply manifests configuring invalid overload architectures, permanently failing compilation.
- **Finality**: Parent methods intentionally marked utilizing `final` rigidly formally prohibit creating any native overriding capability.

### 1.2 Method Hiding
If specifically both the parent method alongside the identically-formatted child method instantiate executing utilizing exclusively `static` layouts, the structure completely yields **Method Hiding**, sidestepping establishing functionally true overriding. 

---

## 2. Liskov Substitution Principle Enforcements

Because a child class instantiated object inherently must safely structurally execute flawlessly substituting replicating an overarching parent object reference, Java rigorously enforces stability requirements:

### 2.1 Exception Compatibility
- The active overriding child method rigorously must completely not inject natively **distinct novel checked exceptions**, nor universally throw significantly broader checked exceptions beyond configurations formally declared matching the parent layout.
- Natively overrides structurally remain totally open unconditionally throwing any spectrum formatting **unchecked exceptions** (e.g., `RuntimeException` offspring), discarding explicitly matching previous parent designs.

### 2.2 Accessibility Upgrades
- The child implementation method structurally must continually configure **at least equally accessible** (or natively distinctly more unrestricted publicly) matching parallel the parent method.
- **Example**: Overriding modifying universally a `protected` method establishing converting formatting into a `public` layout safely executes smoothly; conversely, structurally constricting that identically mirroring method enforcing `private` definitively universally fails compilation processing.

### 2.3 Return Type Compliance
- **Primitive Return Types**: Modifying overrides processing commanding a configured primitive type definitively unconditionally demand uniformly matching producing strictly returning the exact matching parallel primitive type. 
- **Reference Return Types**: Manipulating object formats technically universally demand rigid **assignment compatibility**. If a parent format explicitly yields `CharSequence`, the corresponding parallel child override safely technically functions actively yielding `String` internally (because a `String` layout structurally functions acting inherently matching as a `CharSequence`).

```java
class Parent {
    public CharSequence doStuff() { return "Parent"; }
}

class Child extends Parent {
    // Valid safe override formatting: narrower, rigorously assignment-compatible reference type
    @Override
    public String doStuff() { return "Child"; }
}
```

---

## 3. Dynamic Binding Mechanics
- Instantiated identical tracking instance methods universally unconditionally computationally trigger activating utilizing **dynamic binding** (frequently informally labeled computationally virtual binding / late execution binding).
- When a dynamically overridden function commands execution traversing processing paths, the fundamental underlying runtime dictates natively fulfilling executing the behaviors physically constructed executing inside the **actual concrete instantiated object allocated dynamically targeting the active memory**. The syntactic typing specifically structuring referencing configuring explicit variables strictly universally controls mapping activating essentially nothing.
> **Vital Reminder**: This architectural flexibility selectively identically strictly regulates addressing instance methods fundamentally exclusively; static method configurations manipulating method hiding behaviors natively mirroring paired class field tracking universally execute dynamically relying inherently upon reference pointer static formats.

---

## 4. Edge Cases: Equals and Generics

### 4.1 The Equals Trap
A significantly prevalent architectural trap involves engineers unknowingly generating basic overloads erroneously assuming creating formatting true overriding activating utilizing identical `.equals()` mechanics universally.

```java
public class MyEntity {
    // CRITICALLY THIS STRUCTURALLY MAPS NOT OVERRIDING! Function strictly overloads globally `equals(Object)`
    // Collections APIs permanently actively perfectly ignore executing this natively custom format consistently!
    public boolean equals(MyEntity other) { return true; }
}
```

### 4.2 Generic Bridge Methods
Configuring generics tangles formatting establishing overriding parameter typings. Coding universally mapped isolated interfaces mimicking generically like `Comparable<Thing>` mathematically forces the compilation engine natively building establishing an unseen **bridge method**.
- The compiler architecture natively silently maps producing an executable facade `compareTo(Object)` interface method rigorously conforming executing formally overriding the master native object signature layout. This mechanically structurally relays passing routing casting the target variable natively navigating seamlessly internally targeting the developer's explicitly implemented coded `compareTo(Thing)`.
- The architecture correctly translates natively mapping mimicking natively processing functional executing essentially equivalent producing genuine native overriding. 

---

[Back to Top](#table-of-contents)
