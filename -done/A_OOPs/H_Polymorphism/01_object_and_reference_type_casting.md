# Object and Reference Type Casting

## Table of Contents
- [1. Understanding Reference vs. Object Types](#1-understanding-reference-vs-object-types)
  - [1.1 The "Is-A" Relationship](#11-the-is-a-relationship)
  - [1.2 Compiler vs. Runtime Perspectives](#12-compiler-vs-runtime-perspectives)
- [2. Reference Casting](#2-reference-casting)
  - [2.1 Casting Syntax](#21-casting-syntax)
  - [2.2 Mechanical Effect of a Cast](#22-mechanical-effect-of-a-cast)
- [3. Safely Testing Type Compatibility](#3-safely-testing-type-compatibility)
  - [3.1 The instanceof Operator](#31-the-instanceof-operator)
  - [3.2 Common Pitfalls and Null Rules](#32-common-pitfalls-and-null-rules)
- [4. Operator Precedence in Casting](#4-operator-precedence-in-casting)
  - [4.1 Casting vs. Member Selection](#41-casting-vs-member-selection)
  - [4.2 Casting vs. Binary Operators](#42-casting-vs-binary-operators)

---

## 1. Understanding Reference vs. Object Types

### 1.1 The "Is-A" Relationship

- **Assignment Rule**: A reference of type `P` can point to an object if that object is a `P`.
- **Satisfaction Criteria**:
    - The object is an instance of class `P`.
    - The object is an instance of a **subclass** of `P`.
    - The object belongs to a class that **implements** interface `P`.

> [!NOTE]
> Think of a reference as a specialized "remote control." A "Universal Remote" (Parent reference) can control a "Sony TV" (Child object) because the Sony TV "is a" TV. However, the universal remote only has buttons for basic functions common to all TVs.

### 1.2 Compiler vs. Runtime Perspectives

- **Compiler Constraint**: The compiler only permits access to methods and fields defined in the **Reference Type** (the "buttons" on the remote).
- **Runtime Reality**: The actual behavior (implementation) is determined by the **Object Type**.
- **Specialized Features**: To access features unique to the object's actual class (not present in the reference type), a cast is required.

```java
class Parent { void doWork() {} }
class Child extends Parent { void play() {} }

Parent p = new Child();
p.doWork(); // OK: defined in Parent
// p.play(); // COMPILER ERROR: Parent doesn't know about 'play'
```


---

## 2. Reference Casting

### 2.1 Casting Syntax

- **Format**: `(TargetType) expression`
- **Purpose**: Explicitly tells the compiler to treat a reference as a more specific (or different) type.

```java
Object obj = "Hello";
String s = (String) obj; // Successful cast
```

### 2.2 Mechanical Effect of a Cast

- **Value vs. Type**: A cast does **not** modify the underlying object. It creates a new expression with the same reference value (memory address) but a different compile-time type.
- **Identity Preservation**: The pointer (address) remains the same; only the "lens" through which the compiler views it changes.

> [!IMPORTANT]
> A cast is like putting on "3D glasses" to see hidden depth in a movie. The movie (the object) hasn't changed, but your ability to perceive specific elements (methods/fields) has.


---

## 3. Safely Testing Type Compatibility

### 3.1 The instanceof Operator

- **Function**: Tests at runtime whether an object is compatible with a specific type.
- **Syntax**: `reference instanceof ClassName`
- **Result**: Returns a `boolean`.

```java
if (p instanceof Child) {
    Child c = (Child) p;
    c.play();
}
```

### 3.2 Common Pitfalls and Null Rules

- **Null Interaction**: `null instanceof AnyType` is ALWAYS `false`. It never throws a `NullPointerException`.
- **Plausibility Check**: The compiler rejects `instanceof` tests that are logically impossible (e.g., testing if an `Integer` is a `String`).


---

## 4. Operator Precedence in Casting

### 4.1 Casting vs. Member Selection

- **Precedence**: Member selection (`.`) and array subscripting (`[]`) have **higher** precedence than casting.
- **Common Error**: `(String) obj.length()` is interpreted as `(String) (obj.length())`.

```java
Object obj = "Hello";
// int len = (String) obj.length(); // ERROR: length() not in Object

int len = ((String) obj).length(); // CORRECT: Parentheses force the cast first
```

### 4.2 Casting vs. Binary Operators

- **Precedence**: Casting has **higher** precedence than most binary operators (like `+`).
- **Behavior**: In `(String) cs + 3`, the cast happens first, then the string concatenation.

> [!WARNING]
> Always use extra parentheses when combining casts with method calls to ensure the cast applies to the reference, not the result of the method call.

