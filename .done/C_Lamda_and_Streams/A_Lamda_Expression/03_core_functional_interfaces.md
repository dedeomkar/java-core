# Core Functional Interfaces

## Table of Contents
- [1. Core Concepts and Generics Limitations](#1-core-concepts-and-generics-limitations)
- [2. The Big Four Core Interfaces](#2-the-big-four-core-interfaces)
  - [2.1 Function](#21-function)
  - [2.2 Consumer](#22-consumer)
  - [2.3 Supplier](#23-supplier)
  - [2.4 Predicate](#24-predicate)
- [3. Arity Variations (Bi-interfaces)](#3-arity-variations-bi-interfaces)
  - [3.1 BiFunction and BiConsumer](#31-bifunction-and-biconsumer)
- [4. Operator Interfaces](#4-operator-interfaces)
  - [4.1 UnaryOperator and BinaryOperator](#41-unaryoperator-and-binaryoperator)
- [5. Primitive Variations](#5-primitive-variations)
  - [5.1 Primitive Return Types](#51-primitive-return-types)
  - [5.2 Primitive Argument Types](#52-primitive-argument-types)
- [6. Other Common Functional Interfaces](#6-other-common-functional-interfaces)
  - [6.1 Runnable and Callable](#61-runnable-and-callable)
  - [6.2 Comparator](#62-comparator)
- [7. Composition and Utility Methods](#7-composition-and-utility-methods)
  - [7.1 Chaining Functions and Predicates](#71-chaining-functions-and-predicates)

---

## 1. Core Concepts and Generics Limitations

While custom functional interfaces can be created, the Java standard library provides over 40 standard functional interfaces in `java.util.function` to handle common lambda expression scenarios.

Generics are powerful but have limitations when it comes to lambdas:
- **Arity Limits**: Generics cannot natively represent a variable number of arguments (arity).
- **Return Type Constraints**: Generics do not distinguish between methods that return a value and those that return `void`.
- **Primitive Inefficiency**: Generics only work with reference types. Using primitives requires autoboxing and auto-unboxing, which introduces overhead. 

> [!NOTE]
> Think of the core functional interfaces as pre-made adapter cables. Instead of soldering a custom cable (writing a custom interface) every time you need to connect two devices (pass behavior), you just grab a standard USB or HDMI cable from the `java.util.function` package.

---

## 2. The Big Four Core Interfaces

### 2.1 Function

Represents a transformation: it takes a single argument and produces a result.

- **Method**: `R apply(T t)`

```java
import java.util.function.Function;

public class FunctionExample {
    public static void main(String[] args) {
        Function<String, Integer> lengthFunction = s -> s.length();
        int length = lengthFunction.apply("High-Frequency Trading"); // 22
    }
}
```

### 2.2 Consumer

Represents an operation that accepts a single input argument and returns no result (`void`). Typically used for side effects like printing or modifying state.

- **Method**: `void accept(T t)`

```java
import java.util.function.Consumer;
import java.util.List;

public class ConsumerExample {
    public static void main(String[] args) {
        Consumer<String> logger = msg -> System.out.println("LOG: " + msg);
        
        // Explicitly calling accept
        logger.accept("Initialization complete");
        
        // Or using it where a Consumer is expected (forEach internally calls accept)
        List.of("Order 1", "Order 2").forEach(logger);
    }
}
```

### 2.3 Supplier

Represents a supplier of results. It takes no arguments and returns a value. Often used for deferred execution or factory generation.

- **Method**: `T get()`

```java
import java.util.function.Supplier;
import java.time.Instant;

public class SupplierExample {
    public static void main(String[] args) {
        Supplier<Long> timestampSupplier = () -> Instant.now().toEpochMilli();
        long currentTimestamp = timestampSupplier.get();
    }
}
```

### 2.4 Predicate

Represents a boolean-valued function of one argument. Used for filtering or matching.

- **Method**: `boolean test(T t)`

```java
import java.util.function.Predicate;

public class PredicateExample {
    public static void main(String[] args) {
        Predicate<Integer> isPositive = n -> n > 0;
        boolean result = isPositive.test(150); // true
    }
}
```

---

## 3. Arity Variations (Bi-interfaces)

When operations require exactly two parameters, Java provides `Bi` prefixed interfaces.

### 3.1 BiFunction and BiConsumer

- `BiFunction<T, U, R>` takes two arguments and returns a result via `R apply(T t, U u)`.
- `BiConsumer<T, U>` takes two arguments and returns nothing via `void accept(T t, U u)`.
- `BiPredicate<T, U>` takes two arguments and returns a boolean via `boolean test(T t, U u)`.

```java
import java.util.function.BiFunction;
import java.util.function.BiConsumer;

public class BiInterfacesExample {
    public static void main(String[] args) {
        BiFunction<Double, Double, Double> calculatePrice = (price, qty) -> price * qty;
        
        BiConsumer<String, Double> printReceipt = (item, cost) -> 
            System.out.printf("Item: %s, Cost: %.2f%n", item, cost);
            
        printReceipt.accept("GOOGL Shares", calculatePrice.apply(150.0, 10.0));
    }
}
```

---

## 4. Operator Interfaces

Operators are specialized `Function` or `BiFunction` interfaces where the operand(s) and the result are of the same explicit type.

### 4.1 UnaryOperator and BinaryOperator

- `UnaryOperator<T>` extends `Function<T, T>`.
- `BinaryOperator<T>` extends `BiFunction<T, T, T>`.

> [!IMPORTANT]
> Unary and Binary operators don't define new methods; they inherit `apply` from their super-interfaces but enforce type uniformity across inputs and outputs.

```java
import java.util.function.UnaryOperator;
import java.util.function.BinaryOperator;

public class OperatorExample {
    public static void main(String[] args) {
        UnaryOperator<String> toUpperCase = s -> s.toUpperCase();
        
        BinaryOperator<Integer> sum = (a, b) -> a + b;
        int total = sum.apply(5, 10);
    }
}
```

---

## 5. Primitive Variations

To avoid the performance penalties of autoboxing, Java offers specific interfaces for `int`, `long`, and `double` (operations are generally not performed in smaller sizes like `short` or `byte` due to CPU architecture optimization and Java's arithmetic promotion rules).

### 5.1 Primitive Return Types

Prefixes like `ToInt`, `ToLong`, and `ToDouble` indicate the return type is a primitive. 
- E.g., `ToIntFunction<T>` method `int applyAsInt(T value)`.

*Special case:* `IntSupplier`, `LongSupplier`, and `DoubleSupplier` return primitives, although their names don't use the `To` prefix since they have zero arguments.

### 5.2 Primitive Argument Types

Prefixes like `Int`, `Long`, and `Double` signify that the input parameter is a primitive.
- E.g., `IntConsumer` method `void accept(int value)`.
- E.g., `DoubleBinaryOperator` method `double applyAsDouble(double left, double right)`.

```java
import java.util.function.IntConsumer;
import java.util.function.ToIntFunction;

public class PrimitiveExample {
    public static void main(String[] args) {
        // Primitive parameter, avoids Integer unboxing
        IntConsumer printInt = x -> System.out.println(x * 2);
        printInt.accept(42);
        
        // Primitive return type, avoids Integer boxing
        ToIntFunction<String> stringLength = s -> s.length();
        int len = stringLength.applyAsInt("Low Latency");
    }
}
```

---

## 6. Other Common Functional Interfaces

Some highly utilized functional interfaces live outside `java.util.function`.

### 6.1 Runnable and Callable

- `Runnable` (in `java.lang`): Takes no arguments and returns `void` via `void run()`.
- `Callable<V>` (in `java.util.concurrent`): Takes no arguments and returns a value via `V call() throws Exception`. Crucially, `Callable` allows checked exceptions to be thrown, unlike `Supplier`.

```java
import java.util.concurrent.Callable;

public class ConcurrencyInterfaces {
    public static void main(String[] args) throws Exception {
        Runnable task = () -> System.out.println("Processing async...");
        
        Callable<String> dataFetcher = () -> {
            Thread.sleep(100); // Checked exception allowed
            return "Data stream open";
        };
    }
}
```

### 6.2 Comparator

- `Comparator<T>` (in `java.util`): Takes two arguments of the *same type* and returns an `int` via `int compare(T o1, T o2)`.
- It acts just like a `ToIntBiFunction<T, T>` but predates standard functional interfaces and is semantically strictly meant for determining ordinal grouping.

---

## 7. Composition and Utility Methods

Many functional interfaces provide `default` and `static` methods to chain computations together in a readable or functionally pure manner.

### 7.1 Chaining Functions and Predicates

- `Function` has `.andThen()` and `.compose()`.
- `Predicate` has `.and()`, `.or()`, and `.not()`.
- `Function.identity()` is a static factory returning an object unchanged.

```java
import java.util.function.Function;
import java.util.function.Predicate;

public class CompositionExample {
    public static void main(String[] args) {
        Function<Integer, Integer> multiplyByTwo = x -> x * 2;
        Function<Integer, Integer> addTen = x -> x + 10;
        
        // Chaining functions: multiplies first, then adds ten
        Function<Integer, Integer> combinedProcess = multiplyByTwo.andThen(addTen);
        System.out.println(combinedProcess.apply(5)); // Output: 20
        
        Predicate<Integer> isEven = x -> x % 2 == 0;
        Predicate<Integer> isPositive = x -> x > 0;
        
        // Combining predicates
        Predicate<Integer> validNumber = isEven.and(isPositive);
        System.out.println(validNumber.test(4)); // true
    }
}
```
