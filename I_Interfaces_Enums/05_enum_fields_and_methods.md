# Enum Fields and Methods

## Table of Contents
- [1. Core Capabilities](#1-core-capabilities)
  - [1.1 Members in Enums](#11-members-in-enums)
  - [1.2 Base Class (java.lang.Enum)](#12-base-class-javalang-enum)
- [2. Standard Instance Methods](#2-standard-instance-methods)
  - [2.1 Name vs ToString](#21-name-vs-tostring)
  - [2.2 Placement and Comparison](#22-placement-and-comparison)
- [3. Obtaining Enum Values](#3-obtaining-enum-values)
  - [3.1 Explicit and Type-Specific Retrieval](#31-explicit-and-type-specific-retrieval)
  - [3.2 Generic Enum Value Retrieval](#32-generic-enum-value-retrieval)
- [4. Advanced Implementation Details](#4-advanced-implementation-details)
  - [4.1 Custom Instance Data](#41-custom-instance-data)
  - [4.2 Initialization Flow](#42-initialization-flow)
  - [4.3 Iteration and Display](#43-iteration-and-display)

---

## 1. Core Capabilities

### 1.1 Members in Enums

- Enums are full-fledged classes and can contain fields, methods (static or instance), and nested types.
- **Nested Interfaces**: Always implicitly static, regardless of declaration.
- **Placement**: All members must follow the semicolon-terminated list of enum constants.

> [!NOTE]
> Think of an Enum as a "Fixed Population" class. You can give each "citizen" (Enum value) their own passport (fields) and specialized skills (methods), but you can't create new citizens after the town is founded (compilation).

### 1.2 Base Class (java.lang.Enum)

- Every Enum implicitly extends `java.lang.Enum`.
- This provides several built-in methods that all enums inherit.
- Because Java only allows single inheritance, an Enum cannot extend any other class.

---

## 2. Standard Instance Methods

### 2.1 Name vs ToString

- **name()**: Returns the exact name of the constant as declared (final).
- **toString()**: Defaults to returning the name but is **not final**—it can be overridden for better representation.
- **ordinal()**: Returns the zero-based index of the constant in its declaration list.

```java
public enum Suit { HEARTS, DIAMONDS, CLUBS, SPADES }

// Execution:
Suit heart = Suit.HEARTS;
System.out.println(heart.name());    // Prints: HEARTS
System.out.println(heart.ordinal()); // Prints: 0
```

### 2.2 Placement and Comparison

- **compareTo()**: Based on the source code declaration order (natural ordering of enums).
- **equals() and hashCode()**: Implemented according to standard conventions.
- **Reference Comparison**: `==` is the most efficient and preferred way to compare Enum values since each constant is a singleton instance.

> [!IMPORTANT]
> Because every reference to a specific Enum value (like `Suit.HEARTS`) always points to the exact same object in a JVM, reference comparison (`==`) is guaranteed to work and is faster than `equals()`.

[Back to Top](#table-of-contents)

---

## 3. Obtaining Enum Values

### 3.1 Explicit and Type-Specific Retrieval

- **Static Access**: Accessing constants directly via dot notation.
- **valueOf(String name)**: A static method automatically generated for every Enum type.
- **values()**: Returns an array containing all the constants of that Enum in the order they are declared.

```java
Suit diamond = Suit.DIAMONDS;
Suit heart = Suit.valueOf("HEARTS"); // Case-sensitive
Suit[] allSuits = Suit.values();     // {HEARTS, DIAMONDS, CLUBS, SPADES}
```

### 3.2 Generic Enum Value Retrieval

- You can also use the static `Enum.valueOf` method, which requires the Enum class reference.
- Both `valueOf` methods throw a `java.lang.IllegalArgumentException` if the provided text doesn't match any declared constant.

```java
// Generic approach:
Suit heart = Enum.valueOf(Suit.class, "HEARTS");
```

[Back to Top](#table-of-contents)

---

## 4. Advanced Implementation Details

### 4.1 Custom Instance Data

- Enums can have private instance variables and constructors to store metadata.
- Constructors are run during class loading for each constant.

```java
public enum Suit {
    HEARTS("coeurs"), DIAMONDS("carreaux");

    private final String frenchTranslation;

    private Suit(String translation) {
        this.frenchTranslation = translation;
    }

    public String getTranslation() { return frenchTranslation; }
}
```

### 4.2 Initialization Flow

- Constructor arguments for each value are specified in the constant declaration.
- Static fields/initializers in an Enum run **after** all constants have been constructed.

> [!WARNING]
> If you add fields or methods after the constant list, the semicolon at the end of the constant list is **mandatory**.

### 4.3 Iteration and Display

- Combining `values()`, custom `toString()` overrides, and instance methods allows for sophisticated data management.

```java
@Override
public String toString() {
    String lower = this.name().toLowerCase();
    // Capitalize first letter: "Hearts"
    return lower.substring(0, 1).toUpperCase() + lower.substring(1);
}

// Usage in iteration:
for (Suit s : Suit.values()) {
    System.out.println(s + " : " + s.getTranslation());
}
// Outputs: Hearts : coeurs
```

[Back to Top](#table-of-contents)
