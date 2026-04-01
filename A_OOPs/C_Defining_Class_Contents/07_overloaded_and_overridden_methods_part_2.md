# Overloaded and Overridden Methods, Part 2

## Table of Contents
- [1. Overriding vs. Hiding](#1-overriding-vs-hiding)
- [2. Liskov Substitution Principle (LSP) in Java](#2-liskov-substitution-principle-lsp-in-java)
- [3. Dynamic Binding: The Actor and the Script](#3-dynamic-binding-the-actor-and-the-script)
- [4. Bridge Methods (Internal Mechanics)](#4-bridge-methods-internal-mechanics)

---

## 1. Overriding vs. Hiding

### Concept Definition
- **Overriding**: When a subclass provides its own version of a **parent instance method**. The subclass method "replaces" the parent's behavior for that specific object.
- **Hiding**: When a subclass defines a **static method** with the same name and signature as a parent's static method. Static methods are "hidden," not overridden, because they are tied to the class, not the instance.

### everyday Analogy
- **The Substitute Teacher**: Think of a class syllabus (the parent method). If a replacement teacher (the child method) arrives, they are expected to follow the same syllabus (the signature). They can't suddenly start teaching a different subject (change types) or lock the classroom door (reduce access), but they can provide their own unique way of explaining the material (the implementation).

### Java-Specific Implementation
- **Requirement**: The name and the argument type sequence must match exactly.
- **Final/Private**: You cannot override a `final` or `private` method.

```java
class Parent {
    // Instance method (Overridable)
    public void greet() { System.out.println("Hello from Parent"); }
    
    // Static method (Hidable)
    public static void show() { System.out.println("Static Parent"); }

    // Overloading not Overriding !
    public void greet(String message) { System.out.println("Hello " + message); }
}

class Child extends Parent {
    @Override
    public void greet() { System.out.println("Hello from Child"); }

    // @Override // COMPILATION ERROR: Static methods cannot be overridden, only hidden
    public static void show() { System.out.println("Static Child"); }

}

public class OverrideHideDemo {
    public static void main(String[] args) {
        Parent p = new Child();
        
        // DYNAMIC BINDING (Object type wins)
        p.greet(); // Outputs: Hello from Child
        
        // STATIC BINDING (Reference type wins - method was hidden, not overridden)
        p.show();  // Outputs: Static Parent

        // HOW TO PRINT "Static Child"?
        Child.show();         // Way 1: Use the class name (RECOMMENDED)
        ((Child) p).show();   // Way 2: Cast the reference to the child type

        // Method Overloading NOT Overriding
        p.greet("Dan"); 
    }
}
```

> [!CAUTION]
> **Reference Visibility vs Object Implementation**
> Remember: The **Reference Type** acts as a filter. It determines which method names and parameter signatures the compiler is allowed to "see." If a method (or an overload) only exists in the subclass, you **must cast** the reference to that subclass type before you can invoke it.

[Back to Top](#table-of-contents)

---

## 2. Liskov Substitution Principle (LSP) in Java

### Concept Definition
- The LSP states that a child should be a "perfect substitute" for its parent. To avoid bugs, Java enforces rules so the caller isn't surprised by the child's behavior.

### Rules of Overriding
1. **Return Type**: Must be identical for primitives. For objects, it can be **Covariant** (the same type or a subclass).
2. **Exceptions**: A child cannot throw new or broader **checked exceptions** than the parent.
3. **Accessibility**: A child cannot be *less* accessible (e.g., you can't override a `public` method with a `protected` one).

```java
class Producer {
    protected CharSequence produce() { return "Data"; }
}

class StringProducer extends Producer {
    @Override
    public String produce() { return "Safe Data"; } // Valid: 'String' is a 'CharSequence' (Covariant)
    
    // public void produce() {} // INVALID: Return type is not compatible
}
```

[Back to Top](#table-of-contents)

---

## 3. Dynamic Binding: The Actor and the Script

### Concept Definition
- **Dynamic Binding**: At runtime, Java decides which method to run based on the **actual object type**, not the reference variable's type.

### Java-Specific Implementation
- This polymorphism only applies to instance methods. Fields and static methods use the reference type (Static Binding).

```java
// [ already seen in previous chapters ]
Sound s = new Music();
s.play(); // Runs Music's play() -> Dynamic Binding (The Actor)
System.out.println(s.volume); // Reads Sound's volume field -> Static Binding (The Script)
```

[Back to Top](#table-of-contents)

---

## 4. Bridge Methods (The Universal Adapter)

### Concept Definition
- **The Problem (Type Erasure)**: Java's Generics only exist at compile-time. At runtime, a `Comparable<String>` becomes a raw `Comparable` that expects an `Object`.
- **The Solution (Bridge Methods)**: If your class implements `Comparable<String>` and defines `compareTo(String)`, the JVM would be confused because it's looking for `compareTo(Object)`. To fix this, the compiler secretly inserts a **Bridge Method** that accepts an `Object`, casts it to a `String`, and then calls your real method.

### Everyday Analogy
- **The Universal Adapter**: Imagine you have a European appliance (the generic interface) that expects a **European plug** (an `Object`). But your wall socket (your specific class) only accepts a **US plug** (a `String`). You cannot plug one into the other directly. A **Bridge Method** acts as the **Universal Adapter**—it takes the European plug, adapts it, and safely connects it to your US socket.

### Java-Specific Implementation
- You never write bridge methods yourself; the compiler handles it. However, you can "trigger" them by calling a method through a raw-type reference.

```java
class MyNumber implements Comparable<MyNumber> {
    public int value;
    @Override
    public int compareTo(MyNumber other) { 
        return Integer.compare(this.value, other.value); 
    }
}

// UNDER THE HOOD (The Bridge Method):
// public int compareTo(Object o) {
//    return compareTo((MyNumber) o); // BRIDGE: Casts and forwards the call
// }
```

> **Important Consideration**:
> In low-latency performance tuning, be aware of bridge methods. Because they involve a **type cast from Object**, they can cause a tiny performance overhead and, more importantly, could lead to `ClassCastException` if types are misused at runtime.

[Back to Top](#table-of-contents)
