# 10 TreeSet
## Notes

# 📘 Complete Java TreeSet Reference

### 1️⃣ Overview
* **🧠 Theory:** `TreeSet` is a `NavigableSet` implementation based on a **TreeMap**. It guarantees that the elements will be in **ascending, sorted order**. It does not allow duplicate elements, does **not allow `null` elements**, and is **not thread-safe**.

### 2️⃣ Class Hierarchy
* **🧠 Theory:** It extends `AbstractSet` and implements `NavigableSet<E>` (which extends `SortedSet`), `Cloneable`, and `Serializable`.
* **💻 Code (Architecture):**
  ```text
  Object
    ↳ AbstractCollection<E>
         ↳ AbstractSet<E>
              ↳ TreeSet<E>

```

### 3️⃣ Internal Data Structure (The Secret Wrapper)

* **🧠 Theory:** Just as `HashSet` is a wrapper around `HashMap`, `TreeSet` is literally just a wrapper around a `TreeMap`.
* **The Mechanics:** The elements you add to the `TreeSet` become the **keys** in the backing `TreeMap`. A dummy constant object called `PRESENT` is used as the value for every entry.
* **💻 Code:**
```java
private transient NavigableMap<E,Object> m; // The backing map
private static final Object PRESENT = new Object(); // The dummy value

```



### 4️⃣ 🔬 Deep Dive: Internal Working (Red-Black Tree)

Because it relies on `TreeMap`, it physically stores your data in a **Red-Black Tree**.

1. **Comparison:** When you add an element, it does not calculate a hash code. Instead, it starts at the root node and compares the new element with the root using `compareTo()` (or a `Comparator`).
2. **Traversal:** If your element is *less*, it goes left. If *greater*, it goes right.
3. **Balancing:** Once it finds an empty leaf, it inserts the node. If the Red-Black tree rules are violated, it performs rotations and recoloring to keep the tree balanced.

### 5️⃣ Constructors (Natural vs Custom Order)

* **🧠 Theory:** The set must know how to sort your objects.
* **💻 Code:**
```java
// 1. Natural Ordering (Objects MUST implement Comparable)
TreeSet<String> set = new TreeSet<>();

// 2. Custom Comparator (e.g., sort by string length)
TreeSet<String> set = new TreeSet<>(Comparator.comparing(String::length));

// 3. From Collection (Sorts them instantly)
TreeSet<String> set = new TreeSet<>(existingList);

```



### 6️⃣ Time Complexity

* **🧠 Theory:** Operations scale logarithmically because it must traverse a balanced binary tree.
* **💻 Code (Operations):**
* `add(e)` / `remove(e)` / `contains(e)` → **O(\log n)**
* *Note: This is strictly slower than HashSet's O(1) for basic operations.*



### 7️⃣ Fail-Fast Behavior

* **🧠 Theory:** Iterators are fail-fast. Modifying the set structure while iterating throws `ConcurrentModificationException`.
* **Fun Fact:** Iterating over a `TreeSet` guarantees you get the elements in perfectly sorted order!

### 8️⃣ Thread Safety

* **🧠 Theory:** `TreeSet` is **not** thread-safe.
* **💻 Code:**
```java
// Synchronized Wrapper
SortedSet<String> s = Collections.synchronizedSortedSet(new TreeSet<>());

// Modern Concurrent Alternative
NavigableSet<String> concurrentSet = new ConcurrentSkipListSet<>();

```



### 9️⃣ The "No Null Elements" Rule

* **🧠 Theory:** `TreeSet` absolutely **does not allow `null**`.
* **Why?** Because it has to sort the elements. When it tries to figure out where `null` belongs, it executes `null.compareTo(otherElement)`. This immediately throws a `NullPointerException`.

### 🔟 NavigableSet Superpowers (Range & Neighbors)

* **🧠 Theory:** This is the primary reason `TreeSet` exists. It provides $O(\log n)$ methods to find elements relative to other elements.
* **💻 Code:**
```java
set.first();        // Lowest element
set.last();         // Highest element
set.lower(50);      // Greatest element strictly < 50
set.floor(50);      // Greatest element <= 50
set.ceiling(50);    // Lowest element >= 50
set.higher(50);     // Lowest element strictly > 50

// Returns a view of the set containing elements from 10 to 49
SortedSet<Integer> sub = set.subSet(10, 50); 

```



### 1️⃣1️⃣ Memory Behavior

* **🧠 Theory:** Very **high memory footprint**. Because it uses a `TreeMap` underneath, every single element in your set requires a full `TreeMap.Entry` node (which contains pointers for `left`, `right`, `parent`, a `color` boolean, plus references to your element and the `PRESENT` dummy object).

### 1️⃣2️⃣ When to Use TreeSet

* **🧠 Theory:**
✅ When you need a collection of unique elements strictly maintained in **sorted order**.
✅ When you frequently need to perform **Range Queries** (e.g., "give me all transactions between $100 and $500").
✅ When you need to find the "closest" match to a value (ceiling/floor).

### 1️⃣3️⃣ When NOT to Use

* **🧠 Theory:**
❌ For general-purpose unique collections. Use `HashSet` for $O(1)$ performance.
❌ If you just need elements sorted *once* at the end. (It's faster to put them in an `ArrayList` and call `Collections.sort()` than to pay the $O(\log n)$ penalty on every single insertion).

---

## 🌟 Senior & Lead Developer Addendum (System Design & Internals)

### 1️⃣4️⃣ The `compareTo()` vs `equals()` Trap (The Invisible Bug)

* **🧠 Theory:** A `Set` is defined by the rule "no duplicates allowed." A `HashSet` uses `equals()` to check for duplicates. **`TreeSet` completely ignores `equals()`.**
* **The Rule:** `TreeSet` considers two elements duplicates if and only if `element1.compareTo(element2) == 0`.
* **The Interview Trap:** Imagine an `Employee` class. You override `equals()` to check the `employeeId`. But you write `compareTo()` to sort by `lastName`. If you add two *different* employees who both have the last name "Smith", `compareTo()` returns `0`. The `TreeSet` thinks they are identical and will **refuse to add the second employee**, silently discarding your data.
* **The Fix:** Your `compareTo` method MUST be "consistent with equals." If last names match, your `compareTo` must fall back to comparing the `employeeId` to break the tie.

### 1️⃣5️⃣ Why not a `TreeList`?

* **🧠 Theory:** Interviewers sometimes ask why there is no `TreeList` in Java.
* **The Answer:** A `List` by definition is accessed by an integer index (e.g., `get(5)`). A Red-Black tree has no concept of absolute indexes; it only knows relative placement (greater than/less than). Finding the 5th element in a tree requires an $O(n)$ traversal from the leftmost node, defeating the entire purpose of a $O(\log n)$ tree structure.

### 1️⃣6️⃣ `ConcurrentSkipListSet` (The Multi-threaded Sibling)

* **🧠 Theory:** Because balancing a Red-Black tree across multiple threads is incredibly difficult (a single rotation locks huge portions of the tree), Java provides `ConcurrentSkipListSet` instead of a "ConcurrentTreeSet".
* **How it works:** It uses a Skip List (layers of linked lists with "express lanes"). This provides the same $O(\log n)$ search and sorting capabilities but is infinitely easier to make thread-safe using lock-free CAS (Compare-And-Swap) operations.

### 1️⃣7️⃣ CPU Cache Misses

* **🧠 Theory:** Due to the node-based nature of the Red-Black tree, nodes are scattered across the JVM heap. Iterating through a `TreeSet` sequentially causes constant CPU cache misses, making it mechanically slower for bulk operations than array-backed structures, even if the Big-O math looks good on paper.

```

---

**This covers the entirety of the major Collections framework!** Would you like to shift focus to **Java Design Patterns** (Strategy, Factory, Adapter) that you mentioned earlier, or do you want to cover any of the modern Java 8+ structures (like `Streams` or `Optional`)?

```