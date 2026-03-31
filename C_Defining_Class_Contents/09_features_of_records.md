# Features of Records

## Table of Contents
- [1. Customizing and Adding Methods](#1-customizing-and-adding-methods)
- [2. The Canonical Constructor](#2-the-canonical-constructor)
- [3. The Compact Constructor](#3-the-compact-constructor)
- [4. Overloaded Constructors](#4-overloaded-constructors)
- [5. Advanced Record Features](#5-advanced-record-features)

---

## 1. Customizing and Adding Methods

### Customizing Auto-Methods
- You can replace the automatically generated `equals()`, `hashCode()`, `toString()`, and accessor methods.
- **Replacement vs. Override**: Since there is no `super` implementation for these record-specific methods, you are essentially replacing them from scratch.

### Adding Custom Logic
- Records can have additional instance or static methods.
- **Constraint**: No `native` or `abstract` methods are allowed.

```java
public record MathOperation(int a, int b) {
    // Custom instance method
    public int sum() {
        return a + b;
    }

    // Custom static method
    public static MathOperation identity() {
        return new MathOperation(0, 0);
    }
}
```

---

## 2. The Canonical Constructor

### Concept Definition
- The **Canonical Constructor** is the primary constructor whose argument list matches the record components exactly.
- It is responsible for initializing the implicit private final fields.

### Hand-Coding Rules
- If you write the canonical constructor manually:
    - The accessibility must not be more restrictive than the record class.
    - Parameter names **must match** component names exactly.
    - Every field must be initialized.

```java
public record User(String id, String email) {
    // Manual Canonical Constructor
    public User(String id, String email) {
        this.id = id.toLowerCase(); // Normalization
        this.email = email;
    }
}
```

---

## 3. The Compact Constructor

### Concept Definition
- A **Compact Constructor** is a shorthand syntax designed specifically for validation and normalization.
- It does **not** have a parameter list and does **not** manually assign values to fields.

### Everyday Analogy
- **The Assembly Line Inspector**:
    - Before the "Shipping Container" (Record) is sealed, the inspector checks the items (Components).
    - If an item is faulty, they throw it out (Exception).
    - If an item is dusty, they wipe it (Normalization) before it's officially stored.

### Java-Specific Implementation
- Logic in the compact constructor acts as a **preamble** to the canonical constructor.
- You operate on parameters as local variables; you **cannot** use `this.field` or read the fields yet.

```java
public record Account(int balance) {
    // Compact Constructor (no parens)
    public Account {
        if (balance < 0) {
            throw new IllegalArgumentException("Balance cannot be negative");
        }
        // No 'this.balance = balance' needed; it happens automatically after this block
    }
}
```

---

## 4. Overloaded Constructors

### Delegation Requirements
- Any additional constructors (overloads) **must delegate** to the canonical constructor.
- This is done using the `this(...)` call.

```java
public record Point(int x, int y) {
    // Overloaded constructor for 1D points
    public Point(int x) {
        this(x, 0); // Delegates to the canonical constructor
    }

    // Overloaded constructor for default points
    public Point() {
        this(0); // Chains: Point() -> Point(x) -> Point(x, y)
    }
}
```

---

## 5. Advanced Record Features

### Deserialization Behavior
- Unlike regular classes, records use the **canonical constructor** during deserialization.
- This ensures that any validation or normalization logic in the constructor is applied when the object is reconstructed from a stream.

```java
public record SafeRange(int start, int end) implements Serializable {
    public SafeRange {
        if (start > end) {
            throw new IllegalArgumentException("Start must be <= End");
        }
    }
}

// AT DESERIALIZATION:
// Even if an attacker manually crafts a serialized byte stream where 'start' is 100 
// and 'end' is 50, the JVM's deserialization call to the canonical constructor 
// will trigger the IllegalArgumentException. The invalid object is never born.
```

> [!TIP]
> **Data Integrity Guarantee**: 
> In traditional Java classes, deserialization can sometimes bypass constructor-based validation (using `readObject` or non-standard mechanisms). Records solve this security and data integrity gap by mandating that high-level constructor logic is *never* skipped.

### Static Initializers and Fields
- Records can have static fields and static initializers, but **no instance initializers**.

```java
public record Config(String key) {
    private static final String VERSION;

    static {
        VERSION = "1.0.0"; // Legal: Static initializer
    }

    // { /* illegal */ } // ILLEGAL: Instance initializers are prohibited
}
```

> [!IMPORTANT]
> Because records are aimed at immutability, the `final` components cannot be reassigned. If you need to change a value, you should create a new record instance with the updated data.

[Back to Top](#table-of-contents)
