# Lambda Expressions

## Notes

# 📘 Complete Java 8 Lambda Expressions Reference

### 1️⃣ Overview
* **🧠 Theory:** A **Lambda Expression** is essentially an anonymous function—a block of code that can be passed around to execute later. It brings **Functional Programming** concepts to Java.
* **The Goal:** To treat code (behavior) as data. Before Java 8, if you wanted to pass behavior, you had to wrap it in an object (usually an Anonymous Inner Class). Lambdas remove this boilerplate.

### 2️⃣ Syntax (The Arrow `->`)
* **🧠 Theory:** A lambda consists of three parts:
  1. **Parameters:** The inputs.
  2. **The Arrow Token:** `->` separates parameters from the body.
  3. **The Body:** The code to execute.



* **💻 Code (Evolution):**
  ```java
  // 1. Pre-Java 8 (Anonymous Inner Class)
  Runnable r1 = new Runnable() {
      @Override
      public void run() {
          System.out.println("Old Way");
      }
  };

  // 2. Lambda (Verbose)
  Runnable r2 = () -> { System.out.println("Lambda Way"); };

  // 3. Lambda (Concise - No braces for single line)
  Runnable r3 = () -> System.out.println("Short Lambda");

```

### 3️⃣ Functional Interface (The Requirement)

* **🧠 Theory:** You cannot just create a lambda out of nowhere. It **must** match a **Functional Interface**.
* **Definition:** A Functional Interface is an interface with **exactly one abstract method** (SAM - Single Abstract Method).
* **The Annotation:** `@FunctionalInterface` is optional but recommended. It forces the compiler to throw an error if you accidentally add a second abstract method.
* **💻 Code:**
```java
@FunctionalInterface
interface MathOperation {
    int operate(int a, int b); // Only one abstract method allowed
}

// Usage
MathOperation add = (a, b) -> a + b;
MathOperation sub = (a, b) -> a - b;

```



### 4️⃣ Type Inference

* **🧠 Theory:** You rarely need to specify types (e.g., `(int a, int b)`). The Java compiler infers the types from the context (the Functional Interface definition).
* **💻 Code:**
```java
// Compiler knows 's' must be a String because Consumer<String> expects it
Consumer<String> printer = s -> System.out.println(s);

```



### 5️⃣ Variable Capture (Closures)

* **🧠 Theory:** Lambdas can access variables defined outside their body (in the enclosing scope). However, there is a strict rule: **Local variables used inside a lambda must be final or effectively final.**
* **Effectively Final:** A variable that is not explicitly marked `final` but is never reassigned after initialization.
* **💻 Code:**
```java
int factor = 10; // "Effectively final"

// ✅ Valid
Function<Integer, Integer> multiplier = n -> n * factor;

// ❌ INVALID: Modifying the variable breaks "effectively final"
// factor = 20; 

```



### 6️⃣ Method References (`::`)

* **🧠 Theory:** If a lambda does nothing but call an existing method, you can replace the lambda with a **Method Reference**. It makes code cleaner.
* **Types:**
1. **Static Method:** `ClassName::methodName`
2. **Instance Method of Object:** `instance::methodName`
3. **Instance Method of Class:** `ClassName::methodName`
4. **Constructor:** `ClassName::new`


* **💻 Code:**
```java
List<String> names = Arrays.asList("Alice", "Bob");

// Lambda
names.forEach(s -> System.out.println(s));

// Method Reference (Cleaner)
names.forEach(System.out::println);

```



---

## 🌟 Senior & Lead Developer Addendum (JVM Internals)

### 7️⃣ `this` Reference: Lambda vs Anonymous Class

* **🧠 Theory:** This is a classic interview "gotcha."
* **Anonymous Inner Class:** The keyword `this` refers to the **inner class instance itself**.
* **Lambda:** The keyword `this` refers to the **enclosing class instance**. A lambda does not define its own scope for `this`.


* **💻 Code:**
```java
public class ScopeTest {
    public void test() {
        // Anonymous Class
        new Runnable() {
            public void run() {
                System.out.println(this); // Prints: ScopeTest$1 (The inner class)
            }
        }.run();

        // Lambda
        Runnable r = () -> System.out.println(this); // Prints: ScopeTest (The main class)
        r.run();
    }
}

```



### 8️⃣ Compilation: `invokedynamic` (No Inner Classes!)

* **🧠 Theory:** Many developers think compilation creates a `.class` file for every lambda (like `MyClass$1.class`). **This is wrong.**
* **The Truth:** Java 8 uses the `invokedynamic` bytecode instruction (introduced in Java 7).
* When the compiler sees a lambda, it does **not** generate a class file at compile time.
* Instead, it generates a recipe via `LambdaMetafactory`.
* **At Runtime**, the JVM spins up a lightweight class on the fly.


* **Benefit:** significantly reduces the JAR file size and memory footprint compared to anonymous inner classes.

### 9️⃣ Performance Overhead

* **🧠 Theory:** Are lambdas slower?
* **Startup:** Slightly slower on the *very first* execution because the JVM has to link the call site (spin the class).
* **Runtime:** After linking, they are just as fast as standard method calls (often faster than anonymous classes because they don't have the overhead of a full object instantiation loop).



### 🔟 Serialization Warning

* **🧠 Theory:** You *can* make a lambda Serializable (by casting it: `(Runnable & Serializable) () -> ...`), but you **should never do this** in production.
* **Reason:** The serialized form of a lambda depends on the underlying JVM implementation. A lambda serialized on OpenJDK 11 might crash when deserialized on Oracle JDK 17. Lambdas are for *behavior*, not *state*.

```

---

**Would you like to move on to the built-in Functional Interfaces (`Predicate`, `Consumer`, `Supplier`, `Function`)? That is usually the next logical step in mastering Java 8.**

```