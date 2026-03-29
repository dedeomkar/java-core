# Instance and static fields, part 1

## Table of Contents
- [1. Field Declarations](#1-field-declarations)
  - [1.1 General Declaration Form](#11-general-declaration-form)
  - [1.2 Variable Modifiers](#12-variable-modifiers)
  - [1.3 Types and Multiple Variables](#13-types-and-multiple-variables)
- [2. Default Initialization](#2-default-initialization)
  - [2.1 Primitives and References](#21-primitives-and-references)
  - [2.2 The Final Constraint](#22-the-final-constraint)
- [3. Arrays as Fields](#3-arrays-as-fields)
- [4. Forward References and Corners](#4-forward-references-and-corners)
  - [4.1 Evaluating Initializer Blocks](#41-evaluating-initializer-blocks)
- [5. Method Locals versus Fields](#5-method-locals-versus-fields)

---

## 1. Field Declarations

### 1.1 General Declaration Form
Fields in Java classes follow a standardized form:
*   **Modifiers**: Typically access control keywords, static, final, etc.
*   **Type**: The data type (e.g., `List<E>`, `int`).
*   **Identifier**: The variable's name.
*   **Initialization**: Optional assignments to a value upon instantiation.

### 1.2 Variable Modifiers
- Allowable modifiers include access modifiers (`public`, `protected`, `private`), variable characteristics (`static`, `final`), or more advanced memory hints (`transient`, `volatile`).
- Modifiers cannot be repeated and the order is directed by convention rather than strict syntactic demand.

### 1.3 Types and Multiple Variables
- Generics may safely be declared as types.
- Multiple variables of identical type can share a declaration using commas.
> **Important Consideration**: It is impermissible to mix heterogeneous types in the same comma-separated structure. Doing so causes compilation failure.

```java
// Compilation fails: cannot mix generic types in a single declaration!
// List<E> le, lf, List<String> ls; 

// The correct, separated structure:
List<E> le, lf;
List<String> ls;
```

---

## 2. Default Initialization

### 2.1 Primitives and References
Fields inherently receive a default initialization (zero-like) simultaneously with object creation memory allocation:
*   **Numerics**: `0` or `0.0`
*   **Boolean**: `false`
*   **Char**: `'\u0000'` (number zero)
*   **Reference Types**: `null`

> **Note**: Even arrays act as fields in this respect and their child elements receive zeros or nulls reliably upon creation.

### 2.2 The Final Constraint
- A field declared as `final` must be explicitly initialized. Relying on default initialization for final fields will prompt a compiler error. 
```java
public class Demo {
    // Compilation fails! Must be explicitly initialized.
    final int strictValue; 
}
```

---

## 3. Arrays as Fields
Java supports two array declaration flavors:

1.  **Preferred bracket position**: Preceding the identifier.
```java
int[][] array; // Conventional Java approach
```
2.  **Deprecated C-style position**: Proceeding the identifier.
```java
int array[][]; // Strongly discouraged, yet entirely legal syntax
```

---

## 4. Forward References and Corners

### 4.1 Evaluating Initializer Blocks
- Class loading forces all `static` initialization blocks to execute top to bottom sequentially.
- Surprisingly, blocks can execute assignments to static fields *before* those fields are actually declared. 
- You may use a qualified identifier (e.g., `className.fieldName`) if referring forward to a field's value before it's formally declared in the code. A naked "unqualified" reference in a forward context fails compilation.

```java
class Initializers {
    static {
        // Safe forward read because we've qualified it
        System.out.println(Initializers.count); 
        count = 99; // Assignment without qualification works here
    }
    
    // Declaration strictly located beneath the block
    static int count; 
}
```

---

## 5. Method Locals versus Fields
- Method local variables never receive free default initialization. 
- Thus, attempting to read an uninitialized local variable fails immediately at compile time.

```java
class ScopeCheck {
    static int x; // Static field implicitly initialised to 0
    
    public static void main(String[] args) {
        int y; // Local variable gets NO default initialisation
        
        System.out.println(x); // Valid, outputs 0
        // System.out.println(y); // ERROR: y may not have been initialized
    }
}
```
- It is only the *read* of the uninitialized variable that angers the compiler; a local variable declared yet never read evaluates fine.

---

[Back to Top](#table-of-contents)
