# The Monad-Like Methods

## Table of Contents
- [1. Stream Processing Fundamentals](#1-stream-processing-fundamentals)
  - [1.1 Laziness and 'Pull' Execution](#11-laziness-and-pull-execution)
  - [1.2 Pure Functions](#12-pure-functions)
- [2. The Map Operation](#2-the-map-operation)
  - [2.1 Core Behavior](#21-core-behavior)
  - [2.2 Type Signatures](#22-type-signatures)
- [3. The Filter Operation](#3-the-filter-operation)
  - [3.1 Core Behavior](#31-core-behavior)
  - [3.2 Type Signatures](#32-type-signatures)
- [4. The FlatMap Operation](#4-the-flatmap-operation)
  - [4.1 Core Behavior](#41-core-behavior)
  - [4.2 Type Signatures](#42-type-signatures)

---

## 1. Stream Processing Fundamentals

### 1.1 Laziness and 'Pull' Execution

- Streams operate primarily on an on-demand, or **lazy**, evaluation model.
- **Pull over Push**: Items are not forcefully pushed through the entire pipeline simultaneously. Instead, the terminal operation at the end of the pipeline "pulls" items through one-by-one as requested.
- Because an item will often travel the length of the pipeline individually, it allows for processing of potentially **infinite** streams.

> [!NOTE]
> Think of a stream pipeline less like a water hose constantly blasting water (pushing) and more like drinking from a straw (pulling). Liquid travels up the straw only when you actively draw breath at the very end. 

### 1.2 Pure Functions

- Operations passed into stream methods (like `map` or `filter`) should be **pure** functions.
- **No Mutation**: The function should never modify the input data or any external state. Instead, it must represent changed state by creating entirely new objects.
- **Deterministic**: The output must depend entirely and solely on the provided arguments.

```java
// Anti-pattern: Impure function breaking rules
List<String> userNames = List.of("alice", "bob");
List<String> upperNames = new ArrayList<>();
// Avoid modifying external state (`upperNames`) inside the stream
userNames.stream().forEach(name -> upperNames.add(name.toUpperCase()));

// Idiomatic: Pure function
List<String> pureUpperNames = userNames.stream()
    .map(String::toUpperCase) // Creates a new string, modifies nothing else
    .collect(Collectors.toList());
```

---

## 2. The Map Operation

### 2.1 Core Behavior

- The `map` operation applies a given function to each item in the upstream sequence and puts the potentially transformed result into a new downstream representation.
- The downstream result stream will always have a strict **one-to-one** correspondence; the output count exactly equals the input count. 

> [!NOTE]
> A `map` operation is equivalent to a simple `for` loop processing items exactly one at a time. It touches every element and returns exactly an equivalent amount of elements.

### 2.2 Type Signatures

- The function type uses bounds to handle polymorphism gracefully: `Function<? super A, ? extends R>`.
- The `? super A` ensures you can pass a function expecting a superclass (e.g., `Object`) when the stream contains a subclass type.
- The `? extends R` ensures the returned type will smoothly assign into the downstream generic type.

```java
// `map` perfectly translates one form (String) to another (Integer)
List<String> ids = List.of("101", "102", "103");

List<Integer> parsedIds = ids.stream()
    .map(Integer::parseInt) // Emits exactly one element for every input
    .collect(Collectors.toList()); 
```

---

## 3. The Filter Operation

### 3.1 Core Behavior

- The `filter` operation evaluates each item against an expected condition or rule. 
- The items themselves remain fundamentally **unchanged**. `filter` does not alter values.
- The number of emitted items will be **equal to or less than** the incoming count, depending on how many pass the test.

> [!NOTE]
> A `filter` is fundamentally a standard `for` loop wrapped around a conditional `continue` statement. It simply skips elements that fail to meet criteria and proceeds with those that pass.

### 3.2 Type Signatures

- Takes a `Predicate<? super A>`.
- Like `map`, the generic wildcard means a Stream of subclasses can safely use a Predicate designed strictly for their parent class. 

```java
List<Integer> rawScores = List.of(85, 42, 99, 70);

// Only items fulfilling the Predicate proceed down the pipeline
List<Integer> passingScores = rawScores.stream()
    .filter(score -> score >= 75) 
    .collect(Collectors.toList());
// Output Count: 2 (85, 99)
```

---

## 4. The FlatMap Operation

### 4.1 Core Behavior

- Used when a single input item is meant to result in an **arbitrary** number of output items (zero, one, or many).
- Strips away internal boundaries to "flatten" streams. If each item produces its own smaller stream of lists or sequences, `flatMap` concatenates them directly together into one contiguous sequence.

> [!NOTE]
> Think of `flatMap` like opening multiple boxes (the input items); each box may contain varying amounts of toys (the output items). Instead of maintaining arrays of boxes holding toys, you just dump all toys recursively onto the floor in one giant pile. Programmatically, it works identically to **nested loops**.

### 4.2 Type Signatures

- Takes a `Function<? super A, Stream<? extends R>>`.
- Crucially, the function must return a `Stream` of items, not a single underlying object or Collection wrapper, so it can be unrolled logically.

```java
List<List<String>> organization = List.of(
    List.of("Alice", "Bob"),
    List.of("Charlie"),
    List.of("Dave", "Eve", "Frank")
);

// Without FlatMap, you get streams of streams.
// With FlatMap, the lists dissolve into a single contiguous pool of employees.
List<String> unifiedRoster = organization.stream()
    .flatMap(List::stream) // Unrolls each internal List directly into the main Stream
    .collect(Collectors.toList());
// Output Count: 6 independent Strings
```
