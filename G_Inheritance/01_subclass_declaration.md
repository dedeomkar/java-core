# Subclass Declaration

## Table of Contents
- [1. Core Syntax and Requirements](#1-core-syntax-and-requirements)
  - [1.1 The Extends Clause](#11-the-extends-clause)
  - [1.2 Implicit Inheritance](#12-implicit-inheritance)
- [2. Subclassing Constraints](#2-subclassing-constraints)
  - [2.1 Final Classes](#21-final-classes)
  - [2.2 Constructor Accessibility](#22-constructor-accessibility)
- [3. Instance Method Overriding](#3-instance-method-overriding)
  - [3.1 Overriding Rules](#31-overriding-rules)
  - [3.2 The @Override Annotation](#32-the-override-annotation)
- [4. Static Method Hiding](#4-static-method-hiding)
  - [4.1 Hiding vs Overriding](#41-hiding-vs-overriding)
- [5. Advanced Considerations](#5-advanced-considerations)
  - [5.1 Type Erasure and Generics](#51-type-erasure-and-generics)
  - [5.2 Arrays vs Varargs](#52-arrays-vs-varargs)
  - [5.3 Dynamic vs Static Invocation](#53-dynamic-vs-static-invocation)

---

## 1. Core Syntax and Requirements

### 1.1 The Extends Clause

- Subclassing is initiated using the `extends` keyword.
- **Single Inheritance**: Java permits only one immediate parent class.
- **Ancestry Chain**: While only one immediate parent is allowed, a class can have multiple ancestors (Parent, Grandparent, etc.).
- **Implements Clause**: If used, it must follow the `extends` clause.

> [!NOTE]
> Think of a class hierarchy like a skyscraper built on a single foundation. You can add floors (subclasses), but you can't build one floor on two different buildings simultaneously.

```java
// Correct order: extends then implements
public class SmartPhone extends ElectronicDevice implements Callable, Portable {
    // ...
}
```

### 1.2 Implicit Inheritance

- If no `extends` clause is provided, the class implicitly extends `java.lang.Object`.
- This ensures every class in Java shares a common set of baseline behaviors (e.g., `toString()`, `equals()`).

[Back to Top](#table-of-contents)

---

## 2. Subclassing Constraints

### 2.1 Final Classes

- A class marked `final` cannot be subclassed.
- This is often used for security or to ensure immutability (e.g., `java.lang.String`).

```java
public final class SecureToken { }

// COMPILATION ERROR: Cannot inherit from final 'SecureToken'
// public class HackToken extends SecureToken { } 
```

### 2.2 Constructor Accessibility

- A subclass must initialize its parent.
- If a parent has only `private` constructors, it can only be subclassed by its own nested or inner classes.

> [!IMPORTANT]
> The subclass must have access to at least one constructor in the parent class to successfully initialize the "foundation" of the object.

[Back to Top](#table-of-contents)

---

## 3. Instance Method Overriding

### 3.1 Overriding Rules

Overriding occurs when a subclass provides a specific implementation for a method declared in its parent.

- **Signature**: Must have the identical name and argument type sequence.
- **Accessibility**: Cannot be less accessible than the parent (e.g., `protected` cannot become `package-private`).
- **Return Type**:
    - **Reference Types**: Must be **assignment compatible** (Covariant return types).
    - **Primitive Types**: Must be **identical**.
- **Exceptions**: Any checked exceptions must be the same or subclasses of those declared in the parent.
- **Final Check**: The parent method must not be marked `final`.

```java
class Parent {
    protected Number getNumber() throws IOException { return 10; }
}

class Child extends Parent {
    @Override
    public Integer getNumber() { return 20; } // Valid: Integer is a subclass of Number
}
```

### 3.2 The @Override Annotation

- Used to verify that a method is actually overriding a parent method.
- **Verification only**: It does not *cause* overriding; it simply asks the compiler to double-check your work.

[Back to Top](#table-of-contents)

---

## 4. Static Method Hiding

### 4.1 Hiding vs Overriding

When a subclass defines a static method with the same signature as a static method in the parent, it is called **hiding**, not overriding.

- **Rules**: Most overriding rules apply (accessibility, return types, exceptions, `final` check).
- **Annotation**: The `@Override` annotation **cannot** be used on static methods.

```java
class Utils {
    public static void print() { System.out.println("Base Utils"); }
}

class CustomUtils extends Utils {
    // This HIDES Utils.print()
    public static void print() { System.out.println("Custom Utils"); }
}
```

[Back to Top](#table-of-contents)

---

## 5. Advanced Considerations

### 5.1 Type Erasure and Generics

- Due to **Type Erasure**, generic types like `List<String>` and `List<Integer>` are seen as the same type (`List`) at runtime.
- **Overloading**: Fails if the only difference is the generic type of an argument.
- **Overriding**: Rejected if one uses `List<String>` and the other `List<LocalDate>`.

```java
// This will NOT compile as an overload
public void process(List<String> list) {}
// public void process(List<Integer> list) {} // Error: same erasure
```

### 5.2 Arrays vs Varargs

- For the purposes of overriding, arrays and variable-length arguments (`...`) are treated as equivalent.
- A method taking `String[]` can be overridden by a method taking `String...`.

### 5.3 Dynamic vs Static Invocation

- **Instance Methods (Dynamic)**: The behavior at runtime is determined by the **actual object type** (Polymorphism).
- **Static Methods (Static)**: The behavior is determined at **compile-time** based on the reference type or explicit class name.

> [!WARNING]
> Accessing static methods via object references (e.g., `myObj.staticMethod()`) is legal but highly discouraged as it masks the fact that the call depends on the reference type, not the object type.

[Back to Top](#table-of-contents)
