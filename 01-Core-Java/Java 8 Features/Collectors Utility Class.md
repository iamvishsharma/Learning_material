# Collectors Utility Class

## Notes

# 📘 Complete Java 8 Collectors Reference

### 1️⃣ Overview
* **🧠 Theory:** `Collectors` is a final utility class that provides implementations of the `Collector` interface. It is used as the argument for the `.collect()` terminal operation in the Stream API.
* **The Goal:** To wrap up the results of a stream pipeline into a final structure like a `List`, `Set`, `Map`, or a summarized value.

### 2️⃣ Common Collection Collectors
* **🧠 Theory:** The most frequent use case is turning a stream back into a standard collection.
* **💻 Code:**
  ```java
  // 1. To List
  List<String> list = stream.collect(Collectors.toList());

  // 2. To Set (Deduplication)
  Set<String> set = stream.collect(Collectors.toSet());

  // 3. To Specific Implementation
  LinkedList<String> linked = stream.collect(Collectors.toCollection(LinkedList::new));

```

### 3️⃣ String Joining

* **🧠 Theory:** Concatenates stream elements into a single `String`, with optional delimiters, prefixes, and suffixes.
* **💻 Code:**
```java
String result = list.stream()
    .collect(Collectors.joining(", ", "[", "]")); 
// Output: [Apple, Banana, Orange]

```



### 4️⃣ Data Summarization

* **🧠 Theory:** Quickly calculate statistics like sum, average, or max.
* **💻 Code:**
```java
Double avg = list.stream().collect(Collectors.averagingInt(String::length));

// Get all stats at once (min, max, count, sum, average)
IntSummaryStatistics stats = list.stream().collect(Collectors.summarizingInt(String::length));

```



### 5️⃣ Partitioning vs. Grouping

* **🧠 Theory:**
* **`partitioningBy`:** Splitting data into exactly two groups (**True/False**) based on a Predicate.
* **`groupingBy`:** Creating multiple groups based on a classifier (similar to SQL `GROUP BY`).


* **💻 Code:**
```java
// Partitioning (Even vs Odd)
Map<Boolean, List<Integer>> evenOdd = nums.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));

// Grouping (By Department)
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

```



---

## 🌟 Senior & Lead Developer Addendum (Complex Transformations)

### 6️⃣ Downstream Collectors (Multi-level Grouping)

* **🧠 Theory:** `groupingBy` is overloaded. You can pass a *second* collector to it to perform operations *inside* each group.
* **💻 Code:**
```java
// Get count of employees in each department
Map<String, Long> deptCounts = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment, 
        Collectors.counting() // Downstream collector
    ));

```



### 7️⃣ `toMap()` and Conflict Resolution

* **🧠 Theory:** When using `toMap()`, if two elements have the same key, it throws an `IllegalStateException`. You must provide a "Merge Function."
* **💻 Code:**
```java
// (existingValue, newValue) -> keep the existing one
Map<Integer, String> map = list.stream()
    .collect(Collectors.toMap(
        String::length, 
        s -> s, 
        (existing, replacement) -> existing 
    ));

```



### 8️⃣ `mapping()` and `flatMapping()` (Java 9+)

* **🧠 Theory:** Sometimes you want to transform elements *before* they are collected into a group.
* **💻 Code:**
```java
// Group by Dept, but only collect the Names of the employees
Map<String, List<String>> deptNames = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.mapping(Employee::getName, Collectors.toList())
    ));

```



### 9️⃣ Performance: `joining()` on Large Streams

* **🧠 Theory:** `Collectors.joining()` internally uses `StringBuilder`. While efficient, be careful with massive parallel streams. The final merge step where the strings are combined can become a single-threaded bottleneck.

### 🔟 Custom Collectors

* **🧠 Theory:** You can create your own collector by implementing the `Collector` interface. This requires defining:
1. **Supplier:** Initial result container.
2. **Accumulator:** How to add an element to the container.
3. **Combiner:** How to merge two containers (for parallel processing).
4. **Finisher:** Final transformation (optional).



---

**This covers the core and advanced utility of the Collectors class. Would you like to proceed with Design Patterns (like Factory or Strategy) or should we look at Spring Boot's core annotations and bean lifecycle?**