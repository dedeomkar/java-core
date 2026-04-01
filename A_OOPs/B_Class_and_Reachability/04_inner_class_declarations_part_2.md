# Inner Class Declarations (Part 2)

## Table of Contents
- [1. Implementing the Iterator Pattern](#1-implementing-the-iterator-pattern)
- [2. Constructing Inner Instances](#2-constructing-inner-instances)
- [3. Implicit Arguments under the Hood](#3-implicit-arguments-under-the-hood)
- [4. The Qualified "this" Rule](#4-the-qualified-this-rule)

---

## 1. Implementing the Iterator Pattern

### Concept Definition
Using an **instance inner class**, we can securely and effectively implement the `Iterable` / `Iterator` structural design pattern. Building a local custom list involves creating an inner class solely focused on tracking iteration progress.

### Java-Specific Implementation
```java
// A simple enclosing custom list implementation
public class MyList<E> implements Iterable<E> {
    private E[] data;
    private int count = 0;

    // ... array initialization and add() omitted ...

    @Override
    public Iterator<E> iterator() {
        // Return a fresh Inner class instance for every caller
        return new MyIterator(); 
    }

    // 1. Private: Outside clients only interact with the Iterator interface
    // 2. Instance Inner Class: Needs access to MyList's 'data' array and 'count' integer 
    private class MyIterator implements Iterator<E> {
        private int progress = 0;

        @Override
        public boolean hasNext() {
            // 'progress' is the inner field. 'count' is implicitly the outer field.
            return progress < count;
        }

        @Override
        public E next() {
            // 'data' is implicitly the outer field.
            return data[progress++];
        }
    }
}
```

> **Important Considerations**: 
> Notice that `count` and `data` are seamlessly referenced by the inner class method as if they actually belonged to it natively.

[Back to Top](#table-of-contents)

---

## 2. Constructing Inner Instances

### Concept Definition
How does the `MyIterator` inner instance in the `iterator()` method actually gain the reference to the `MyList` object? 

If an inner class is instantiated in an instance context of the enclosing class (i.e. inside an instance method), the `this` object from that context is used automatically as the enclosing instance.

### Java-Specific Implementation
Within the `MyList` instance method `iterator()`, writing `return new MyIterator();` acts like syntactic sugar for explicitly passing `this`.

```java
public Iterator<E> iterator() {
    // Explicit syntax for providing the 'this' instance into the Inner constructor 
    return this.new MyIterator();
}
```

### Best Practice/Consideration
If you need to construct the inner class from an entirely different static context (like a `main` method), you **must** supply a specific object reference. The syntax is:
`instanceOfOuter.new InnerClass()`

```java
Outer outer = new Outer();

// Instantiating Inner requires calling 'new' directly on the specific Outer object
Outer.Inner inner = outer.new Inner();

// Quick inline approach 
Outer.Inner quickInner = new Outer().new Inner();
```

[Back to Top](#table-of-contents)

---

## 3. Implicit Arguments under the Hood

### Concept Definition
The compiler works in the background to inject the connection between the inner class and its enclosing class at the byte-code level. You do not see this in the Java source code.

1. When a `.class` file is generated, an inner class like `MyIterator` inside `MyList` is named `MyList$MyIterator.class`
2. If you disassemble the bytecode, the constructor for `MyIterator` (even if you defined it taking no arguments) will actually take an instance of `MyList` as the **first implicit argument**. 
3. When `new MyIterator()` runs, the compiler injects the current `this` parameter from the invoking context.

[Back to Top](#table-of-contents)

---

## 4. The Qualified "this" Rule

### Concept Definition
In general, you don't need to specify the enclosing instance. But there are rare layout scenarios when you must disambiguate members (for example, terrible variable shadowing where both inner class and outer class have a field named exactly the same). 

To explicitly refer to the outer class instance, Java defines a concept known as **Qualified "this"**, which takes the shape of `OuterClassName.this`.

### Java-Specific Implementation
```java
public class Outer {
    private String name = "Outer Jim";

    public class Inner {
        private String name = "Inner Bill";

        public void showNames() {
            // 1. Closest scope (Inner variable)
            System.out.println(this.name); // Prints: Inner Bill
            System.out.println(name);      // Prints: Inner Bill

            // 2. Qualified this (Outer variable)
            System.out.println(Outer.this.name); // Prints: Outer Jim
        }

        public Outer getOuter() {
            // Handing back the enclosing object reference to a caller
            return Outer.this;
        }
    }
}
```

> **Important Considerations**: In Java 11 and later, standard inner classes cannot declare `static` members (methods, fields, types), with one exception: constants. A `static final` field initialized with a constant expression is considered valid. 
> Ex: `static final String constName = "Valid inside Inner";`

[Back to Top](#table-of-contents)
