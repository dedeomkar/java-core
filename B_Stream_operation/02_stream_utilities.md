# Stream Utilities

## Table of Contents
- [1. Primitive Streams and Conversions](#1-primitive-streams-and-conversions)
  - [1.1 Specialized Primitive Streams](#11-specialized-primitive-streams)
  - [1.2 Boxing and Unboxing Operations](#12-boxing-and-unboxing-operations)
- [2. Stream Factory Methods](#2-stream-factory-methods)
  - [2.1 Core Interface Factories](#21-core-interface-factories)
  - [2.2 Additional Stream Sources](#22-additional-stream-sources)
  - [2.3 Range Generation](#23-range-generation)
- [3. Intermediate Stream Operations](#3-intermediate-stream-operations)
  - [3.1 Deduplication and Sorting](#31-deduplication-and-sorting)
  - [3.2 Slicing and Filtering](#32-slicing-and-filtering)
- [4. The Spliterator](#4-the-spliterator)
  - [4.1 Splitting and Parallelism](#41-splitting-and-parallelism)
  - [4.2 Spliterator Usage](#42-spliterator-usage)

---

## 1. Primitive Streams and Conversions

### 1.1 Specialized Primitive Streams

Boxing and unboxing primitives is inefficient. To mitigate this, the Stream API provides three specialized variations that handle primitive data directly:
- `IntStream`
- `LongStream`
- `DoubleStream`

These data types mirror the precise primitive structures found in `java.util.function`.

> [!NOTE]
> Think of primitive streams as a dedicated highway lane strictly for motorbikes (primitives), avoiding the overhead of converting them to cars (objects) just to fit in the regular lanes.

### 1.2 Boxing and Unboxing Operations

Primitive and object streams provide robust conversion mechanisms:

- **Object to Primitive**: `mapToInt`, `mapToLong`, `mapToDouble`, `flatMapToInt`, etc.
- **Primitive to Object**: `mapToObj`, `flatMapToObj`, and the `boxed()` method.

> [!WARNING]
> Be highly mindful of implicit boxing. Using `map(String::length)` on a stream of strings requires boxing the integer length returned to go downstream as an object stream. Using `mapToInt(String::length)` avoids this penalty.

```java
// Example: Avoiding Boxing overhead
IntStream lengthStream = Stream.of("Fred", "Jim", "Sheila")
                               .mapToInt(String::length); // Maps to primitive int stream

Stream<Integer> boxedStream = lengthStream.boxed(); // Back to an Object Stream 
```

---

## 2. Stream Factory Methods

### 2.1 Core Interface Factories

The core stream interfaces provide multiple native factories for streamlined generation:

- `Stream.empty()`: Creates a completely empty stream.
- `Stream.of(args...)`: Constructs a stream from passed arguments (varargs).
- `Stream.ofNullable(arg)`: Yields a single-element stream, or empty if `null`.
- `Stream.concat(s1, s2)`: Connects two discrete streams seamlessly, maintaining lazy processing.
- `Stream.generate(Supplier)`: Constructs an *infinite* stream repeatedly evaluating a supplier.
- `Stream.iterate(seed, UnaryOperator)`: Calculates subsequent values recursively. An overloaded variant `iterate(seed, Predicate, UnaryOperator)` supports a specific cutoff condition.
- `Stream.builder()`: Yields an object that acts as a `Consumer` to accumulate items manually into a stream before locking it down.

### 2.2 Additional Stream Sources

Arrays and standard JDK utilities supply numerous alternative sources:
- `Arrays.stream(array)` and overloads for arrays.
- `Collection.stream()` and `Collection.parallelStream()` for common data structures.
- `StreamSupport.stream(Spliterator, boolean)` to draw from custom splits.
- `Pattern.splitAsStream(CharSequence)` to tokenize textual input without intermediate allocations.
- `Random.ints()`, `Random.doubles()`, `Random.longs()` for streams of pseudo-random occurrences.
- File operations: `Files.find()`, `Files.walk()`, `Files.list()`, and `Files.lines()`.

```java
// Sourcing from a Map
Map<Integer, String> employeeMap = new HashMap<>();
// A Map itself is not a Collection, you must extract its entries:
Stream<Map.Entry<Integer, String>> entryStream = employeeMap.entrySet().stream();

// Sourcing from text
Pattern delimiter = Pattern.compile(":");
Stream<String> words = delimiter.splitAsStream("High:Throughput:Low:Latency");
```

### 2.3 Range Generation

`IntStream` and `LongStream` establish specific factories to populate monotonically increasing series, crucial for numeric algorithms:
- `range(start, end)`: A strict fence logic (e.g., `< end`).
- `rangeClosed(start, end)`: Inclusive loop sequence (e.g., `<= end`).

**Example: Creating a stream of odd numbers (1, 3, 5, 7 ...) up to 1000**
There are multiple creative ways to generate this sequence leveraging different factory methods:

```java
// 1. Use iterate with a predicate (Java 9+)
// Parameters: (seed=1, hasNext predicate: v < 1000, next operator: v + 2)
IntStream.iterate(1, v -> v < 1000, v -> v + 2)

// 2. Use iterate without predicate, capped by limit
// Parameters: (seed=1, next operator: v + 2), followed by a limit of 500 items
IntStream.iterate(1, v -> v + 2).limit(500)

// 3. Use rangeClosed and filter
// Parameters: (startInclusive=1, endInclusive=999), followed by an odd-number filter
IntStream.rangeClosed(1, 999).filter(v -> v % 2 != 0)

// 4. Use strictly range fence and filter
IntStream.range(1, 1000).filter(v -> v % 2 != 0)

// 5. Use length-based range and mathematical mapping
IntStream.range(0, 500).map(v -> v * 2 + 1)
```

---

## 3. Intermediate Stream Operations

### 3.1 Deduplication and Sorting

- `distinct()`: Tracks all passing items continuously and denies duplicates from proceeding downstream.
- `sorted()` & `sorted(Comparator)`: Orders sequences organically or per explicit definition. 

> [!IMPORTANT]
> The `sorted` operation is a highly **stateful barrier**. Since deciding an absolute order requires full sequence awareness, no element can proceed past `sorted()` until the *entirety* of incoming data is scanned. Avoid this logic if an infinite stream structure is present.

### 3.2 Slicing and Filtering

- `limit(n)`: Sets a strict cut-off to halt stream digestion after `n` elements.
- `skip(n)`: Bypasses the initial elements safely ignoring them entirely.
- `takeWhile(Predicate)`: Lets iterations continue exclusively until the predicate registers a single `false` evaluation. The stream then terminates completely.
- `dropWhile(Predicate)`: Evaluates iteratively; initially rejects data while `true`, seamlessly routing everything remaining downstream the moment a `false` triggers.

---

## 4. The Spliterator

### 4.1 Splitting and Parallelism

The `Spliterator` (splitable iterator) provides low-level structure enabling parallel execution chunks. Instead of holding all data strictly in-memory, it grants threaded access to "lumps" of lazily-sourced inputs. 

The `trySplit()` interface method attempts to sever the active segment directly in half. The active pointer receives a truncated front segment, and a strictly new Spliterator encompasses the remaining back segment. Successive calls further slice those portions.

> [!NOTE]
> Think of spliterators like splitting a pizza iteratively among friends. You slice it in half continuously, handing chunks to separate threads (friends) who consume them simultaneously.

### 4.2 Spliterator Usage

Streams and parallelism operate natively over Spliterators. Individual segment blocks process elements actively by invoking `tryAdvance(Consumer)`.

```java
import java.util.Spliterator;
import java.util.stream.IntStream;

public class ExampleSpliterator {
    public static void main(String[] args) {
        // Initial spliterator for range 0 to 999
        Spliterator.OfInt sp1 = IntStream.range(0, 1000).spliterator();
        
        // Element '0' is processed. sp1 now covers 1 to 999.
        sp1.tryAdvance((int x) -> System.out.println("sp1 before: " + x)); 

        // Split 1: sp2 gets the front half (~1 to 500), sp1 keeps the back half (~501 to 999)
        var sp2 = sp1.trySplit();
        
        sp1.tryAdvance((int x) -> System.out.println("sp1: " + x)); // Prints roughly 501
        sp2.tryAdvance((int x) -> System.out.println("sp2: " + x)); // Prints 1

        // Split 2: Further subdividing the existing chunks
        var sp3 = sp1.trySplit(); // sp3 gets ~502 to 750, sp1 keeps ~751 to 999
        var sp4 = sp2.trySplit(); // sp4 gets ~2 to 250, sp2 keeps ~251 to 500
        
        sp1.tryAdvance((int x) -> System.out.println("sp1: " + x)); // Prints roughly 751
        sp2.tryAdvance((int x) -> System.out.println("sp2: " + x)); // Prints roughly 251
        sp3.tryAdvance((int x) -> System.out.println("sp3: " + x)); // Prints roughly 502
        sp4.tryAdvance((int x) -> System.out.println("sp4: " + x)); // Prints 2
    }
}
```

> [!TIP]
> Notice how every call to `trySplit()` cleanly delegates the "front" of the remaining data to the newly returned Spliterator, while the original Spliterator retains the back portion. This ensures deterministic, non-overlapping workloads for parallel processing.
