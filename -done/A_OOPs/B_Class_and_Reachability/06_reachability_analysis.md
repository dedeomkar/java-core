# Reachability Analysis

## Table of Contents
- [1. Overview of Garbage Collection](#1-overview-of-garbage-collection)
- [2. The Roots of Reachability](#2-the-roots-of-reachability)
- [3. How Objects Become Unreachable](#3-how-objects-become-unreachable)
- [4. The Edge Case of Return Values](#4-the-edge-case-of-return-values)

---

## 1. Overview of Garbage Collection

### Concept Definition
The fundamental rule governing Java's Garbage Collection (GC) is simple: **if a program could possibly use an object, that object will not be garbage collected.**

At intervals, the JVM performs a "reachability analysis."
- It determines all objects that are reachable (i.e., some active variable or expression can navigate to the object).
- Any object that is **unreachable** is then considered *eligible* for garbage collection.

### Best Practice/Consideration
- **Eligibility vs. Collection**: Understanding that an object being *eligible* for GC does not mean it will be instantly removed from memory. The actual recovery process depends on the specific GC implementation (e.g., G1, ZGC). 
- Note that reachability is independent of the GC algorithm—for instance, the Epsilon GC never collects anything, but reachability rules still apply!
- Reachability analysis has absolutely nothing to do with "reference counting" (a mechanism used in older languages and systems).


---

## 2. The Roots of Reachability

### Concept Definition
Reachability analysis is **transitive** and begins from a specific set of strong roots. 

The roots of reachability are defined as:
1. The **entire live stack** of every live thread (i.e., local variables in currently executing methods).
2. All **static fields** of every loaded class.

### Everyday Analogy
Imagine the memory heap as a giant spider web of objects. 
The active execution threads and static variables are the "anchors" attached to solid rock. The JVM pulls on these anchors. If an object is woven into the web connecting back to the anchor, it is "reachable" and safe. If an object (or a cluster of objects) has detached and floats loosely, the JVM sweeps it away, regardless of whether the floating objects are holding onto each other.


---

## 3. How Objects Become Unreachable

### Concept Definition
An object transitions from reachable to unreachable when the active roots (or the objects connected to the roots) lose their pointers to that object. This happens in two primary ways:

1. **Reassignment**: A reference variable is explicitly pointed to a different object, or explicitly pointed to `null`.
2. **Going Out of Scope**: When a method completes and returns to its caller, the method's "stack frame" is destroyed. All local variables inside that stack frame instantly evaporate, severing any pointers they held to objects on the heap.

### Java-Specific Implementation
```java
public void performWork() {
    // 1. Array is created and is reachable via the local 'data' variable
    byte[] data = new byte[1024]; 
    
    // 2. Reassignment: The 'data' variable now points to null. 
    // The byte array just became unreachable (eligible for GC).
    data = null; 

    // 3. New String is created and is reachable via 'message'
    String message = new String("Hello Heap");
}
// 4. Method ends. The stack frame vanishes. 
// The 'message' variable no longer exists, so the String object becomes unreachable.
```


---

## 4. The Edge Case of Return Values

### Concept Definition
When a method returns, its local stack frame is destroyed, meaning all local variables inside it technically die. However, if the method **returns a reference to an object**, that reference is immediately passed up into the caller's stack frame. 

Thus, an object created in a method does not necessarily become unreachable when the method ends if its reference was yielded back to the caller.

### Java-Specific Implementation
```java
public LocalDate createDate() {
    LocalDate l1 = LocalDate.now(); // Object A created
    return l1;                      // Returning the reference to Object A
}

public void callerMethod() {
    // The caller catches the reference. 
    // Even though createDate() has ended and 'l1' is gone, 
    // Object A is now safely anchored by 'savedDate'.
    LocalDate savedDate = createDate(); 

    // If the caller didn't save the return value, e.g.:
    // createDate(); 
    // Then Object A would instantly become unreachable.
}
```

> **Important Considerations**: 
> Exam questions frequently try to trick you by having a method build an object, assign variables to `null`, but then return a reference that was saved beforehand. Trace the pointer, not just the local variable names!

