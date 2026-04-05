# Lambda Expression Contexts

## Table of Contents
- [1. Target Typing and Contexts](#1-target-typing-and-contexts)
- [2. L-Value Context](#2-l-value-context)
- [3. Method Parameter and Return Contexts](#3-method-parameter-and-return-contexts)
  - [3.1 Method Argument Context](#31-method-argument-context)
  - [3.2 Return Statement Context](#32-return-statement-context)
- [4. Cast Context](#4-cast-context)
  - [4.1 Multiple Bounds](#41-multiple-bounds)
- [5. Closures and Variable Capture](#5-closures-and-variable-capture)

---

## 1. Target Typing and Contexts

When a lambda expression is declared, the compiler must unambiguously determine which functional interface it implements. This required type is inferred from the surrounding **context**.

Java supports four valid contexts for target typing a lambda expression:
1. **L-Value Assignment**: Assigning to a variable or array element.
2. **Method Arguments**: Passing as an argument to a method.
3. **Return Statements**: Returning from a method.
4. **Cast Expressions**: Explicitly casting the lambda to a functional interface type.

> [!NOTE]
> Think of a lambda expression as a chameleon. It doesn't have a fixed color (type) until it lands on a specific leaf (context) that demands a certain hue (functional interface).

---

## 2. L-Value Context

An l-value is simply a target that can be assigned a value (the left-hand side of an assignment).

- The assignment does not need to be to a simple variable.
- It can be to an array slot or an array initializer literal.

```java
public interface PredicateString {
    boolean test(String s);
}

public class LValueExample {
    public static void main(String[] args) {
        // Simple variable assignment
        PredicateString validLength = s -> s.length() > 3;

        // Array element assignment
        PredicateString[] psa = new PredicateString[1];
        psa[0] = s -> s.length() > 3;

        // Array literal assignment
        PredicateString[] psaLiteral = new PredicateString[] {
            s -> s.length() > 3
        };
    }
}
```

---

## 3. Method Parameter and Return Contexts

### 3.1 Method Argument Context

Lambdas are predominantly passed as arguments to methods. The compiler infers the intended functional interface from the method signature.

```java
public class MethodArgumentExample {
    public static void announce(PredicateString ps) {
        System.out.println(ps.test("Hello Context"));
    }

    public static void main(String[] args) {
        // The argument's target type is unambiguously inferred as the PredicateString interface
        announce((String s) -> s.length() > 3);
    }
}
```

> [!WARNING]
> Method overloads can sometimes cause ambiguity. If multiple overloaded methods accept different functional interfaces with similar method signatures, the compiler may struggle to infer the exact target type without a cast or explicit parameter typing.

### 3.2 Return Statement Context

Lambdas can be directly returned from methods. The expected return type of the method dictates the target functional interface.

```java
public class ReturnContextExample {
    public static PredicateString makeTest() {
        // The return type indicates the target context
        return s -> s.length() > 3;
    }
}
```

---

## 4. Cast Context

Sometimes, a target context is unavailable or ambiguous (e.g., when assigning to `Object`). In such cases, an explicit cast can force the target type.

```java
public class CastContextExample {
    public static void main(String[] args) {
        // INVALID: Object provides no context for inference
        // Object test1 = s -> s.length() > 3; // Compilation Error
        
        // VALID: Explicit cast provides the context
        Object test2 = (PredicateString) s -> s.length() > 3;
    }
}
```

### 4.1 Multiple Bounds

A cast context is uniquely empowered to assign **multiple bounds** to a lambda expression. The resulting object will implement all specified interfaces.

- Only one interface can declare an abstract method (the functional interface).
- Additional interfaces must have zero abstract methods (e.g., marker interfaces like `Serializable`).

```java
import java.io.Serializable;

public class MultipleBoundsExample {
    public static void main(String[] args) {
        // Creates a lambda that implements both PredicateString and Serializable
        Object test = (PredicateString & Serializable) s -> s.length() > 3;
    }
}
```

---

## 5. Closures and Variable Capture

Lambdas can act as *closures* because they bundle a function together with its surrounding state (lexical environment). When a lambda captures variables from its enclosing method or scope, it is essentially bringing those external values into its own context to use whenever the lambda is eventually executed.

- Any captured local variable must be **final** or **effectively final**.
- Effectively final means the variable's value is never altered after initialization, even if the `final` keyword is omitted.

> [!TIP]
> This restriction guards against race conditions in concurrent execution. It applies to the *reference*, not the object's *contents*.

```java
public class ClosureExample {
    public static void main(String[] args) {
        int[] counter = {0}; // Effectively final reference
        int threshold = 3;   // Effectively final primitive
        
        // counter is not reassigned, but its contents are modified.
        // threshold is never modified.
        PredicateString ps = s -> {
            counter[0]++; // Valid: mutating the array state, not the reference
            return s.length() > threshold; // Valid: reading effectively final variable
        };
        
        // threshold = 5; // Compilation Error: Breaks effectively final status
        // counter = new int[]{1}; // Compilation Error: Breaks effectively final status
    }
}
```
