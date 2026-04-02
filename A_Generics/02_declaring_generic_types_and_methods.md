# Declaring Generic Types and Methods

## Table of Contents
- [1. Generic Class Declarations](#1-generic-class-declarations)
  - [1.1 Defining Type Parameters](#11-defining-type-parameters)
  - [1.2 Instantiation and Type Inferencing](#12-instantiation-and-type-inferencing)
- [2. Multiple Type Parameters](#2-multiple-type-parameters)
  - [2.1 Comma-Separated List](#21-comma-separated-list)
- [3. Generic Methods](#3-generic-methods)
  - [3.1 Static Generic Methods](#31-static-generic-methods)
  - [3.2 Instance Generic Methods](#32-instance-generic-methods)
- [4. Scope and Accessibility Constraints](#4-scope-and-accessibility-constraints)
  - [4.1 Static Context Restrictions](#41-static-context-restrictions)
  - [4.2 Inner and Nested Classes](#42-inner-and-nested-classes)
- [5. Generic Inheritance](#5-generic-inheritance)
  - [5.1 Passing Parameters to Parent Classes](#51-passing-parameters-to-parent-classes)

---

## 1. Generic Class Declarations

### 1.1 Defining Type Parameters

- **Placeholder Declaration**: The generic type parameter (usually `E`, `T`, `K`, `V`) is declared immediately after the class name.
- **Consistency Checking**: Once declared, the compiler uses this placeholder to ensure all uses of that type within the class are consistent.

> [!NOTE]
> Think of a generic class like a specialized shipping crate with labeled slots. You don't know what's inside yet, but you know that everything going into the "left" slot must match the "left" type label.

```java
public class Pair<E> {
    private E left;
    private E right;

    public Pair(E left, E right) {
        this.left = left;
        this.right = right;
    }

    public E getLeft() { return left; }
}
```

### 1.2 Instantiation and Type Inferencing

- **Assignment to Expression**: Generic type parameters are associated with *expressions*, not with the underlying type itself.
- **Diamond Operator (`<>`)**: Allows the compiler to infer the type based on the assignment context, reducing boilerplate.

```java
// Expression-based type assignment
Pair<String> ps = new Pair<>("NIHAO", "Hello"); 

// The compiler works out that E must be String for this instance
String leftValue = ps.getLeft(); 
```

[Back to Top](#table-of-contents)

---

## 2. Multiple Type Parameters

### 2.1 Comma-Separated List

- Classes and interfaces can accept multiple parameters (e.g., `Map<K, V>`).
- These are declared as a comma-separated list within the angle brackets.

```java
// Example: A key-value mapping structure
public interface MyMap<K, V> {
    void put(K key, V value);
    V get(K key);
}
```

> [!IMPORTANT]
> Standard naming conventions use single capital letters: `K` for Key, `V` for Value, `E` for Element, `T` for Type.

[Back to Top](#table-of-contents)

---

## 3. Generic Methods

### 3.1 Static Generic Methods

- **Standalone Expression**: Because static methods lack an instance context, they **must** declare their own type parameters before the return type.
- **Independence**: They cannot access the type parameters defined at the class level.

> [!NOTE]
> Think of a static generic method as a **standalone workstation**. Because it is `static`, it isn't connected to the building's main power (the class's type parameters). It must specify exactly what tool types (`K` and `V`) it needs right at the entrance.

```java
public class Processor {
    // Static methods must declare <K, V> themselves
    public static <K, V> void processElement(Map<K, V> map, K key, Consumer<V> op) {
        V element = map.get(key);
        if (element != null) op.accept(element);
    }
}
// called like this :
Map<Integer, String> mis = Map.of(1, "One", 2, "Two");

processElement(mis, 1, v -> System.out.println("Number is " + v));
// Here you don't have to write processElement<Integer, String>(mis, 1, ...)—the compiler is smart enough 
// to "solve the equation" for you based on the arguments you pass in.
```

> [!NOTE]
> This pattern is called Behavior Parameterization. Instead of just passing data (like a string), you are passing behavior (code) into the method to be executed later.

#### **Understanding `Consumer<V> op.accept()`**

| Component | Role | Description |
| :--- | :--- | :--- |
| **`Consumer<V>`** | **Functional Interface** | A "blueprint" for an operation that takes one input (`V`) and returns nothing. |
| **`op`** | **Object Instance** | The actual behavior (often a lambda) passed into the method. |
| **`accept()`** | **Interface Method** | The "Start" button that executes the provided behavior on a specific piece of data. |

[Back to Top](#table-of-contents)

### 3.2 Instance Generic Methods

- **Merged Scope**: Instance methods have access to the class-level type parameters but can also define their own additional parameters.
- **Method-Specific Types**: Useful for transformations where the output type differs from the internal storage type.

> [!NOTE]
> Imagine a "Transformation Room" in a factory. It can use the factory's standard raw materials (class-level `E`), but it can also bring in a special tool (method-level `F`) to produce a completely different product.

```java
public class Container<E> {
    private E item;

    // Instance method adding its own type parameter <F>
    // modifier | <new_generic> | return_type | method_name (params)
    public <F> Container<F> map(Function<E, F> transformer) {
        F newItem = transformer.apply(item);
        return new Container<>(newItem);
    }
}
```

[Back to Top](#table-of-contents)

---

## 4. Scope and Accessibility Constraints

### 4.1 Static Context Restrictions

- **Static vs Instance**: A static method or field cannot refer to a class-level type parameter `T`.
- **Reasoning**: The type `T` is tied to a specific instance expression. Static members exist globally across the class, independent of any instance.

### 4.2 Inner and Nested Classes

- **Inner Classes (Instance)**: Have access to the enclosing class's type parameters.
- **Static Nested Classes**: Like static methods, they are isolated and cannot see the enclosing type's parameters.
- **Inner Interfaces**: Are **implicitly static**, meaning they cannot use the enclosing class's type parameters.

> [!WARNING]
> Attempting to use an enclosing type parameter `T` in an inner interface will result in a compilation error because interfaces cannot be "instance-specific" in their generic declaration.

[Back to Top](#table-of-contents)

---

## 5. Generic Inheritance

### 5.1 Passing Parameters to Parent Classes

- Subclasses can "pass up" their own type parameters to a generic parent class.
- This allows the subclass to specialize or constrain the behavior of the parent.

```java
public class TextPair<F extends CharSequence> extends Pair<F> {
    public TextPair(F left, F right) {
        super(left, right);
    }
}
```

- **Specialization**: In the example above, `TextPair` ensures that its `Pair` functionality is only ever used with `CharSequence` types (like `String` or `StringBuilder`).

[Back to Top](#table-of-contents)
