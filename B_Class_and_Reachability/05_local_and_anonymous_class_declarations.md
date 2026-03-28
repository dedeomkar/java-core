# Local and Anonymous Class Declarations

## Table of Contents
- [1. Local Classes](#1-local-classes)
- [2. Closures and Effectively Final Variables](#2-closures-and-effectively-final-variables)
- [3. Anonymous Classes](#3-anonymous-classes)
- [4. Anonymous Classes vs. Lambda Expressions](#4-anonymous-classes-vs-lambda-expressions)

---

## 1. Local Classes

### Concept Definition
Java allows classes to be declared entirely inside a block of code (such as inside a method). These are called **local classes**. 

Because they are local to that specific block of code:
- Access control modifiers (`public`, `protected`, `private`) are **not permitted**.
- The `static` keyword is **not permitted**.
- They behave as either instance or static classes depending entirely on whether the enclosing block has an instance context (has a `this` reference) or a static context.
- Following Java 11, both local **enums** and local **interfaces** are strictly prohibited.

### Java-Specific Implementation
```java
public class Processor {
    public void processItems() {
        // A complete class declared inside the method block
        class LocalTask {
            public void run() {
                System.out.println("Running local task.");
            }
        }
        
        LocalTask task = new LocalTask();
        task.run();
    }
}
```

> **Important Considerations**: A local class is only visible and understandable within the curly braces of the method that declared it. Attempting to use a local class as the return type of the method will fail to compile because the caller has no idea what that class is. (However, returning it as a generic `Object` or a known `Interface` is completely valid).

[Back to Top](#table-of-contents)

---

## 2. Closures and Effectively Final Variables

### Concept Definition
A local class might need to refer to a local variable defined in the enclosing method. This behavior is called creating a **closure**. 

However, this is only permitted if the local variable is **final** or **effectively final**. 
- **Final**: Explicitly marked with the `final` keyword.
- **Effectively Final**: The variable is definitely initialized once and never reassigned anywhere in the method. 

### Everyday Analogy
Think of a closure like taking a Polaroid photo of your surroundings. When the local class is created, it takes a "snapshot" of the method's variables at that exact moment. If the method were allowed to keep changing the variable later, the snapshot would no longer match reality. Therefore, Java enforces that the variable must never change after compilation (must be effectively final).

### Java-Specific Implementation
```java
public void attemptClosure() {
    int count = 10; // This is 'effectively final' as long as we never change it again

    class ClosureTask {
        public void execute() {
            // Legal: 'count' has not been modified
            System.out.println("Count is: " + count); 
        }
    }
    
    // IF we uncomment the next line, 'count' is no longer effectively final,
    // and the code inside ClosureTask will instantly fail to compile!
    // count = 11; 
}
```

> **Best Practice/Consideration**: Keep in mind that for reference types (objects), "effectively final" means the *reference* cannot be reassigned to a new object. It does **not** mean the object itself cannot mutate its internal state!

[Back to Top](#table-of-contents)

---

## 3. Anonymous Classes

### Concept Definition
If a class is to be declared and instantiated exactly once, at the exact same location in the code, no class name is truly necessary. This is known as an **anonymous class**. 

Instead of naming the class, you specify a single generalized **parent type** (an interface, an abstract class, or a concrete class) that you are implementing or extending.

### Java-Specific Implementation
```java
public void executeWorkflow() {
    // Declaring and instantiating a Runnable on the fly
    Runnable r = new Runnable() {
        @Override
        public void run() {
            System.out.println("Anonymous Runnable executed!");
        }
    };
    r.run();
}
```

### Passing Constructor Arguments
Unlike interfaces, if an anonymous class is subclassing an abstract or concrete class, you can actually pass arguments directly to the parent class's constructor!

```java
abstract class CustomTask {
    protected String message;
    public CustomTask(String message) {
        this.message = message;
    }
    public abstract void doWork();
}

public void setup() {
    // Passing "Hello!" up to the parent constructor
    CustomTask task = new CustomTask("Hello!") {
        @Override
        public void doWork() {
            System.out.println("Working with message: " + message);
        }
    };
    task.doWork();
}
```

[Back to Top](#table-of-contents)

---

## 4. Anonymous Classes vs. Lambda Expressions

### Concept Definition
While modern Java heavily relies on Lambda expressions for concise logic, there are unique capabilities anonymous inner classes possess that lambdas simply cannot do.

### Core Differences
1. **Parent Types**: Lambda expressions can *only* implement interfaces. Anonymous classes can subclass concrete or abstract classes.
2. **Constructors**: Because they can subclass abstract/concrete classes, anonymous classes can pass initialization arguments to parent class constructors.
3. **Multiple Methods**: Anonymous inner classes can implement interfaces that have multiple abstract methods. A lambda expression is strictly bound to "functional interfaces" (interfaces with exactly one abstract method).
4. **Custom Additions**: Anonymous inner classes can define completely new fields, nested types, and override multiple methods.
5. **The "this" Pointer**: 
   - Inside an anonymous class, the `this` keyword refers to the instance of the anonymous class itself.
   - Inside a Lambda expression, there is no distinct `this`! Using the word `this` inside a lambda refers transparently to the enclosing outer object.

[Back to Top](#table-of-contents)
