# Functional Interface

## Notes

# 📘 Complete Java 8 Functional Interface Reference

### 1️⃣ Overview
* **🧠 Theory:** A **Functional Interface** is an interface that contains **exactly one abstract method**. 
* **The Purpose:** It acts as the "target type" for Lambda Expressions and Method References. Before Java 8, interfaces like `Runnable` and `Callable` were functional interfaces, but we didn't have a name for them.

### 2️⃣ The `@FunctionalInterface` Annotation
* **🧠 Theory:** This annotation is **informative** (not mandatory). However, it is **highly recommended** because it forces the compiler to generate an error if you accidentally add a second abstract method, breaking the contract.
* **💻 Code:**
  ```java
  @FunctionalInterface
  public interface MyInterface {
      void doSomething(); // Only one abstract method allowed
      
      // Default & Static methods are allowed (they have bodies!)
      default void print() { System.out.println("Default"); }
      static void log() { System.out.println("Static"); }
  }

```

### 3️⃣ The "Big 4" Built-in Interfaces (`java.util.function`)

* **🧠 Theory:** Java 8 introduced a standard library of functional interfaces so you don't have to write your own for every single lambda.

#### A. Predicate `<T>` (The Boolean Logic)

* **Role:** Takes one argument, returns a **boolean**.
* **Method:** `boolean test(T t)`
* **Usage:** Filtering streams, conditional checks.
* **💻 Code:**
```java
Predicate<String> isLong = s -> s.length() > 5;
System.out.println(isLong.test("Hello World")); // true

// Chaining (Default Methods)
Predicate<String> startsWithA = s -> s.startsWith("A");
Predicate<String> complex = isLong.and(startsWithA).negate(); 

```



#### B. Consumer `<T>` (The Action)

* **Role:** Takes one argument, returns **void** (performs a side-effect).
* **Method:** `void accept(T t)`
* **Usage:** Printing, saving to DB, sending an email.
* **💻 Code:**
```java
Consumer<String> printer = s -> System.out.println(s);
printer.accept("Hello"); // Output: Hello

// Chaining
Consumer<String> logger = s -> log.info(s);
printer.andThen(logger).accept("Error"); 

```



#### C. Function `<T, R>` (The Transformer)

* **Role:** Takes one argument of type `T`, returns a result of type `R`.
* **Method:** `R apply(T t)`
* **Usage:** Mapping objects (e.g., `String` to `Integer`), data transformation.
* **💻 Code:**
```java
Function<String, Integer> lengthMapper = s -> s.length();
Integer len = lengthMapper.apply("Java"); // 4

// Chaining (compose vs andThen)
Function<Integer, Integer> square = i -> i * i;
// First length(), then square()
lengthMapper.andThen(square).apply("Java"); // 16

```



#### D. Supplier `<T>` (The Factory)

* **Role:** Takes **no arguments**, returns a result of type `T`.
* **Method:** `T get()`
* **Usage:** Lazy generation of values, factories.
* **💻 Code:**
```java
Supplier<Double> random = () -> Math.random();
System.out.println(random.get());

```



### 4️⃣ Primitive Specializations (Performance Optimization)

* **🧠 Theory:** The standard generic interfaces (`Function<Integer, Integer>`) force **Autoboxing** (converting `int` ↔ `Integer`), which kills performance in high-frequency loops.
* **Solution:** Java provides specialized interfaces for `int`, `long`, and `double` to avoid this overhead.
* **Examples:**
* `IntPredicate` (takes `int`, returns `boolean`)
* `LongConsumer` (takes `long`, returns `void`)
* `DoubleFunction<R>` (takes `double`, returns `R`)


* **💻 Code:**
```java
// ❌ BAD (Boxing overhead)
Predicate<Integer> p = i -> i > 10; 

// ✅ GOOD (No boxing)
IntPredicate ip = i -> i > 10; 

```



### 5️⃣ Bi-Functional Interfaces (Two Arguments)

* **🧠 Theory:** Sometimes you need two inputs. Java provides "Bi" versions of the main interfaces.
* **Examples:**
* `BiPredicate<T, U>`: `test(T t, U u)` -> boolean
* `BiConsumer<T, U>`: `accept(T t, U u)` -> void
* `BiFunction<T, U, R>`: `apply(T t, U u)` -> R


* **Note:** There is no `TriFunction`. If you need 3+ arguments, you must define your own interface.

### 6️⃣ Operator Interfaces (Same Type)

* **🧠 Theory:** Special cases of `Function` where the input and output are the **same type**.
* `UnaryOperator<T>`: extends `Function<T, T>`
* `BinaryOperator<T>`: extends `BiFunction<T, T, T>`


* **💻 Code:**
```java
UnaryOperator<Integer> square = x -> x * x; // Input Int -> Output Int

```



---

## 🌟 Senior & Lead Developer Addendum (Design & Patterns)

### 7️⃣ Default Methods & Multiple Inheritance

* **🧠 Theory:** Functional Interfaces can have `default` methods. This leads to the "Diamond Problem" if a class implements two interfaces with the same default method.
* **Resolution:** The implementing class **must** override the conflicting method to resolve the ambiguity.
* **💻 Code:**
```java
interface A { default void hello() { System.out.println("A"); } }
interface B { default void hello() { System.out.println("B"); } }

class C implements A, B {
    // Compilation Error unless you override!
    public void hello() {
        A.super.hello(); // Explicitly choose A
    }
}

```



### 8️⃣ Checked Exceptions in Lambdas

* **🧠 Theory:** This is the biggest pain point in Java 8. Functional Interfaces **do not** declare checked exceptions.
* **The Problem:** You cannot throw a checked exception (like `IOException`) inside a standard lambda.
* **The Fix:** You must wrap the code in a `try-catch` block inside the lambda, or write a "Wrapper" method that converts the checked exception into a RuntimeException.
* **💻 Code:**
```java
// ❌ Compile Error
// Consumer<String> c = s -> new FileWriter(s); 

// ✅ Ugly Fix
Consumer<String> c = s -> {
    try { new FileWriter(s); }
    catch (IOException e) { throw new RuntimeException(e); }
};

```



### 9️⃣ "Effectively Final" Variable Capture

* **🧠 Theory:** Lambdas capture values, not variables. If you use a local variable inside a lambda, it is effectively "copied" into the lambda instance.
* **Why?** Because the local variable lives on the **Stack**, which disappears when the method returns. The lambda object lives on the **Heap** and might execute long after the stack frame is gone. To ensure consistency, Java forbids changing the variable, creating an immutable snapshot.

```

---

**Would you like to continue to the next logical step: `Streams API` (which heavily uses these interfaces)? Or do you want to explore `Optional`?**

```