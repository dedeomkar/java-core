# Interfaces and Functional Interfaces

## Table of Contents
- [1. Interface Method Types](#1-interface-method-types)
  - [1.1 Abstract Methods](#11-abstract-methods)
  - [1.2 Static Methods](#12-static-methods)
  - [1.3 Private Methods](#13-private-methods)
  - [1.4 Default Methods](#14-default-methods)
- [2. Interface Rules and Constraints](#2-interface-rules-and-constraints)
  - [2.1 Explicit Receiver Parameters](#21-explicit-receiver-parameters)
  - [2.2 Accessibility and Modifiers](#22-accessibility-and-modifiers)
  - [2.3 Fields in Interfaces](#23-fields-in-interfaces)
- [3. Inheritance and Redeclaration](#3-inheritance-and-redeclaration)
  - [3.1 Extending Interfaces](#31-extending-interfaces)
  - [3.2 Redeclaring Methods](#32-redeclaring-methods)
- [4. Functional Interfaces](#4-functional-interfaces)
  - [4.1 Single Abstract Method (SAM) Rule](#41-single-abstract-method-sam-rule)
  - [4.2 Object Class Exclusions](#42-object-class-exclusions)
  - [4.3 The @FunctionalInterface Annotation](#43-the-functionalinterface-annotation)

---

## 1. Interface Method Types

Java interfaces have evolved significantly, allowing various types of method declarations beyond simple abstract signatures.

### 1.1 Abstract Methods

- **Definition**: Methods without a body, ending in a semicolon.
- **Defaults**: Implicitly `public` and `abstract`.
- **Constraint**: Cannot have a body.

> [!NOTE]
> Think of an abstract method as a **Job Description**. It lists what needs to be done but doesn't provide the person (implementation) to do it.

```java
public interface Task {
    void execute(); // Implicitly public abstract
}
```

### 1.2 Static Methods

- **Usage**: Associated with the interface itself, not an instance.
- **Accessibility**: Can be `public` or `private`.
- **Invocation**: Must be called using the interface name (e.g., `MyInterface.myStaticMethod()`). Unlike class statics, they cannot be called via an instance reference.

### 1.3 Private Methods

- **Purpose**: Encapsulate shared logic between default or static methods within the same interface.
- **Types**: Can be instance or static.
- **Constraint**: Must have a body.

### 1.4 Default Methods

- **Definition**: Public instance methods that provide a "fallback" implementation.
- **Constraint**: Must have a body and are implicitly `public`.
- **Inheritance**: Subclasses inherit the implementation unless they override it.

> [!WARNING]
> Notice there are no instance methods **eg. void test() {}**. We can either make `test()` -> `abstract` , `static` , `private` or `default` to resolve the compilation error.

> [!TIP]
> Use default methods to add new functionality to existing interfaces without breaking existing implementations. It's like adding a **Standard Operating Procedure** (SOP) that everyone can follow unless they have a specialized way of doing things.


---

## 2. Interface Rules and Constraints

### 2.1 Explicit Receiver Parameters

- In default or private instance methods, the `this` parameter (receiver) can be declared explicitly.
- **Syntax**: The type must be the interface type, and the name must be `this`.

```java
interface A {
    default void doWork(A this) { 
        // 'this' is explicitly typed as A
    }
}
```

### 2.2 Accessibility and Modifiers

- **Public by Default**: All members (methods and fields) are `public` by default.
- **No Protected/Package-Private**: Interfaces do not allow `protected` or package-access (`default` in the access modifier sense).
- **Final Constraint**: Methods in an interface cannot be `final` (except private methods which are effectively final).

### 2.3 Fields in Interfaces

- **Implicit Modifiers**: Fields are always `public static final`.
- **Initialization**: Must be initialized at the time of declaration.
- **Mutability**: While the reference is `final`, it can point to a mutable object (e.g., `StringBuilder`).

```java
interface Constants {
    int MAX_RETRY = 5; // Implicitly public static final
    StringBuilder LOG_BUFFER = new StringBuilder(); // Legal, even if mutable
}
```


---

## 3. Inheritance and Redeclaration

### 3.1 Extending Interfaces

- An interface can extend multiple other interfaces using the `extends` keyword.
- It inherits all abstract and default methods. It does **not** inherit private or static methods.

### 3.2 Redeclaring Methods

- A child interface can redeclare a parent's method.
- **Abstract to Default**: Provide a body for a previously abstract method.
- **Default to Abstract**: Remove the body, forcing subclasses to provide an implementation.


---

## 4. Functional Interfaces

### 4.1 Single Abstract Method (SAM) Rule

- A functional interface is an interface that declares **exactly one** abstract method.
- It can have any number of `static` or `default` methods.

### 4.2 Object Class Exclusions

- Methods that are exact overloads of `java.lang.Object` methods (like `boolean equals(Object obj)`) do **not** count toward the SAM limit.

```java
@FunctionalInterface
interface MyFunction {
    void apply();
    boolean equals(Object obj); // Does not count as an abstract method here
}
```

### 4.3 The @FunctionalInterface Annotation

- **Purpose**: Documents the intent and asks the compiler to verify that the interface meets the functional interface requirements.
- **Lambda Compatibility**: Any interface meeting the SAM rule can be implemented by a lambda expression, regardless of whether the annotation is present.

