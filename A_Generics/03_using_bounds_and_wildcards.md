# Using Bounds and Wildcards

## Table of Contents
- [1. Bounded Type Parameters](#1-bounded-type-parameters)
  - [1.1 The Extends Bound](#11-the-extends-bound)
  - [1.2 Multiple Bounds](#12-multiple-bounds)
- [2. Wildcards in Generics](#2-wildcards-in-generics)
  - [2.1 Wildcards for Beginners (The "Fruit Crate" Analogy)](#21-wildcards-for-beginners-the-fruit-crate-analogy)
  - [2.2 Upper Bounded Wildcards (? extends T)](#22-upper-bounded-wildcards--extends-t)
  - [2.3 Lower Bounded Wildcards (? super T)](#23-lower-bounded-wildcards--super-t)
  - [2.4 Unbounded Wildcards (?)](#24-unbounded-wildcards-)
- [3. Declaration Constraints](#3-declaration-constraints)
  - [3.1 Valid vs Invalid Declarations](#31-valid-vs-invalid-declarations)
  - [3.2 Static Method Requirements](#32-static-method-requirements)
- [4. Advanced Functional Patterns](#4-advanced-functional-patterns)
  - [4.1 Improving Generic Interfaces](#41-improving-generic-interfaces)

---

## 1. Bounded Type Parameters

### 1.1 The Extends Bound

- **Constraint Mechanism**: Restricts a type parameter to only those types that provide specific features or behaviors.
- **Assignment Compatibility**: In generics, `extends` means **"is assignment compatible to."** It covers three distinct scenarios:
  1. **Identity**: `E` is exactly the same as the bound.
  2. **Inheritance**: `E` is a subclass of the bound (if the bound is a class).
  3. **Implementation**: `E` is a class that implements the bound (if the bound is an interface).
- **Benefit**: Allows the compiler to guarantee the availability of specific methods (e.g., `compareTo`).

| Scenario | Code Example | Meaning |
| :--- | :--- | :--- |
| **Class** | `<E extends Number>` | `E` can be `Number`, `Integer`, `Double`, etc. |
| **Interface** | `<E extends Runnable>` | `E` can be any class that `implements Runnable`. |
| **Final Class** | `<E extends String>` | `E` can **only** be `String` (because it's final). |

> [!IMPORTANT]
> Even if the bound is an **Interface**, you must use the `extends` keyword in generics. Java uses this single keyword to simplify the syntax for all upper bounds.

> [!NOTE]
> Think of a bounded type like a professional certification. You can hire "any certified electrician," which guarantees they know how to handle wiring, rather than just hiring "any person" and hoping for the best.

```java
// E must implement Comparable
public class OrderedPair<E extends Comparable<E>> {
    private E left, right;

    public OrderedPair(E left, E right) {
        this.left = left;
        this.right = right;
    }

    public void order() {
        if (left.compareTo(right) > 0) {
            E temp = left;
            left = right;
            right = temp;
        }
    }
}
```

### 1.2 Multiple Bounds

- **Syntax**: Use the ampersand (`&`) to specify multiple constraints: `T extends ClassA & InterfaceB & InterfaceC`.
- **Order Requirement**: If a class bound is present, it **must** be listed first.
- **Single Inheritance**: Only one class is permitted in the bounds list due to Java's single implementation inheritance.

> [!IMPORTANT]
> Using a comma (`,`) instead of an ampersand (`&`) would declare multiple separate type parameters rather than adding multiple bounds to one parameter.

[Back to Top](#table-of-contents)

---

## 2. Wildcards in Generics

### 2.1 Wildcards for Beginners (The "Fruit Crate" Analogy)

Wildcards are the "key" to making generic collections flexible. Without them, Java is extremely strict about types.

- **The Problem**: A **`List<Apple>`** is NOT a **`List<Fruit>`**. If it were, you could accidentally add an Orange to an Apple list, which would cause a crash.
- **The Solution**: Use a **Wildcard (`?`)** to tell Java exactly how you intend to interact with the list (Reading vs Writing).

#### **The PECS Rule: Producer Extends, Consumer Super**

| Scenario | Rule | Analogy | Primary Use |
| :--- | :--- | :--- | :--- |
| **Reading Data** | `? extends T` | You are **consuming** fruit from a crate. You can safely call it "Fruit" but can't add anything new. | **Producer** (List passed is a Producer) |
| **Writing Data** | `? super T` | You are **producing** fruit to put in a crate. You can safely add an Apple to any Fruit-compatible crate. | **Consumer** (List passed is a Consumer) |

> [!TIP]
> Use `extends` when you want to **get** items out of a collection. Use `super` when you want to **put** items into a collection.

---

### 2.2 Upper Bounded Wildcards (? extends T)

- **Covariance**: Represents a list of "anything that is assignment compatible to T."
- **Primary Use**: **Reading** from a structure.
- **Safety**: The compiler ensures that whatever is in the list can be safely assigned to a variable of type `T`.
- **Writing Restriction**: You cannot add items to this list (except `null`) because the compiler doesn't know the exact subtype.

```java
public void processItems(List<? extends CharSequence> list) {
    for (CharSequence cs : list) {
        System.out.println(cs.length()); // Safe to read as CharSequence
    }
    // list.add("New String"); // COMPILATION ERROR
}
// in usage 
processItems(new ArrayList<String>()); // ok 
processItems(new ArrayList<StringBuilder>()); // ok 
processItems(new ArrayList<Number>()); // not ok : compiler stop us from prossessing Number as a CharSequence
```

### 2.2 Lower Bounded Wildcards (? super T)

- **Contravariance**: Represents a list of "anything that T can be assigned to."
- **Primary Use**: **Writing** to a structure.
- **Safety**: Ensures the list can safely accept an object of type `T`.
- **Reading Restriction**: Items read from the list are only guaranteed to be `Object`.

```java
public void addStrings(List<? super String> list) {
    list.add("Hello World"); // Safe to add a String
    // String s = list.get(0); // COMPILATION ERROR: list item is only known as Object
}
// in usage 
addStrings(new ArrayList<String>()); // ok 
addStrings(new ArrayList<Object>()); // ok 
addStrings(new ArrayList<Number>()); // not ok : compiler stop us from adding String to List<Number>

```
### Benefits of wildcards :
 
Wildcards are useful because they provide **flexibility while maintaining safety**. Without them, Java's generic collections are "frozen"—they only work with one exact type.  
  
Imagine you want to write a method that calculates the sum of all numbers in a list.

**Without Wildcards:**
```java
public double sum(List<Number> list) { ... }
```
- **The Failure**: You **cannot** pass a `List<Integer>` or a `List<Double>` to this method. You would have to write 10 different methods for 10 different number types!

**With Wildcards:**
```java
public double sum(List<? extends Number> list) { ... }
```
- **The Success**: This **one method** now works for `List<Integer>`, `List<Double>`, `List<Long>`, and even `List<BigDecimal>`. You've made your code universal.

[Back to Top](#table-of-contents)

---

### 2.4 Unbounded Wildcards (?)

- **Shorthand**: `List<?>` is equivalent to `List<? extends Object>`.
- **Use Case**: When the code only uses methods provided by `Object` or doesn't care about the specific type of the contents.

[Back to Top](#table-of-contents)

---

## 3. Declaration Constraints

### 3.1 Valid vs Invalid Declarations

- **`extends` vs `super`**: You can use `extends` in a type parameter declaration, but you **cannot** use `super`.
- **Wildcard Declarations**: The question mark (`?`) cannot be used to declare a named type parameter. It only constrains an existing type at the point of use.

```java
// VALID: E is declared with a bound
public <E extends CharSequence> void process(E element) { ... }

// INVALID: Cannot use wildcard in declaration
// public <?> void process(List<?> list) { ... } 

// INVALID: Cannot use 'super' in declaration
// public <E super String> void process(E element) { ... }
```

### 3.2 Static Method Requirements

- **Independence**: Static methods do not have access to the class's type parameters (e.g., an `E` declared at the class level).
- **Explicit Declaration**: Every static method must declare its own type parameters before the return type.

> [!WARNING]
> An instance method *can* access the class-level type parameter, but a static method exists outside the context of a specific instance and thus has no "expression" from which to infer the class's `E`.

[Back to Top](#table-of-contents)

---

## 4. Advanced Functional Patterns

### 4.1 Improving Generic Interfaces

- **Flexible APIs**: When writing functions, use wildcards to make the API more flexible for callers.
- **Rule of Thumb**: 
    - Argument types often benefit from `? super`.
    - Return types often benefit from `? extends`.

```java
// More flexible than exactly Function<A, A>
public void modify(Function<? super A, ? extends A> transformer) {
    this.value = transformer.apply(this.value);
}
```

[Back to Top](#table-of-contents)
