# Method References

## Table of Contents
- [1. Core Characteristics](#1-core-characteristics)
- [2. The Four Syntax Patterns](#2-the-four-syntax-patterns)
  - [2.1 Static Method References](#21-static-method-references)
  - [2.2 Constructor References](#22-constructor-references)
  - [2.3 Instance Method of an Arbitrary Object of a Particular Type](#23-instance-method-of-an-arbitrary-object-of-a-particular-type)
  - [2.4 Instance Method of an Existing Object](#24-instance-method-of-an-existing-object)
- [3. Advanced Considerations](#3-advanced-considerations)
  - [3.1 Type Matching and Ambiguity](#31-type-matching-and-ambiguity)
  - [3.2 The Usage of `super` and `this`](#32-the-usage-of-super-and-this)

---

## 1. Core Characteristics

Method references provide a compact syntax for lambda expressions that purely delegate behavior to an already existing method.

- **Purpose**: They emphasize **what** behavior is being passed (by focusing on the method name and enclosing class) rather than **how** the arguments are marshaled.
- **Limitation**: Method references entirely hide argument passing. You cannot manipulate, reorder, perform calculations on, or introduce additional arguments. The parameters dictated by the functional interface must unambiguously connect to the method being referenced.

> [!NOTE]
> Think of a method reference as a direct, rigid pipeline. If a traditional lambda expression is a custom-built connector that allows you to filter or mix water as it passes through, a method reference is a rigid, straight pipe. It only snaps into place if the connections on both ends match perfectly.

---

## 2. The Four Syntax Patterns

### 2.1 Static Method References

Used when a lambda delegates entirely to a static method.
- **Pattern**: `ClassName::staticMethodName`
- **Rule**: The arguments supplied by the functional interface perfectly match the arguments given to the static method.

```java
// Traditional Lambda
BiFunction<Integer, Integer, Integer> maxLambda = (a, b) -> Math.max(a, b);
System.out.println(maxLambda.apply(10, 20)); // Prints: 20

// Method Reference Equivalent
BiFunction<Integer, Integer, Integer> maxRef = Math::max;
System.out.println(maxRef.apply(10, 20)); // Prints: 20

// High-Density Spring Boot Example (Routing)
// RouterFunctions.route(path, Handler::handleRequest)
```

### 2.2 Constructor References

Constructors can be referenced just like static methods, essentially acting as factory functions.

- **Object Construction Pattern**: `ClassName::new`
- **Array Construction Pattern**: `Type[]::new`
- **Rule**: The functional interface dictates the number and types of arguments, mapping to the appropriate overloaded constructor. For arrays, a single `int` argument defines the designated array size.

```java
// Traditional Lambda (Object Creation)
Function<String, StringBuilder> sbLambda = str -> new StringBuilder(str);

// Method Reference Equivalent
Function<String, StringBuilder> sbRef = StringBuilder::new;

// Traditional Lambda (Array Creation)
IntFunction<String[]> arrLambda = size -> new String[size];

// Method Reference Equivalent
IntFunction<String[]> arrRef = String[]::new;
```

### 2.3 Instance Method of an Arbitrary Object of a Particular Type

Utilized when the first argument provided by the lambda directly acts as the target instance upon which the method is invoked, and any subsequent arguments are passed into that method.

- **Pattern**: `ClassName::instanceMethodName`
- **Rule**: At least one argument must be present. Argument 1 acts as the context receiver, and arguments 2 through N seamlessly map to the method parameters.

```java
// Traditional Lambda
BiFunction<String, String, Boolean> startsWithLambda = (str, prefix) -> str.startsWith(prefix);

// Method Reference Equivalent
BiFunction<String, String, Boolean> startsWithRef = String::startsWith;
```

### 2.4 Instance Method of an Existing Object

Used when a specific external object instance is captured, and its method is invoked using all lambda arguments.

- **Pattern**: `instanceReference::instanceMethodName`
- **Rule**: The object expression is evaluated **exactly once** during the creation of the method reference (behaving like an effectively final capture). The lambda arguments map completely to the method's parameters.

```java
List<String> names = new ArrayList<>();

// Traditional Lambda
Consumer<String> addLambda = name -> names.add(name);

// Method Reference Equivalent
Consumer<String> addRef = names::add;
```

> [!IMPORTANT]
> The evaluation timing is critical. If you have `complexCalculation()::doStuff`, the `complexCalculation()` is evaluated instantly immediately when the method reference is formed, and not dynamically upon every subsequent execution of the lambda's inner behavior.
> 
> ```java
> // The getWorkerFactory() method is evaluated exactly once, directly at assignment.
> Runnable task = getWorkerFactory()::doWork; 
> 
> // doWork() executes twice, but getWorkerFactory() is NOT re-evaluated.
> task.run(); 
> task.run(); 
> ```

---

## 3. Advanced Considerations

### 3.1 Type Matching and Ambiguity

A method reference must unambiguously match the target functional interface types.

- **Void Compatibility**: Expression statements like `aSet::add` (which intrinsically returns a `boolean`) can be utilized in contexts where a `void` return is expected (e.g., `Consumer<String>`). The return value is safely discarded.
- **Ambiguity**: If multiple expansions or overloaded methods are structurally valid and the compiler cannot determine the single intended method, compilation fails.

```java
// Valid: println accepts Object/any, returns void. Consumer expects void.
Consumer<LocalDate> printRef = System.out::println; 

// Valid: startsWith returns boolean. Predicate expects boolean return.
Predicate<String> testPrefix = "Java"::startsWith;
```

```java
class AmbiguityDemo {
    static void process(int a, int b) {}
    static void process(Integer a, Integer b) {}
}

// COMPILATION ERROR: Ambiguous method reference. Both process() signatures physically match.
// BiConsumer<Integer, Integer> ambiguousRef = AmbiguityDemo::process;
```

### 3.2 The Usage of `super` and `this`

You can seamlessly employ `this` and `super` as the instance reference within an instance context (non-static methods).

```java
class ParentHandler {
    public void processTask() { 
        System.out.println("Executing baseline routing logic."); 
    }
}

class ChildHandler extends ParentHandler {
    public Runnable getParentExecutor() {
        // Method reference using 'super' keyword targeting parent method
        return super::processTask; 
    }
    
    public Runnable getSelfExecutor() {
        // Method reference using 'this' targeting current class method
        return this::processTask;
    }
}
```
