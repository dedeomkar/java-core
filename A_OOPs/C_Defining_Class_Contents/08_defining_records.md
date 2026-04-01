# Defining Records

## Table of Contents
- [1. Introduction to Records](#1-introduction-to-records)
- [2. Basic Syntax and Declaration](#2-basic-syntax-and-declaration)
- [3. Implicit Features and Members](#3-implicit-features-and-members)
- [4. Equality and HashCode](#4-equality-and-hashcode)
- [5. Record Constraints and Rules](#5-record-constraints-and-rules)

---

## 1. Introduction to Records

### Concept Definition
- **Records**: A special type of class designed to act as a transparent carrier for immutable data.
- **Aggregate Data Type**: Groups related values (components) into a single container without the boilerplate of traditional POJOs.
- **Design Intent**: Focuses on "data as data" rather than data as encapsulated state that might change.

### Everyday Analogy
- **The Sealed Passport**:
    - A passport carries specific data: Name, ID, Nationality.
    - Once issued, you don't "set" a new name on that specific passport (Immutability).
    - If you compare two passports with the exact same data, they represent the same identity information (Value-based equality).

### Java-Specific Implementation
- Records were introduced to reduce the massive boilerplate required for simple data transfer objects (DTOs).
- They automatically provide storage and accessors for the data they describe.

---

## 2. Basic Syntax and Declaration

### Syntax Breakdown
- Use the `record` keyword followed by the name and a **component list** in parentheses.
- **Mandatory**: Even if empty, parentheses `()` and curly braces `{}` are required.

```java
// A simple record declaration
public record Customer(String name, int creditLimit) { }
```

### Usage and Construction
- Records are instantiated just like regular classes using the `new` keyword.
- The compiler automatically provides a "Canonical Constructor" matching the component list.

```java
public class RecordDemo {
    public static void main(String[] args) {
        // Constructing a record
        Customer c1 = new Customer("Fred", 1000);
        
        // Printing the record (Uses automatic toString)
        System.out.println(c1); // Outputs: Customer[name=Fred, creditLimit=1000]
    }
}
```

---

## 3. Implicit Features and Members

### Automatic Accessors
- Accessors do **not** use the `get` prefix.
- The method name matches the component name exactly.

```java
Customer c1 = new Customer("Jim", 500);

// Accessing components
String name = c1.name();       // NOT getName()
int limit = c1.creditLimit();  // NOT getCreditLimit()
```

### Component Storage
- Components are stored as `private final` instance fields.
- This ensures the data carrier remains unmodifiable after construction.

> [!NOTE]
> While the record fields are `final` (unmodifiable references), if the component itself is a mutable object (like a `StringBuilder` or `ArrayList`), the *content* of that object can still be changed.

---

## 4. Equality and HashCode

### Value-Based Equality
- Records implement `equals()` and `hashCode()` automatically based on their content.
- Two records are equal if all their components are equal.

```java
Customer c1 = new Customer("Fred", 1000);
Customer c2 = new Customer("Fred", 1000);
Customer c3 = new Customer("Freddy", 2000);

System.out.println(c1.equals(c2)); // true
System.out.println(c1.equals(c3)); // false

// HashCodes will be identical for c1 and c2
System.out.println(c1.hashCode() == c2.hashCode()); // true
```

---

## 5. Record Constraints and Rules

### Class Hierarchy
- Records are **implicitly final**; they cannot be subclassed.
- Records **cannot extend** other classes (they implicitly extend `java.lang.Record`).
- Records **can implement** interfaces.

### Field Restrictions
- You **cannot declare instance fields** manually inside the record body.
- All instance state must be defined in the component list.
- **Static Fields** and **Static Initializers** are permitted.

```java
public record Player(String username) implements Serializable {
    // static int playerCount = 0; // Legal
    // int score = 0;             // ILLEGAL: Managed instance fields not allowed
}
```

### Accessor Rules
- Accessors are always `public`, take zero arguments, and have no `throws` clause.
- **Varargs Support**: If the last component is a varargs (`String... items`), the accessor returns an array (`String[]`).

> [!WARNING]
> You cannot have component names that collide with methods in `java.lang.Object` (e.g., you cannot name a component `wait`, `notify`, or `getClass`).

[Back to Top](#table-of-contents)
