# Methods of Deque and Map

## Table of Contents
- [1. The Queue Interface](#1-the-queue-interface)
  - [1.1 Insertion and Retrieval](#11-insertion-and-retrieval)
  - [1.2 Success vs Failure Handling](#12-success-vs-failure-handling)
- [2. The Deque Interface](#2-the-deque-interface)
  - [2.1 Double-Ended Operations](#21-double-ended-operations)
  - [2.2 Stack Semantics (LIFO)](#22-stack-semantics-lifo)
  - [2.3 Specialized Deque Methods](#23-specialized-deque-methods)
- [3. The Map Interface](#3-the-map-interface)
  - [3.1 Structure and Core API](#31-structure-and-core-api)
  - [3.2 Map Views and Factories](#32-map-views-and-factories)
- [4. Advanced Map Operations](#4-advanced-map-operations)
  - [4.1 The Merge Method](#41-the-merge-method)
  - [4.2 Compute Methods](#42-compute-methods)
  - [4.3 Immutability Gotchas](#43-immutability-gotchas)

---

## 1. The Queue Interface

### 1.1 Insertion and Retrieval

- **Purpose**: Designed for holding elements prior to processing.
- **Ordering**: Typically FIFO (First-In, First-Out), though priority queues can order by natural order or comparator.
- **Endpoint Access**: Items are inserted at one end (tail) and removed from the other (head).

> [!NOTE]
> Think of a Queue like a **coffee shop line**. Customers enter at the back and are served (removed) from the front. If you just want to see who is next without serving them, you're "peeking" at the front of the line.

### 1.2 Success vs Failure Handling

Java's Queue API provides two forms for each major operation: one that throws an exception and one that returns a sentinel value (`null` or `false`).

| Operation Type | Throws Exception | Returns Sentinel |
| :--- | :--- | :--- |
| **Insert** | `add(e)` | `offer(e)` |
| **Remove** | `remove()` | `poll()` |
| **Examine** | `element()` | `peek()` |

```java
// Choice: ArrayDeque is typically preferred for performance.
// LinkedList is used here as a common pedagogical example.
Queue<String> messages = new ArrayDeque<>(); 

// Safe insertion
boolean accepted = messages.offer("Incoming FIX Message"); 

// Safe retrieval - returns null if empty
String nextMsg = messages.poll(); 
```

> [!IMPORTANT]
> **Performance Note: LinkedList vs. ArrayDeque**
> In high-throughput, low-latency systems, **`ArrayDeque`** is generally superior to `LinkedList` for `Queue` operations:
> - **Memory Overhead**: `LinkedList` allocates a new `Node` object for every element, incurring garbage collection pressure.
> - **Cache Locality**: `ArrayDeque` uses a contiguous array, which is significantly more CPU-cache friendly.
> - **Null Handling**: `ArrayDeque` **forbids null elements**, while `LinkedList` allows them.


---

## 2. The Deque Interface

### 2.1 Double-Ended Operations

- **Definition**: A "Double Ended Queue" (pronounced "deck").
- **Capabilities**: Supports element insertion and removal at both ends.
- **Hierarchy**: Extends `Queue`, meaning all Queue methods are available.

> [!TIP]
> Think of a Deque like a **deck of cards** on a table. You can add or remove cards from the top or the bottom. It's essentially a two-way street for data.

#### Head vs Tail Access

| endpoint | Operation | Exception | Sentinel |
| :--- | :--- | :--- | :--- |
| **Head (Front)** | Insert | `addFirst(e)` | `offerFirst(e)` |
| | Remove | `removeFirst()` | `pollFirst()` |
| | Examine | `getFirst()` | `peekFirst()` |
| **Tail (Back)** | Insert | `addLast(e)` | `offerLast(e)` |
| | Remove | `removeLast()` | `pollLast()` |
| | Examine | `getLast()` | `peekLast()` |

### 2.2 Stack Semantics (LIFO)

Deques should be used in preference to the legacy `Stack` class for LIFO (Last-In, First-Out) operations.

#### Stack to Deque Mapping
| Stack Method | Deque Equivalent | Behavior on Empty |
| :--- | :--- | :--- |
| `push(e)` | `addFirst(e)` | Throws Exception (if full) |
| `pop()` | `removeFirst()` | Throws `NoSuchElementException` |
| `peek()` | `peekFirst()` | Returns `null` |

```java
// Rule of Thumb: Never use the legacy 'Stack' class (synchronized overhead).
Deque<String> stack = new ArrayDeque<>();

// LIFO Operations (Push/Pop)
stack.push("Base Layer");
stack.push("Top Layer");

System.out.println(stack.pop());  // "Top Layer"
```

> [!TIP]
> Use **`ArrayDeque`** for stacks because it is faster than `Stack` (no synchronization) and `LinkedList` (no node allocation). It behaves as a circular buffer, making it very efficient for both ends.

### 2.3 Specialized Deque Methods

- **Occurrence Removal**: `removeFirstOccurrence(Object)` and `removeLastOccurrence(Object)`.
- **Iteration**: `descendingIterator()` allows traversing the Deque from tail to head.
- **Contract Note**: Unlike `List` or `Set`, `Queue` and `Deque` are **not** expected to provide meaningful `equals()` or `hashCode()` implementations.


---

## 3. The Map Interface

### 3.1 Structure and Core API

- **Model**: Represents a mapping between unique keys and values (Associative Array).
- **Lookup**: Optimized for high-speed retrieval via keys.
- **Hierarchy**: Not a child of `Collection` or `Iterable`.

> [!NOTE]
> Think of a Map like a **coat check system**. You give a unique tag (Key) and get back a specific coat (Value). The tag is yours and unique; if two people had the same tag, the system would break.

#### Common Operations
- **Return Values**: Most modification methods (like `put` or `remove`) return the **previous value** associated with the key, or `null`.
- **`putIfAbsent(K, V)`**: Only inserts if the key is missing or mapped to `null`.
- **`getOrDefault(key, default)`**: Prevents `null` handling by providing a fallback.

```java
Map<String, Double> stockPrices = new HashMap<>();

// Standard update
Double oldPrice = stockPrices.put("AAPL", 150.0);

// Conditional update
stockPrices.putIfAbsent("GOOG", 2800.0);

// Safe retrieval
Double price = stockPrices.getOrDefault("TSLA", 0.0);
```

### 3.2 Map Views and Factories

Maps provide collection views to allow iteration:
1. **`keySet()`**: A `Set` of all unique keys.
2. **`values()`**: A `Collection` of all values (may contain duplicates).
3. **`entrySet()`**: A `Set` of `Map.Entry<K, V>` objects (the "tuples").

#### Unmodifiable Factories (Java 9+)
- `Map.of(k1, v1, ...)`: Direct enumeration for up to 10 pairs.
- `Map.ofEntries(entry(k1, v1), ...)`: Arbitrary number of entries using `Map.entry()`.

> [!WARNING]
> Maps created via `Map.of()` and `Map.copyOf()` are **strictly unmodifiable**. Attempting to `put` or `remove` will result in an `UnsupportedOperationException`.


---

## 4. Advanced Map Operations

### 4.1 The Merge Method

The `merge(key, value, BiFunction)` method is a powerful tool for cumulative updates.

**Logic Flow**:
1. Look up the value for `key`.
2. If `null`, insert the provided `value`.
3. If not `null`, apply the `BiFunction` to (existingValue, newValue).
4. If the function result is `null`, remove the entry. Otherwise, update with the result.

```java
Map<String, String> sessionData = new HashMap<>();
sessionData.put("userID", "user_123");

// Appending more data to an existing key
sessionData.merge("userID", "_session_A", (oldVal, newVal) -> oldVal + newVal);

System.out.println(sessionData.get("userID")); // user_123_session_A
```

### 4.2 Compute Methods

- **`compute(key, remappingFunction)`**: Always computes a value.
- **`computeIfPresent(key, remappingFunction)`**: Only computes if the key currently exists.
- **`computeIfAbsent(key, mappingFunction)`**: Only computes if the key is missing (useful for lazy initialization of nested collections).

```java
Map<String, List<String>> treeMap = new HashMap<>();

// Lazy-creating a list if it doesn't exist
treeMap.computeIfAbsent("fruits", k -> new ArrayList<>()).add("Apple");
```

### 4.3 Immutability Gotchas

A common exam trick involves trying to modify a map returned by a factory.

```java
// Logic error: Map.of returns an unmodifiable map
Map<String, Integer> scores = Map.of("Alice", 10);

try {
    scores.compute("Alice", (k, v) -> v + 5); 
} catch (UnsupportedOperationException e) {
    System.out.println("Cannot modify unmodifiable map!");
}

// Workaround: Wrap in a new HashMap
Map<String, Integer> modifiableScores = new HashMap<>(Map.of("Bob", 20));
modifiableScores.merge("Bob", 5, Integer::sum);
```

