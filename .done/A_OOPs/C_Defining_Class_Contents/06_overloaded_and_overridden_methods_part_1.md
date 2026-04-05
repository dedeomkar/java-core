# Overloaded and Overridden Methods, Part 1

## Table of Contents
- [1. Method Overloading Concepts](#1-method-overloading-concepts)
- [2. Overloading Rules and Collisions](#2-overloading-rules-and-collisions)
- [3. The Resolution Order (Priority)](#3-the-resolution-order-priority)
- [4. Ambiguity and Compilation Failures](#4-ambiguity-and-compilation-failures)

---

## 1. Method Overloading Concepts

### Concept Definition
- **Overloading**: This occurs when multiple methods in the same class (or inherited from a parent) share the same name but have different **argument type sequences**. This allows you to perform similar logic on different types of data.

### Everyday Analogy
- **The Coffee Machine**: Imagine a coffee machine with a single "Brew" button. 
  - If you insert a **Single-Serve Pod** (argument A), it makes one cup.
  - If you insert a **Filter with Grounds** (argument B), it makes a whole pot.
  - The name of the action is always "Brew," but the machine automatically knows what to do based on what you put inside it.

### Java-Specific Implementation
- **Differentiation**: Overloading is decided strictly by the method's name and its parameters. 
- **Non-Factors**: Return types, access modifiers (public/private), and keywords like `synchronized` or `final` do **not** contribute to overloading. If they are the only difference, the code will not compile.

```java
public class OverloadConcept {
    public void print(String s) { System.out.println(s); }
    public void print(int i)    { System.out.println(i); } // Valid Overload
    
    // INVALID: Only return type differs
    // public int print(String s) { return 0; } 
}
```


---

## 2. Overloading Rules and Collisions

### Concept Definition
- Certain types in Java are considered identical for overloading purposes, which can lead to "collisions" where the compiler cannot tell methods apart.

### Java-Specific Implementation
- **Generics**: `List<String>` and `List<Integer>` are both just `List` at runtime (due to Type Erasure) and cannot be used to overload each other.
- **Varargs vs. Arrays**: A method with `String[]` and one with `String...` are considered to have the same signature.

```java
public class CollisionDemo {
    public void process(List<String> list) {}
    // public void process(List<Integer> list) {} // ERROR: Collision (both are List)

    public void display(String[] names) {}
    // public void display(String... names) {} // ERROR: Collision (both are String[])
}
```


---

## 3. The Resolution Order (Priority)

### Concept Definition
- When you call an overloaded method with arguments that don't *exactly* match any signature, Java follows a strict "Pecking Order" to find the best fit.

### Everyday Analogy
- **The Tailor**: When you go to a tailor for a suit:
  1. They first look for a suit that **fits perfectly** (Exact Match).
  2. If none fit, they find one that is **slightly too large** and can be taken in (Widening Promotion).
  3. They only try **stretchy fabrics** (Autoboxing) or **adjustable waistbands** (Varargs) as a last resort if standard sizes fail.

### Java-Specific Implementation (The Priority List)
1. **Exact Match or Widening**: The compiler first tries to match the type exactly or promote a primitive to a larger type (e.g., `int` to `long`).
2. **Autoboxing/Unboxing**: If widening fails, it tries wrapping/unwrapping (e.g., `int` to `Integer`).
3. **Varargs**: The lowest priority; it is only checked if all other attempts fail.

```java
public class ResolutionDemo {
    public static void compute(long l)      { System.out.println("Widening"); }
    public static void compute(Integer i)   { System.out.println("Boxing"); }
    public static void compute(int... nums) { System.out.println("Varargs"); }

    public static void main(String[] args) {
        int x = 10;
        // int can widen to long (Step 1) OR box to Integer (Step 2).
        // Widening wins!
        compute(x); // Outputs: Widening
    }
}
```


---

## 4. Ambiguity and Compilation Failures

### Concept Definition
- If Java finds two possible matches at the **same priority level**, it won't guess. It will simply fail to compile, citing "Ambiguity."

### Java-Specific Implementation
- This often happens when one argument can be widened in two different ways across two methods.

```java
public class AmbiguityDemo {
    public static void talk(int i, long l) { System.out.println("int-long"); }
    public static void talk(long l, int i) { System.out.println("long-int"); }

    public static void main(String[] args) {
        // talk(10, 10); // ERROR: Ambiguous! 
        // Both args are 'int'. Which one should be widened to 'long'?
    }
}
```

> **Important Consideration**:
> In low-latency systems, **overloading with primitives vs. objects** can be a trap. Calling the boxed version (e.g., `Integer`) incurs an allocation cost. Always ensure your hot-path methods have a clear, primitive-exact match to avoid the "silent" performance penalty of autoboxing.

