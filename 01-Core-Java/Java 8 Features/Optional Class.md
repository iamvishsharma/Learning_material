# Optional Class

## Notes

# 📘 Complete Java 8 Optional Class Reference

### 1️⃣ Overview
* **🧠 Theory:** `Optional<T>` is a container object which may or may not contain a non-null value. It was introduced to provide a type-level solution for representing "optional" values instead of using `null` references.
* **The Goal:** To reduce `NullPointerException` (NPE) and force the developer to explicitly think about the case where a value might be missing.
* **Key Concept:** It is a **wrapper**. You don't access the value directly; you go through the Optional API.

### 2️⃣ Creating Optional Objects
* **🧠 Theory:** There are three static factory methods to create an Optional.
* **💻 Code:**
  ```java
  // 1. Empty Optional
  Optional<String> empty = Optional.empty();

  // 2. Non-null value (Throws NPE if value is null)
  Optional<String> opt = Optional.of("Java");

  // 3. Nullable value (Safe: returns empty() if value is null)
  Optional<String> maybe = Optional.ofNullable(someVariable);

```

### 3️⃣ Accessing the Value (The Safe Way)

* **🧠 Theory:** Never call `.get()` without checking `.isPresent()`. Better yet, use the functional "Or" methods.
* **💻 Code:**
```java
Optional<String> name = getNameFromDB();

// ❌ BAD: Can still throw NoSuchElementException
String s = name.get(); 

// ✅ BETTER (Traditional):
if (name.isPresent()) { ... }

// ✅ BEST (Functional):
String result = name.orElse("Default Name"); 
String result2 = name.orElseGet(() -> fetchDefaultFromService());
name.ifPresent(System.out::println);

```



### 4️⃣ Chaining with Map and Filter

* **🧠 Theory:** Just like Streams, `Optional` has `map`, `flatMap`, and `filter`. This allows you to process a potential value without writing nested `if` blocks.
* **💻 Code:**
```java
// Replaces nested null checks
Optional<String> city = Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .filter(c -> c.startsWith("New"));

```



### 5️⃣ Difference between `orElse` and `orElseGet`

* **🧠 Theory:** This is a common senior interview question.
* `orElse(T other)`: The "other" value is evaluated **always**, even if the Optional is not empty.
* `orElseGet(Supplier other)`: The supplier is executed **only** if the Optional is empty (Lazy evaluation).


* **💻 Performance Tip:** Use `orElseGet` if the default value requires a database call or heavy computation.

---

## 🌟 Senior & Lead Developer Addendum (Best Practices & Design)

### 6️⃣ Optional is for Return Types ONLY

* **🧠 Theory:** According to Brian Goetz (Java Architect), Optional was intended strictly for **Library Method Return Types** where there is a "no result" case.
* **❌ Anti-Pattern:** Using Optional as a method parameter.
* *Why?* It forces the caller to wrap values, creating unnecessary objects on the heap. It's better to use standard null checks or `@NotNull` annotations on parameters.


* **❌ Anti-Pattern:** Using Optional as a field in a class.
* *Why?* Optional is **not Serializable**. If your class needs to be serialized (like in a DTO for a session), it will fail.



### 7️⃣ `flatMap` and Nested Optionals

* **🧠 Theory:** If your mapping function returns an `Optional`, `map()` will result in a nested `Optional<Optional<T>>`. Use `flatMap()` to keep it flat.
* **💻 Code:**
```java
// address.getCity() returns Optional<String>
Optional<String> city = user.map(User::getAddress)
                            .flatMap(Address::getCity); 

```



### 8️⃣ Performance and Memory Overhead

* **🧠 Theory:** Every `Optional` is an object. In a high-performance loop (e.g., processing millions of items), wrapping every result in an `Optional` increases pressure on the **Garbage Collector** and fills up the **Eden Space**.
* **Lead Perspective:** Use `Optional` for high-level business logic. For low-level, high-frequency internal logic, sticking to `null` and careful null-checking might be more performant.

### 9️⃣ The Primitive Optionals

* **🧠 Theory:** Just like Streams, avoid `Optional<Integer>` to prevent boxing.
* **💻 The Fix:** Use `OptionalInt`, `OptionalLong`, and `OptionalDouble`.
```java
OptionalInt max = IntStream.of(1, 2, 3).max();

```



### 🔟 How it Handles the "Billion Dollar Mistake"

* **🧠 Theory:** Tony Hoare (inventor of null) called it his "billion-dollar mistake." `Optional` doesn't *eliminate* nulls; it makes them **visible**. It shifts the burden from the runtime (NPE crashes) to the compile-time (API forced handling).

```

---

**This wraps up the core Java 8 features! Would you like to move on to the Design Patterns you requested earlier? I suggest starting with the Factory Method Pattern or the Abstract Factory Pattern.**

```