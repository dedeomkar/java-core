# Using Bounds and Wildcards

## Table of Contents
- [1. Bounded Type Parameters](#1-bounded-type-parameters)
  - [1.1 The Extends Bound](#11-the-extends-bound)
  - [1.2 Multiple Bounds](#12-multiple-bounds)
- [2. Wildcards in Generics](#2-wildcards-in-generics)
  - [2.1 Upper Bounded Wildcards (? extends T)](#21-upper-bounded-wildcards--extends-t)
  - [2.2 Lower Bounded Wildcards (? super T)](#22-lower-bounded-wildcards--super-t)
  - [2.3 Unbounded Wildcards (?)](#23-unbounded-wildcards-)
- [3. Declaration Constraints](#3-declaration-constraints)
  - [3.1 Valid vs Invalid Declarations](#31-valid-vs-invalid-declarations)
  - [3.2 Static Method Requirements](#32-static-method-requirements)
- [4. Advanced Functional Patterns](#4-advanced-functional-patterns)
  - [4.1 Improving Generic Interfaces](#41-improving-generic-interfaces)

---

## 1. Bounded Type Parameters

### 1.1 The Extends Bound

- **Constraint Mechanism**: Restricts a type parameter to only those types that provide specific features or behaviors.
- **Assignment Compatibility**: In generics, `extends` means "is assignment compatible to" (covers both class inheritance and interface implementation).
- **Benefit**: Allows the compiler to guarantee the availability of specific methods (e.g., `compareTo`).

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

### 2.1 Upper Bounded Wildcards (? extends T)

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
```

### 2.3 Unbounded Wildcards (?)

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
