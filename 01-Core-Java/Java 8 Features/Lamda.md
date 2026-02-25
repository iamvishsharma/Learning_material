# Lambda Expressions

## Notes

# 📘 Complete Java 8 Lambda Expressions Reference

### 1️⃣ Overview
* **🧠 Theory:** A **Lambda Expression** is essentially an anonymous function—a block of code that can be passed around to execute later. It brings **Functional Programming** concepts to Java.
* **The Goal:** To treat code (behavior) as data. Before Java 8, passing behavior required wrapping it in an Anonymous Inner Class object. Lambdas remove this boilerplate.

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

* **🧠 Theory:** You cannot create a lambda out of nowhere; it **must** match a **Functional Interface**.
* **Definition:** An interface with **exactly one abstract method** (SAM - Single Abstract Method).
* **The Annotation:** `@FunctionalInterface` is optional but recommended. It forces the compiler to throw an error if a second abstract method is added.
* **💻 Code:**
```java
@FunctionalInterface
interface MathOperation {
    int operate(int a, int b); // Only one abstract method allowed
}

// Usage: The body {} is mapped to the 'operate' method
MathOperation add = (a, b) -> a + b;
MathOperation sub = (a, b) -> a - b;

```



### 4️⃣ Type Inference

* **🧠 Theory:** You rarely need to specify types (e.g., `(int a, int b)`). The Java compiler infers types from the Functional Interface definition in the context.
* **💻 Code:**
```java
// Compiler knows 's' must be a String because Consumer<String> expects it
Consumer<String> printer = s -> System.out.println(s);

```



### 5️⃣ Variable Capture (Closures)

* **🧠 Theory:** Lambdas can access variables defined outside their body. **Rule:** Local variables used inside a lambda must be **final or effectively final**.
* **Effectively Final:** A variable not marked `final` but never reassigned after initialization.
* **💻 Code:**
```java
int factor = 10; // "Effectively final"

// ✅ Valid
Function<Integer, Integer> multiplier = n -> n * factor;

// ❌ INVALID: Modifying factor would break "effectively final" rule
// factor = 20; 

```



### 6️⃣ Method References (`::`)

* **🧠 Theory:** Shorthand for a lambda that only calls an existing method.
* **Types:**
1. **Static Method:** `Integer::parseInt`
2. **Instance Method of Object:** `System.out::println`
3. **Instance Method of Class:** `String::toUpperCase`
4. **Constructor:** `ArrayList::new`


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

* **🧠 Theory:** * **Anonymous Inner Class:** `this` refers to the **inner class instance**.
* **Lambda:** `this` refers to the **enclosing class instance**. Lambdas don't define their own scope for `this`.


* **💻 Code:**
```java
public class ScopeTest {
    public void test() {
        new Runnable() {
            public void run() {
                System.out.println(this); // Prints: ScopeTest$1
            }
        }.run();

        Runnable r = () -> System.out.println(this); // Prints: ScopeTest
        r.run();
    }
}

```



### 8️⃣ Compilation: `invokedynamic`

* **🧠 Theory:** Lambdas do **not** generate a `.class` file for every expression (unlike Anonymous Classes).
* **The Process:** Java 8 uses `invokedynamic`. The compiler generates a "recipe" via `LambdaMetafactory`. At runtime, the JVM spins up a lightweight class on the fly.
* **Benefit:** Significantly reduces JAR size and memory footprint.

### 9️⃣ Performance Overhead

* **🧠 Theory:** * **Startup:** Slightly slower on the first execution due to call-site linking.
* **Runtime:** After linking, they are as fast as standard method calls and often faster than anonymous classes due to better JIT inlining.



### 🔟 Serialization Warning

* **🧠 Theory:** You *can* make a lambda Serializable, but you **should never do this**.
* **Reason:** The serialized form is implementation-dependent (OpenJDK vs Oracle JDK). A lambda serialized on one might crash on another.

```

---

```