# Streams API

## Notes

# 📘 Complete Java 8 Streams API Reference

### 1️⃣ Overview
* **🧠 Theory:** The **Streams API** is a powerful tool for processing sequences of elements (collections, arrays, etc.) in a functional and declarative style. 
* **The Core Concept:** A Stream is **not** a data structure. It does not store data; it carries data from a source through a pipeline of computational steps.
* **Key Traits:** * **Declarative:** You describe *what* you want (filter, sort), not *how* to loop.
    * **Pipelined:** Most operations return a new stream, allowing for chaining.
    * **Lazy:** Operations aren't executed until a final result is needed.

### 2️⃣ The Stream Pipeline
* **🧠 Theory:** Every stream operation consists of three parts:
    1. **Source:** Where the data comes from (`List`, `Set`, `Array`, `I/O channel`).
    2. **Intermediate Operations:** Transform the stream into another stream (e.g., `filter`, `map`).
    3. **Terminal Operation:** Produces a result or a side-effect (e.g., `collect`, `forEach`, `count`).



### 3️⃣ Intermediate Operations (Lazy)
* **🧠 Theory:** These operations are **lazy**. They are recorded but not executed until a terminal operation is called.
* **Common Operations:**
    * `filter(Predicate)`: Excludes elements that don't match the condition.
    * `map(Function)`: Transforms each element (e.g., `String` to `Integer`).
    * `flatMap(Function)`: Flattens nested structures (e.g., `Stream<List<String>>` to `Stream<String>`).
    * `sorted()`: Sorts the elements.
    * `distinct()`: Removes duplicates based on `equals()`.

### 4️⃣ Terminal Operations (Eager)
* **🧠 Theory:** These trigger the processing of the pipeline. Once a terminal operation is called, the stream is **consumed** and cannot be used again.
* **Common Operations:**
    * `forEach(Consumer)`: Performs an action for each element.
    * `collect(Collector)`: Converts the stream back into a collection (`toList`, `toSet`, `joining`).
    * `reduce(BinaryOperator)`: Combines elements into a single value (e.g., sum, max).
    * `anyMatch`, `allMatch`, `noneMatch`: Returns a boolean.
    * `findFirst`, `findAny`: Returns an `Optional`.

### 5️⃣ 🔬 Deep Dive: Lazy Evaluation & Short-Circuiting
* **🧠 Theory:** Java Streams are highly efficient because they perform **Loop Fusion**. Instead of looping through the whole list for `filter` and *then* again for `map`, Java combines them into a single pass.
* **Short-Circuiting:** Operations like `limit(n)` or `findFirst()` can stop processing as soon as the criteria is met, even if the source has millions of items.



### 6️⃣ Parallel Streams
* **🧠 Theory:** You can turn any stream into a parallel stream by calling `.parallelStream()`. This uses the **ForkJoinPool.commonPool()** to process chunks of data across multiple CPU cores.
* **💻 Code:**
  ```java
  long count = list.parallelStream()
                   .filter(s -> s.startsWith("A"))
                   .count();

```

* **⚠️ Warning:** Only use parallel streams for large datasets and stateless, non-blocking operations. For small lists, the overhead of splitting and merging threads is slower than a sequential stream.

---

## 🌟 Senior & Lead Developer Addendum (Performance & Design)

### 7️⃣ `map()` vs `flatMap()` (The Interview Classic)

* **🧠 Theory:** * `map()` is a **1-to-1** transformation.
* `flatMap()` is a **1-to-many** transformation.


* **Use Case:** If you have a `List<Order>` and each order has a `List<LineItem>`, use `flatMap` to get a single `Stream<LineItem>` containing every item from every order.

### 8️⃣ Stateless vs. Stateful Operations

* **🧠 Theory:** * **Stateless:** `filter`, `map`. The operation on one element doesn't depend on others. These are fast and scale well.
* **Stateful:** `sorted`, `distinct`, `limit`. These require the stream to see *all* or *some* previous elements before proceeding. This can be a bottleneck in parallel streams.



### 9️⃣ Stream Reuse Trap

* **🧠 Theory:** A stream is like a physical pipe; once the data flows through it, it's gone.
* **💻 Code:**
```java
Stream<String> s = list.stream();
s.count(); // Works
s.forEach(System.out::println); // ❌ IllegalStateException: stream has already been operated upon or closed

```



### 🔟 Numeric Streams (Avoiding Boxed Overhead)

* **🧠 Theory:** Standard `Stream<Integer>` suffers from boxing/unboxing overhead.
* **💻 The Fix:** Use primitive streams: `IntStream`, `LongStream`, and `DoubleStream`.
```java
// ❌ Slow: Stream<Integer>
int sum = list.stream().mapToInt(i -> i).sum(); 

// ✅ Fast: IntStream
int sum = IntStream.range(1, 100).sum(); 

```



### 1️⃣1️⃣ The "Side-Effect" Danger

* **🧠 Theory:** Functional programming should be **Pure**. You should avoid modifying external variables inside a stream (`peek` or `forEach`).
* **💻 Bad Practice:**
```java
List<String> results = new ArrayList<>();
list.stream().filter(s -> s.length() > 5).forEach(results::add); // ❌ Side-effect!

// ✅ Good Practice: Use Collectors
List<String> results = list.stream().filter(s -> s.length() > 5).collect(Collectors.toList());

```



```

---

**Would you like to explore `Optional` next? It is the standard way to handle the results of stream operations like `findFirst()` or `reduce()`.**

```