# 07 Tree Map

## Notes

# 📘 Complete Java TreeMap Reference

### 1️⃣ Overview
* **🧠 Theory:** `TreeMap` is a map implementation based on a **Red-Black Tree**. It stores its keys in a **sorted and ascending order**. Unlike `HashMap`, it does not use hashing at all.
* **Key Feature:** It implements `NavigableMap`, which means it is heavily optimized for **range queries** (e.g., "give me all keys between A and F" or "what is the closest key strictly greater than X?").

### 2️⃣ Class Hierarchy
* **🧠 Theory:** It extends `AbstractMap` and implements `NavigableMap<K,V>` (which extends `SortedMap`), `Cloneable`, and `Serializable`.
* **💻 Code (Architecture):**
  ```text
  Object
    ↳ AbstractMap<K,V>
         ↳ TreeMap<K,V>

```

### 3️⃣ Internal Data Structure (Red-Black Tree)

* **🧠 Theory:** There is no underlying array. Every key-value pair is wrapped in a tree `Entry` (Node). Each node has references to its parent, left child, right child, and a boolean representing its "Color" (Red or Black).
* **💻 Code:**
```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK; // Color flag for tree balancing
}

```



### 4️⃣ 🔬 Deep Dive: Internal Working (Red-Black Balancing)

When you call `map.put(key, value)`, `TreeMap` must figure out exactly where to place the node to keep the tree sorted and balanced.

1. **Binary Search:** It starts at the `root` node. If your new key is *less* than the root, it goes left. If *greater*, it goes right. It repeats this until it finds an empty spot (a leaf).
2. **Insertion (Always Red):** It drops the new node in as a **Red** node.
3. **The Balancing Act:** A Red-Black tree has strict rules (e.g., you cannot have two Red nodes in a row). If dropping the new node breaks the rules, the tree performs **Rotations** (shifting nodes left or right) and **Recolorings** to fix the tree.
4. **The Guarantee:** This self-balancing ensures the tree never becomes a straight line (a Linked List), guaranteeing **O(log n)** performance even in the worst case.

### 5️⃣ Constructors (Natural vs Custom Ordering)

* **🧠 Theory:** A `TreeMap` must know how to sort its keys. You can either let it use the keys' natural ordering (keys must implement `Comparable`), or you can pass in a custom `Comparator`.
* **💻 Code:**
```java
// 1. Default (Natural Ordering - Keys MUST implement Comparable)
TreeMap<String, Integer> map = new TreeMap<>();

// 2. Custom Comparator (e.g., Reverse Order)
TreeMap<String, Integer> map = new TreeMap<>(Collections.reverseOrder());

```



### 6️⃣ Time Complexity

* **🧠 Theory:** Because it is a balanced binary search tree, operations scale logarithmically. It is significantly slower than `HashMap` for raw lookups, but much faster for finding "neighbors".
* **💻 Code (Operations):**
* `get(key)` / `put(key, value)` / `remove(key)` → **O(log n)**
* `containsKey(key)` → **O(log n)**



### 7️⃣ Fail-Fast Behavior

* **🧠 Theory:** Standard `modCount` fail-fast implementation. Iterating through the map's `entrySet()`, `keySet()`, or `values()` and structurally modifying the map concurrently will throw a `ConcurrentModificationException`.

### 8️⃣ Thread Safety

* **🧠 Theory:** `TreeMap` is **not** thread-safe.
* **💻 Code:**
```java
// Synchronized Wrapper
SortedMap<K,V> syncMap = Collections.synchronizedSortedMap(new TreeMap<>());

// Better Concurrent Alternative:
ConcurrentSkipListMap<K,V> concurrentMap = new ConcurrentSkipListMap<>();

```



### 9️⃣ The "No Null Keys" Rule

* **🧠 Theory:** `TreeMap` **does not allow `null` keys**. When you call `put(null, value)`, the tree must compare `null` against existing keys to figure out where to place it in the sorted tree. Calling `null.compareTo(otherKey)` immediately throws a `NullPointerException`.
* **Note:** `null` *values* are perfectly fine.

### 🔟 NavigableMap Superpowers (Range Queries)

* **🧠 Theory:** This is the primary reason to use a `TreeMap`. It provides extremely fast methods to find keys relative to other keys without scanning the whole map.
* **💻 Code:**
```java
map.firstKey();       // Lowest key
map.lastKey();        // Highest key
map.higherKey("C");   // Strictly greater than "C"
map.lowerKey("C");    // Strictly less than "C"
map.ceilingKey("C");  // Greater than OR equal to "C"

// Returns a view of the map between "A" and "M"
SortedMap<String, V> sub = map.subMap("A", "M"); 

```



### 1️⃣1️⃣ Memory Overhead

* **🧠 Theory:** The memory overhead is very high. Every single entry requires an object header, the key reference, the value reference, three node pointers (`left`, `right`, `parent`), and a boolean for `color`.

### 1️⃣2️⃣ When to Use TreeMap

* **🧠 Theory:**
✅ When you need a dictionary strictly sorted by keys.
✅ When you frequently need to perform **Range Queries** (e.g., "fetch all logs between Date X and Date Y").
✅ When you need to find the "closest" match (ceiling/floor).

### 1️⃣3️⃣ When NOT to Use

* **🧠 Theory:**
❌ For general-purpose caching or lookups (use `HashMap` for $O(1)$ speed).
❌ When you only care about insertion order (use `LinkedHashMap`).

---

## 🌟 Senior & Lead Developer Addendum (System Design & Internals)

### 1️⃣4️⃣ The `compareTo()` vs `equals()` Gotcha (Interview Trap!)

* **🧠 Theory:** `HashMap` uses `equals()` and `hashCode()` to determine if a key is a duplicate. **`TreeMap` ignores `equals()` entirely.** * **The Rule:** `TreeMap` considers two keys duplicate if and only if `key1.compareTo(key2) == 0`.
* **The Danger:** If your custom object returns `0` for `compareTo()` based only on an ID, but `equals()` checks ID *and* Name, `TreeMap` will overwrite your entries thinking they are duplicates, even though `equals()` says they are different. Your `compareTo()` logic **must** be consistent with `equals()`.

### 1️⃣5️⃣ Why Red-Black Tree instead of AVL Tree?

* **🧠 Theory:** Interviewers love asking why Java chose Red-Black Trees over AVL trees for `TreeMap`.
* **The Answer:** AVL trees are strictly balanced, meaning lookups are marginally faster, but insertions and deletions cause massive amounts of tree rotations. Red-Black trees allow slightly more imbalance (the longest path is no more than twice the shortest path). This means **Red-Black trees require significantly fewer rotations during `put()` and `remove()**`, making them much more efficient for general-purpose read/write workloads.

### 1️⃣6️⃣ ConcurrentSkipListMap (The Concurrent Sibling)

* **🧠 Theory:** There is no "ConcurrentTreeMap" in Java. If you need a sorted, concurrent map, you must use `ConcurrentSkipListMap`.
* **How it works:** Instead of a complex Red-Black tree (which is a nightmare to lock effectively because a single rotation can affect the whole tree), it uses a **Skip List**. A Skip List is a series of linked lists stacked on top of each other with "express lanes" to skip nodes. It provides the same $O(\log n)$ performance but is vastly easier to make thread-safe using lock-free CAS operations.

### 1️⃣7️⃣ Cache Locality and Heap Fragmentation

* **🧠 Theory:** Because every `put()` instantiates a new `Entry` object, a long-lived `TreeMap` will have its nodes scattered completely randomly across the JVM Heap.
* **The Impact:** Iterating over a `TreeMap` causes constant CPU Cache Misses because the hardware cannot pre-fetch contiguous memory blocks (like it can with an `ArrayList`). Despite $O(\log n)$ math, a `TreeMap` is mechanically slower at the hardware level than array-backed structures for bulk operations.

```

---

**This effectively concludes the deep dives into the core Java Collections framework!** Since you are preparing for a 9-year experience interview, the next logical step is **System Design / GoF Design Patterns**. 

**Would you like me to start creating your GitLab notes for Design Patterns, beginning with the Strategy Pattern or the Factory Pattern?**

```