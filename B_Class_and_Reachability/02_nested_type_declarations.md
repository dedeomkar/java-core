# Nested Type Declarations

## Table of Contents
- [1. Understanding Nested Types](#1-understanding-nested-types)
- [2. Instantiation](#2-instantiation)
- [3. Access Control and Visibility Rules](#3-access-control-and-visibility-rules)
- [4. The Builder Pattern Example](#4-the-builder-pattern-example)

---

## 1. Understanding Nested Types

### Concept Definition
A type (like a class or interface) declared inside another type is generally known as a nested type. Depending on whether it has the `static` modifier, nested types can be distinguished conceptually:
- **Static Nested Class**: Behaves like a regular top-level class that just happens to be lexically nestled inside another class. It does not have an implicit reference to an instance of the enclosing class.
- **Inner Class**: A non-static nested class (covered in later chapters).

By default, the fully qualified name of a nested type is `OuterName.NestedName`. However, within the scope of the outer class, it can usually be referred to simply by its short name, `NestedName`.

### Best Practice/Consideration
- Nested classes can have any of the standard access modifiers: `public`, `protected`, default (package-private), and `private`.
- The enclosing (outer) class and its nested classes share privileged access to each other's members, including `private` fields and methods.

[Back to Top](#table-of-contents)

---

## 2. Instantiation

### Concept Definition
Since a static nested class is essentially a top-level class tucked inside a namespace, you do not need an instance of the outer class to instantiate it.

### Java-Specific Implementation
To instantiate a public static nested class from an entirely different scope (outside the enclosing class):

```java
public class Outer {
    public static class Nested {
        public void display() {
            System.out.println("Inside static nested class.");
        }
    }
}

class Client {
    public static void main(String[] args) {
        // Instantiate using the Outer.Nested qualification
        Outer.Nested nestedObj = new Outer.Nested();
        nestedObj.display();
    }
}
```

> **Important Considerations**: If you have imported the nested class directly (e.g., `import package.Outer.Nested;`), you can instantiate it using just `new Nested()`.

[Back to Top](#table-of-contents)

---

## 3. Access Control and Visibility Rules

### Concept Definition
A common misconception in Java is that a `private` member is visible *only* inside the exact class that declares it. This is **inaccurate**.

The true visibility rule for a `private` member is: **It is accessible anywhere inside the enclosing top-level curly braces that surround the declaration.**

### Everyday Analogy
Imagine a large, fenced-in family estate (the top-level class). Inside the estate, there are several separate houses (nested classes). If something is marked "Private Family Property" (`private`), anyone living within the main fence (the top-level curly braces) can use it, even if they live in different houses on the estate.

### Java-Specific Implementation
```java
public class Outer { // <--- Top-level curly brace starts here
    private int outerX = 10;

    private static class Nester1 {
        private int nester1X = 20;

        public void show(Outer t, Nester2 v) {
            // Can access its own private field
            System.out.println(this.nester1X);
            
            // Can access private field of enclosing class
            System.out.println(t.outerX);
            
            // Can access private field of sibling nested class!
            System.out.println(v.nester2X);
        }
    }

    private static class Nester2 {
        private int nester2X = 30;
        
        public void show(Nester1 u) {
            // Can access sibling's private field
            System.out.println(u.nester1X); 
        }
    }
} // <--- Top-level curly brace ends here
```

> **Important Considerations**: In the example above, `Nester1` and `Nester2` are siblings. Because they share the same top-level enclosing braces (`Outer`), they can access each other's private members, provided they have a reference to an instance of the target class.

[Back to Top](#table-of-contents)

---

## 4. The Builder Pattern Example

### Concept Definition
The generous visibility of `private` members within the top-level braces is what allows common design patterns like the **Builder Pattern** to function seamlessly and securely.

### Java-Specific Implementation
A standard builder pattern utilizes a private constructor in the target class (so it cannot be instantiated directly from the outside) and a `public static` Builder class nested inside it.

```java
public class BuildMe {
    private String name;
    private String date;

    // 1. Private constructor: Access is restricted to these top-level braces
    private BuildMe() {}

    public static Builder builder() {
        // Accesses private constructor of the nested class
        return new Builder(); 
    }

    public static class Builder {
        private BuildMe template;

        // 2. Private constructor
        private Builder() {
            // Accesses the private constructor of the enclosing class
            this.template = new BuildMe(); 
        }

        public Builder name(String name) {
            // Accesses private field 'name' of the enclosing class
            this.template.name = name; 
            return this;
        }

        public Builder date(String date) {
            // Accesses private field 'date' of the enclosing class
            this.template.date = date; 
            return this;
        }

        public BuildMe build() {
            BuildMe result = this.template;
            // Clear the template to avoid accidental reuse
            this.template = null; 
            return result;
        }
    }
}
```
### Usage example :

```java
public class BuilderPatternUsageSample {
    public static void main(String[] args) {
        // Example 1: Build with all properties
        BuildMe obj1 = BuildMe.builder()
                .name("John Doe")
                .date("2026-03-29")
                .build();
        
        System.out.println("Object 1 created with name and date");
        
        // Example 2: Build with only name
        BuildMe obj2 = BuildMe.builder()
                .name("Jane Smith")
                .build();
        
        System.out.println("Object 2 created with only name");
        
        // Example 3: Build with only date
        BuildMe obj3 = BuildMe.builder()
                .date("2025-12-25")
                .build();
        
        System.out.println("Object 3 created with only date");
        
        // Example 4: Build with no properties (empty)
        BuildMe obj4 = BuildMe.builder()
                .build();
        
        System.out.println("Object 4 created empty");
    }
}
```
The usage sample demonstrates:

- Fluent API chaining
- Flexibility in setting properties
- Multiple scenarios (all properties, partial, and empty builds)
- Immutability of BuildMe class, which also enables thread-safety

### Best Practice/Consideration
In this builder layout:
- The `BuildMe` constructor is private, so only `BuildMe` or its inner/nested classes can call it.
- The `Builder` class creates the `BuildMe` instance and directly accesses its private fields (`name`, `date`) to populate the data.
- The access is perfectly legal because the `Builder` class sits strictly within the top-level curly braces of `BuildMe`.

[Back to Top](#table-of-contents)
