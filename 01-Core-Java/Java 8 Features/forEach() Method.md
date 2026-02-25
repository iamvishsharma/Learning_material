# forEach() Method

## Notes

# 📘 Complete Java 8 forEach() Method Reference

### 1️⃣ Overview
* **🧠 Theory:** `forEach()` is a default method introduced in the `Iterable` interface (for collections) and a terminal operation in the `Stream` API. It performs a specific action for each element of the data source.
* **The Goal:** To provide a clean, internal iteration mechanism that supports functional programming and lambdas.

### 2️⃣ Collection.forEach() vs. Stream.forEach()
* **🧠 Theory:**
    * **`Iterable.forEach()`:** Iterates over the collection directly (e.g., `List`, `Set`).
    * **`Stream.forEach()`:** A terminal operation that consumes the stream. It can be executed in parallel, whereas collection iteration is always sequential.
* **💻 Code:**
  ```java
  List<String> items = Arrays.asList("A", "B", "C");

  // 1. Collection forEach
  items.forEach(System.out::println);

  // 2. Stream forEach
  items.stream().filter(s -> s.startsWith("A")).forEach(System.out::println);

```

### 3️⃣ Map.forEach()

* **🧠 Theory:** Java 8 added a specialized `forEach` to the `Map` interface that takes a **BiConsumer**, making it significantly cleaner than the old `entrySet()` loops.
* **💻 Code:**
```java
Map<Integer, String> map = new HashMap<>();
map.put(1, "Java");

// No more EntrySet boilerplate!
map.forEach((key, value) -> System.out.println(key + " : " + value));

```



### 4️⃣ Internal vs. External Iteration

* **🧠 Theory:**
* **External (Old):** You control the iterator (`for`, `while`). You decide *how* to iterate.
* **Internal (New):** You pass the *what* (the logic) to the collection, and the collection decides *how* to iterate. This allows for better optimizations and easy parallelization.



### 5️⃣ The Consumer Contract

* **🧠 Theory:** `forEach` accepts a `Consumer<T>`. This means:
1. It takes one input.
2. It returns **void**.
3. It is used strictly for **Side Effects** (printing, logging, updating external state).



---

## 🌟 Senior & Lead Developer Addendum (Constraints & Logic)

### 6️⃣ The Break/Continue Problem

* **🧠 Theory:** This is a classic interview question. **You cannot use `break` or `continue` inside a `forEach` loop.**
* **Why?** A lambda is an anonymous function. `break` and `continue` only work inside standard loops (`for`/`while`).
* **The Workaround:** * If you need to skip an element, use `return;` (acts like `continue`).
* If you need to stop the loop, you must use a traditional `for` loop or a Stream with `anyMatch()` or `takeWhile()` (Java 9+).



### 7️⃣ forEach() in Parallel Streams (Order Danger)

* **🧠 Theory:** In a `parallelStream()`, `forEach()` executes in multiple threads. The order of execution is **not guaranteed**.
* **The Fix:** If order is required, use `forEachOrdered()`.
* **💻 Performance Note:** `forEachOrdered()` destroys the performance benefit of parallel processing because it forces the threads to coordinate and wait to maintain sequence.
```java
// Prints in random order
list.parallelStream().forEach(System.out::println); 

// Prints in insertion order, but slower
list.parallelStream().forEachOrdered(System.out::println); 

```



### 8️⃣ Modifying the Collection (Fail-Fast)

* **🧠 Theory:** Modifying the underlying collection inside its own `forEach` method (e.g., `list.add()` or `list.remove()`) will throw a `ConcurrentModificationException`.
* **💻 Code:**
```java
// ❌ Throws Exception
list.forEach(item -> {
    if(item.equals("A")) list.remove(item); 
});

```



### 9️⃣ Side Effects vs. Stream Design

* **🧠 Theory:** At the senior level, you should know that using `forEach` to populate another list is an **anti-pattern**.
* **Lead Perspective:** If you are using `forEach` to transform data and put it elsewhere, you should be using `.collect(Collectors.toList())` instead. `forEach` should be reserved for terminal actions like printing or saving to a database.

### 🔟 forEach with Checked Exceptions

* **🧠 Theory:** Because `Consumer.accept()` does not throw checked exceptions, you cannot call a method that throws `IOException` directly inside `forEach`.
* **The Fix:** Wrap it in a `try-catch` or use a functional wrapper that converts checked exceptions into `RuntimeException`.

```

---

**This covers the most important aspects of Java 8 iteration. Would you like to proceed to the Design Patterns (Factory, Strategy, Observer) or discuss the differences between Java 8 and higher versions (9-17)?**

```