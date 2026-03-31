# 01. Rules of Scope in Java

## Table of Contents
- [1. Scope Fundamentals](#1-scope-fundamentals)
  - [1.1 Defining Scope](#11-defining-scope)
  - [1.2 Scope vs. Accessibility](#12-scope-vs-accessibility)
  - [1.3 Simple vs. Qualified Names](#13-simple-vs-qualified-names)
- [2. Categories of Declaration Scope](#2-categories-of-declaration-scope)
  - [2.1 Local Variable Scope](#21-local-variable-scope)
  - [2.2 Member Scope](#22-member-scope)
  - [2.3 Formal Parameter Scope](#23-formal-parameter-scope)
  - [2.4 Static Import Scope](#24-static-import-scope)
- [3. Conflict Resolution and Boundaries](#3-conflict-resolution-and-boundaries)
  - [3.1 Duplicate Names and Shadowing](#31-duplicate-names-and-shadowing)
  - [3.2 Nested Class Scope Boundaries](#32-nested-class-scope-boundaries)
- [4. Special Case: Enum Constants in Switch](#4-special-case-enum-constants-in-switch)
  - [4.1 Enum-Typed switch Expression](#41-enum-typed-switch-expression)
  - [4.2 General Type switch (Pattern Matching)](#42-general-type-switch-pattern-matching)
- [5. Interface Statics and Scope](#5-interface-statics-and-scope)

---

## 1. Scope Fundamentals

### 1.1 Defining Scope
- **Scope**: The region of code where a declared entity can be reached using a **simple name**.
- **Analogy**: Like a "Office Intercom" – in your own department, you can just call out "Hey, Jim!" (simple name). In another department, you'd need "Jim from Accounting" (qualified name).

### 1.2 Scope vs. Accessibility
- **Scope**: Determines if the name is "known" in a region.
- **Accessibility**: Determines if the user has "permission" to use it.
- **Independent Matrix**: An item can be in scope but inaccessible (e.g., a `private` field in a superclass).

### 1.3 Simple vs. Qualified Names
- **Simple Name**: No prefix (e.g., `balance`).
- **Qualified Name**: Preceded by a type or reference (e.g., `Account.balance` or `user.balance`).

---

## 2. Categories of Declaration Scope

### 2.1 Local Variable Scope
- **Range**: From the point of declaration to the end of the immediately enclosing block.
- **Constraint**: Does not cover the entire block, only the code following the declaration.

### 2.2 Member Scope (Fields & methods)
- **Range**: Active throughout the entire enclosing type (class/interface), nested types, and inherited subtypes.
> [!NOTE]
> Private members are technically not "inherited," so they are inaccessible in derived types, preventing scope leak.

### 2.3 Formal Parameter Scope
- **Method/Lambda Arguments**: Active throughout the entire body.
- **For-Loop Variables**: From declaration to the end of the loop body.
- **Catch Clause Exceptions**: Limited to the catch block.
- **Try-with-Resources**: Limited to the try block.

```java 
import java.util.List;

public class LambdaScope {
    public static void main(String[] args) {
        List<String> names = List.of("Alice", "Bob", "Charlie");

        // The scope of 'name' starts at the arrow (->) 
        // and ends at the closing brace (})
        names.forEach(name -> {
            System.out.println("Hello, " + name);
        });

        // ERROR: 'name' is NOT in scope here!
        // System.out.println(name); 
    }
}
```

### 2.4 Static Import Scope
- **Impact**: Brings static fields or methods into scope for the whole **compilation unit** (the `.java` file).

---

## 3. Conflict Resolution and Boundaries

### 3.1 Duplicate Names and Shadowing
- **Strict Rule**: Two local variables in the same block cannot share a name—it's an unambiguous error.
- **Member Disambiguation**: A class can have a field and a method with identical names because the `()` in method calls provides a natural disambiguator.

```java
public class ScopeConflict {
    int x = 50; // Member field

    void process() {
        // int x = 10; // WOULD CAUSE CONFLICT with local scope
        System.out.println(x); // Refers to member field
    }
    
    void x() { // Method with same name as field - VALID
         System.out.println("Method called");
    }
}
```

### 3.2 Nested Class Scope Boundaries
- Nested classes create a fresh scope boundary.
- They allow for re-declaring names used in outer scopes (shadowing), as the boundary provides a clear separation.

---

## 4. Special Case: Enum Constants in Switch

### 4.1 Enum-Typed switch Expression
- If the switch expression is an `enum`, the constants are naturally "in scope."
- **Legacy Java**: Only unqualified names (e.g., `case RED`) were allowed.
- **Modern Java (SE 21)**: Qualified forms (e.g., `case Color.RED`) are now permitted.

### 4.2 General Type switch (Pattern Matching)
- When switching on a general type (e.g., `Object`), the compiler doesn't "know" the enum context.
- **Constraint**: Must use the **qualified name** (e.g., `case Color.GREEN`).
- **Requirement**: Pattern matching switches must be exhaustive (usually requiring a `default` case).

```java
// Switch on specific Enum type
switch (day) {
    case MONDAY -> ...
    case Days.TUESDAY -> ... // Permitted in modern Java
}

// Switch on Object (Pattern Matching)
Object obj = Days.MONDAY;
switch (obj) {
    case Days.MONDAY -> ... // MUST be qualified
    default -> ...
}
```

---

## 5. Interface Statics and Scope

- **Inheritance Gap**: Implementing an interface does **not** bring its static methods into your class scope.
- **Direct Access**: Must use `InterfaceName.method()`.
- **Static Import Solution**: Use `import static MyInterface.myMethod` to use the simple name `myMethod()`.

[Back to Top](#table-of-contents)

---
