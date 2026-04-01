# Special Cases in Sealed Type Hierarchies

## Table of Contents
- [1. Syntax Shortcuts: Omitting the Permits Clause](#1-syntax-shortcuts-omitting-the-permits-clause)
- [2. Package and Module Constraints](#2-package-and-module-constraints)
- [3. Records and Enums as Leaf Nodes](#3-records-and-enums-as-leaf-nodes)
- [4. Intermediate and Leaf Node Rules Recap](#4-intermediate-and-leaf-node-rules-recap)

---

## 1. Syntax Shortcuts: Omitting the Permits Clause

### Concept Definition
Typically, a sealed class must explicitly list its permitted subclasses using the `permits` clause. However, if the sealed class and all its subclasses are defined within the **same source file**, the `permits` clause can be omitted. The compiler automatically identifies the subclasses within that file.

### Everyday Analogy
Imagine a family living in a single house. Because everyone is already together in one place, you don't need a guest list to know who belongs to the family. If a family member lives in a different house (different file), you must explicitly list them on the "official family register" (`permits` clause) for the system to recognize them as part of the family hierarchy.

### Java-Specific Implementation
```java
// Omitted permits clause because all classes are in the same file
public sealed class Base {
    public final class Sub1 extends Base {}
    public final class Sub2 extends Base {}
}

// This would FAIL if declared in a separate file without 'permits'
final class Sub3 extends Base {} 
```

> [!IMPORTANT]
> Any attempt to add a subclass in a different file without updating the `permits` clause will result in a compilation error, even if the subclass is in the same package as the sealed class.

[Back to Top](#table-of-contents)

---

## 2. Package and Module Constraints

### Concept Definition
To maintain the integrity of a "sealed" hierarchy, Java restricts where subclasses can be declared relative to the root sealed type. This prevents unauthorized extensions from outside the designated scope.

### Everyday Analogy
Think of a private club's membership rules. If the club operates under "Local Rules" (non-modular system), all members must live in the same neighborhood (same package). If the club operates under "Global Franchise Rules" (Java Platform Module System - JPMS), members can live in different neighborhoods (different packages) as long as they all belong to the same franchise (same module).

### Java-Specific Implementation
- **Non-Modular Environment**: All members of a sealed hierarchy (root and all subtypes) MUST reside in the same package.
- **Modular Environment (JPMS)**: Members can be in different packages but MUST be part of the same named module.

```java
// File: package com.fruits;
public sealed class Fruit permits Orange, Tomato {}

// This works in JPMS if in the same module, even if package is different
// package com.vegetables;
public final class Tomato extends Fruit {} 

// In a standard classpath (non-modular), the above fails. 
// Tomato would HAVE to be in com.fruits.
```

[Back to Top](#table-of-contents)

---

## 3. Records and Enums as Leaf Nodes

### Concept Definition
Records and Enums have specific structural constraints that allow them to participate in sealed hierarchies without traditional class modifiers like `final`.

### Everyday Analogy
- **Records**: Like a sealed envelope. Once the data is inside and the envelope is closed, you can't change it or add more layers to it (Records are implicitly final).
- **Enums**: Like a fixed set of choices on a menu. You can't add a new dish to the menu from outside the kitchen (Subclasses can only exist inside the enum declaration as constant bodies).

### Java-Specific Implementation
- **Records**: Are implicitly `final`. They can implement a sealed interface and serve as a leaf node. Using the `final` keyword on a record is allowed but redundant.
- **Enums**: Are not technically `final` (because constants can have class bodies), but they are sufficiently "closed." You **cannot** use the `final` keyword on an enum.

```java
public sealed interface BaseIF permits SubEnum, SubRecord {}

// Record as a valid leaf node
public record SubRecord(int id) implements BaseIF {}

// Enum as a valid leaf node
public enum SubEnum implements BaseIF {
    INSTANCE {
        @Override
        public void action() { System.out.println("Enum Action"); }
    };
    public abstract void action();
}
```

> [!WARNING]
> You cannot mark an Enum as `final` or `sealed`. They are permitted as leaf nodes because their subclassing is internally strictly controlled by the JVM.

[Back to Top](#table-of-contents)

---

## 4. Intermediate and Leaf Node Rules Recap

### Concept Definition
Every subtype of a sealed class must explicitly declare its own "sealed" status to ensure the compiler can reason about the entire hierarchy.

### Pedagogical Flow
1. **Root Node**: Must be marked `sealed`.
2. **Intermediate Node**: Can be `sealed` (requires its own `permits`) or `non-sealed`.
3. **Leaf Node**: Can be `final`, `non-sealed`, or a `record`/`enum`.

### Best Practice
> [!TIP]
> Use `non-sealed` sparingly. It effectively "opens" a branch of the hierarchy for arbitrary extension by any code, which can bypass the safety benefits of sealed types in pattern matching.

| Modifier | Extension Rule |
| :--- | :--- |
| `final` | No further subclasses allowed. |
| `sealed` | Only specific, listed subclasses allowed in its own `permits` clause. |
| `non-sealed` | Any subclass allowed; effectively returns to traditional inheritance. |

[Back to Top](#table-of-contents)
