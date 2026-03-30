# Instance and Static Fields, Part 2

## Table of Contents
- [1. Unqualified Identifiers](#1-unqualified-identifiers)
- [2. Name Resolution Rules](#2-name-resolution-rules)
- [3. Class vs. Instance Context](#3-class-vs-instance-context)
- [4. Early vs. Late Binding (Shadowing vs. Overriding)](#4-early-vs-late-binding-shadowing-vs-overriding)
- [5. Interface Fields](#5-interface-fields)

---

## 1. Unqualified Identifiers

### Definition
- An unqualified identifier is a variable name used directly without any prefix (e.g., `theField` instead of `myObj.theField` or `this.theField`).

[Back to Top](#table-of-contents)

---

## 2. Name Resolution Rules

### Search Order
When encountering an unqualified identifier, the Java compiler resolves the reference in the following prioritized sequence:
1. **Method Local Variable**: The compiler first looks for an active local variable with that exact name. Local variables always take highest precedence and shadow any fields.
2. **Current Level Instance/Class**: If not found locally, it prefixes the field with an implicit `this.` (if in an instance context) or an implicit `ClassName.` (if working in a static context).
3. **Class Hierarchy Walk**: The compiler repeats step 2, walking up the class hierarchy. The closest match in the hierarchy becomes the default.

### Disambiguation
- If multiple fields with the same name exist at different levels, you can explicitly disambiguate them using:
  - **Class Name Prefix**: `ClassName.identifier` (for static members).
  - **Instance `this` Prefix**: `this.identifier` (for this class's instance members).
  - **Instance `super` Prefix**: `super.identifier` (for the direct parent's instance members).
  - **Casting**: `((ParentClass) this).identifier` (to reach a specific parent's field).

> **Important Consideration**:
> You can only travel exactly *one* level up the hierarchy using `super.`. Using `super.super.identifier` is invalid Java syntax.

[Back to Top](#table-of-contents)

---

## 3. Class vs. Instance Context

### Static Features
- **Class Association**: Static elements belong to the class as a whole rather than a specific instance. Consequently, there is only *one* shared copy of a static variable.

```java
public class StatClass {
    public static int index = 99;
}
```

### Static Syntactic Sugar and Traps
- **Bad Form**: Early Java designers allowed static members to be accessed via an instance object prefix (`obj.staticField`).
- **Potential Confusion**: This is syntactically valid but severely misleading, as the change affects the single shared variable. Note that this shortcut *does not work* for static methods located within interfaces.

```java
public class StatTest {
    public static void main(String[] args) {
        StatClass s1 = new StatClass();
        StatClass s2 = new StatClass();
        
        // Allowed syntax but considered terrible practice:
        s1.index++; // index = 100
        s2.index++; // index = 101
        
        // This output is 101, despite being logically called through s1
        System.out.println(s1.index);
    }
}
```

### Keyword Restrictions
- **The `var` Keyword**: You cannot use `var` for class/instance fields. It is strictly reserved for method-local variables.

[Back to Top](#table-of-contents)

---

## 4. Field Shadowing (No Polymorphism for Fields)

### Concept Definition
- **No Overriding for Fields**: Unlike methods, fields do not support "polymorphism" or "overriding". If a child class declares a field with the exact same name as a parent class field, it simply *hides* or *shadows* the parent's field rather than replacing it.

### Everyday Analogy
- **The Two ID Cards**: Imagine a person who works for two companies. When you ask to see their ID, the address you read depends entirely on *which ID card* (the reference type) they hand you, even though it's the exact same person standing in front of you (the actual object in memory).

### Java-Specific Implementation
- **Compile-Time Decision**: Java decides which field to access entirely at **compile-time**. It looks *strictly* at the variable's declared type (the reference), completely ignoring the actual object type created in memory.

```java
class Parent {
    int x = 100;
}

class Child extends Parent {
    int x = 200; // This 'shadows', but does not 'override', Parent's x
}

public class FieldBindingDemo {
    public static void main(String[] args) {
        // We create a Child object, but hold it in a Parent reference variable
        Parent p = new Child();
        
        // At compile-time, Java sees 'p' is declared as type 'Parent'.
        // So it uses Parent's 'x' (100), ignoring that it's actually a 'Child' object!
        System.out.println(p.x); // Output: 100
        
        // If we cast the reference to Child, it uses Child's 'x'
        System.out.println(((Child) p).x); // Output: 200
    }
}
```

[Back to Top](#table-of-contents)

---

## 5. Interface Fields

### Default Modifiers
- All fields explicitly or implicitly declared within an `interface` enforce the `public`, `static`, and `final` modifiers. 

### Implementation Nuances
- **No `super` Binding**: While interfaces behave similarly to abstract classes conceptually, `super` cannot be used to invoke or capture fields of implemented interfaces; it solely works for class ancestry.

```java
interface ParentInterface {
    int X = 100;
}

class Child implements ParentInterface {
    public void printX() {
        // System.out.println(super.X); // COMPILATION ERROR: super cannot be used for interfaces
        System.out.println(ParentInterface.X); // Correct: Use the interface name
        System.out.println(X); // Correct: Unqualified (if not shadowed)
    }
}
```

[Back to Top](#table-of-contents)
