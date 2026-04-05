# Source Files and Basic Type Declarations

## Table of Contents
- [1. Source File Structure](#1-source-file-structure)
- [2. Type Declarations](#2-type-declarations)
- [3. Class Modifiers](#3-class-modifiers)
- [4. Type Members and Restrictions](#4-type-members-and-restrictions)

---

## 1. Source File Structure

### Concept Definition
A Java source file is the foundational unit of a Java program, containing the necessary declarations to define types (classes, interfaces, records, etc.) and organize them into packages. 

- A source file can contain at most one `package` statement, which must be the first syntactically significant element (only whitespace and comments can precede it).
- It can contain any number of `import` statements, placed after the package statement.
- The base name of the file must exactly match the name of any `public` type it contains.
- Consequently, a single source file can contain at most one `public` type. 

### Java-Specific Implementation
```java
// Must be the first statement
package com.example.structure;

// Imports follow the package
import java.util.List;

// The public class name must match the file name: SourceExample.java
public class SourceExample {
    // Class body
}

// Non-public types can coexist in the same file
class HelperType {
    // Helper logic
}
```

> **Important Considerations**: 
> - If a file has two package statements, compilation fails.
> - If the package name doesn't match the physical directory structure (e.g., package `src` inside a directory named `other`), compilation can fail or cause runtime/classpath issues depending on the compiler.
> - Having two `public` types in one file will cause a compilation error.


---

## 2. Type Declarations

### Concept Definition
Java allows various types of declarations at the top level of a file, each serving a different structural purpose in object-oriented execution. 

A top-level type can be:
- A class (regular, `sealed`, `non-sealed`, or `final`)
- An `abstract` class
- An enumeration (`enum`)
- A `record` (introduced after Java 11)
- An `interface`
- An annotation (`@interface`)

### Best Practice/Consideration
- **Concrete Finality**: Only concrete classes can be explicitly marked as `final`.
- **Implicit Finality**: Records and annotations are implicitly `final`. Enums are effectively final since they can only be subclassed inside their own declaration.

### Java-Specific Implementation
The general structure of a class declaration involves modifiers, the class layout, generalizations, and sealed clauses:
```java
// Modifiers | "class" | Name | Generalizations | Permits Clause
public abstract class Animal extends Entity implements Identifiable {
    // Body
}
```


---

## 3. Class Modifiers

### Concept Definition
Modifiers define the accessibility and inheritance limitations of a type. Java has nine possible modifiers for a class, but they carry restrictive combinations.

1. **Access Modifiers**:
   - `public`: Allowed on top-level and member classes.
   - `protected` / `private`: Allowed *only* on member classes (nested/inner). 
   - Note: Only one access modifier is permitted at a time. Local and anonymous classes cannot take access modifiers.

2. **Implementation Modifiers**:
   - `static`: Can only be applied to member classes (creating a nested class). Local classes in static contexts are implicitly static, but cannot use the keyword directly.
   - `abstract`: Indicates the class must be subclassed. Cannot be `final`.
     - Constructors can be `private`, ensuring only member classes can subclass it.
   - `sealed` / `non-sealed` / `final`: Mutually exclusive. A `sealed` type restricts its subclasses, requiring a `permits` clause or implicit co-located children. A `non-sealed` class must have a `sealed` parent. Local types cannot use these.
   - `strictfp`: Now completely obsolete (since Java 17).

### Everyday Analogy
Think of class modifiers like security badges for a building:
- `public` means the front door is open to anyone.
- `private` means the room is only accessible to people already inside the main office (the enclosing class).
- `final` means the blueprints cannot be altered or expanded upon by future architects.

### Java-Specific Implementation
```java
public class OuterClass {
    // Protected member class
    protected class InnerClass { }
    
    // Private static nested class
    private static class NestedClass { }
}

// Illegal: Combining abstract and final
// abstract final class ImpossibleClass { }
```

> **Important Considerations**: 
> - Abstract classes exist solely to be subclassed; thus, combining `abstract` with `final` is a logical contradiction and results in a compilation failure.


---

## 4. Type Members and Restrictions

### Concept Definition
A regular or abstract class declared at the top level can encapsulate various members, categorized broadly into static contexts (belonging to the class) and instance contexts (belonging to an object).

### Java-Specific Implementation
A standard top-level class can contain:
1. Static and instance fields
2. Static and instance methods
3. Static and instance initializers (blocks)
4. Constructors
5. **Nested Classes**: Static member classes.
6. **Inner Classes**: Instance member classes.
7. **Implicitly Static Types**: `enum`, `record`, and `interface` declarations inside a class are implicitly static.

```java
public class ComprehensiveClass {
    // 1. Fields
    static String staticField = "Shared";
    int instanceField = 42;

    // 2. Initializers
    static { System.out.println("Static init"); }
    { System.out.println("Instance init"); }

    // 3. Constructors
    public ComprehensiveClass() { }

    // 4. Nested/Inner Types
    static class StaticNested { }
    class InstanceInner { }
    
    // 5. Implicitly Static Types
    enum Status { ACTIVE, INACTIVE }
    interface Handler { }
}
```

### Special Type Rules
- **Abstract Classes, Enums, Records**: Behave like regular classes but with unique semantic limitations (e.g., Records cannot easily extend other classes).
- **Interfaces**: Always `abstract` (cannot be `final`).
  - Fields inside an interface are inherently `public static final`.
- **Annotations**: A specific type of interface.

> **Important Considerations**: Be wary of scenarios where an interface has a field like `int x = 1;`. This is implicitly `public static final x = 1;`. Modifying it later in code will result in a compilation error because it is a constant.

