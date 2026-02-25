# Method Reference

## Notes

# 📘 Complete Java 8 Method Reference Reference

### 1️⃣ Overview
* **🧠 Theory:** A **Method Reference** is a shorthand notation for a lambda expression that only calls an existing method. It provides a way to refer to a method without executing it.
* **The Goal:** To make the code even more readable than a lambda. If a lambda is just "passing through" arguments to a method, a method reference replaces the boilerplate.
* **Symbol:** Uses the double colon `::` operator.

### 2️⃣ The Relationship with Lambdas
* **🧠 Theory:** Every method reference has a corresponding lambda expression, but not every lambda can be a method reference (only those that call exactly one existing method).
* **💻 Code:**
  ```java
  // Lambda version
  names.forEach(s -> System.out.println(s));

  // Method Reference version (Identical behavior)
  names.forEach(System.out::println);

```

### 3️⃣ The Four Types of Method References

There are four distinct ways to use the `::` operator:

#### A. Static Method Reference

* **Syntax:** `ContainingClass::staticMethodName`
* **💻 Code:**
```java
// Lambda: (str) -> Integer.parseInt(str)
Function<String, Integer> converter = Integer::parseInt;

```



#### B. Instance Method of a Particular Object

* **Syntax:** `containingObject::instanceMethodName`
* **💻 Code:**
```java
String prefix = "LOG: ";
// Lambda: (msg) -> prefix.concat(msg)
UnaryOperator<String> logger = prefix::concat;

```



#### C. Instance Method of an Arbitrary Object of a Particular Type

* **Syntax:** `ContainingClass::instanceMethodName`
* **🧠 Theory:** This is the trickiest one. The first argument of the lambda becomes the *target* of the method call, and any subsequent arguments become the method *parameters*.
* **💻 Code:**
```java
// Lambda: (s1, s2) -> s1.compareToIgnoreCase(s2)
// s1 is the target object, s2 is the parameter
Comparator<String> comp = String::compareToIgnoreCase;

```



#### D. Constructor Reference

* **Syntax:** `ClassName::new`
* **💻 Code:**
```java
// Lambda: () -> new ArrayList<>()
Supplier<List<String>> listFactory = ArrayList::new;

```



### 4️⃣ Parameter Matching Logic

* **🧠 Theory:** How does the compiler know which method to pick?
* **The Rule:** The signature of the referenced method must be "compatible" with the Functional Interface’s abstract method. The compiler checks the number of arguments and the return type.
* **Example:** `System.out::println` works for `Consumer<String>` because `println` takes one argument and returns `void`, matching `accept(T t)`.

### 5️⃣ When to Use Method References

* **🧠 Theory:**
✅ Whenever a lambda is simply acting as a "proxy" for a single method call.
✅ To improve code clarity and follow the "Clean Code" principle of reducing unnecessary syntax.

---

## 🌟 Senior & Lead Developer Addendum (Internals & Nuances)

### 6️⃣ Laziness and Evaluation

* **🧠 Theory:** When you use `containingObject::instanceMethodName`, the `containingObject` reference is evaluated **immediately** when the method reference is created, not when it is eventually executed.
* **💻 The Trap:**
```java
MyObject obj = new MyObject();
Runnable r = obj::doWork; 
obj = null; // This does NOT affect the method reference
r.run();    // Works fine because the reference to the original object was captured

```



### 7️⃣ Bytecode Mechanics (`invokedynamic`)

* **🧠 Theory:** Just like lambdas, method references do not generate anonymous inner class files. They use the `invokedynamic` instruction.
* **Internal Optimization:** In many cases, the JVM can optimize a method reference better than a lambda because the target method is already explicitly known at compile time, allowing for more aggressive inlining by the JIT (Just-In-Time) compiler.

### 8️⃣ Handling Overloaded Methods

* **🧠 Theory:** If a class has multiple overloaded methods with the same name, the compiler chooses the one that matches the Functional Interface's descriptor.
* **💻 Code:**
```java
// System.out.println has overloads for int, String, boolean, etc.
// The compiler picks println(String) because the interface is Consumer<String>.
Consumer<String> c = System.out::println; 

```



### 9️⃣ The "Arbitrary Object" vs "Static" Confusion

* **🧠 Theory:** A common interview question is why `String::toUpperCase` looks like a static reference when `toUpperCase()` is an instance method.
* **The Answer:** It's the "Type C" reference (Arbitrary Object). The compiler understands that the lambda `(String s) -> s.toUpperCase()` can be represented as `String::toUpperCase`. This is the core of functional patterns in Java Streams (e.g., `stream.map(String::trim)`).

```

---

**Next up, we should tackle the `Optional` class. It’s a game-changer for handling `NullPointerException` in a functional way. Ready?**

```