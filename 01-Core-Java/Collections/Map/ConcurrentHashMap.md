# 06 ConcurrentHashMap

## Notes

# 📘 Complete Java ConcurrentHashMap Reference

### 1️⃣ Overview
* **🧠 Theory:** `ConcurrentHashMap` is a highly concurrent, thread-safe implementation of the `ConcurrentMap` interface. It provides full concurrency of retrievals and high expected concurrency for updates. Unlike `Hashtable` or `Collections.synchronizedMap`, it **does not lock the entire map** to achieve thread safety.

### 2️⃣ Class Hierarchy
* **🧠 Theory:** It extends `AbstractMap` and implements `ConcurrentMap<K,V>` and `Serializable`.
* **💻 Code (Architecture):**
  ```text
  Object
    ↳ AbstractMap<K,V>
         ↳ ConcurrentHashMap<K,V>

```

### 3️⃣ Internal Data Structure (Java 8+)

* **🧠 Theory:** Like `HashMap`, it uses an array of buckets (bins) containing Linked Lists or Red-Black Trees. However, its internal `Node` class makes heavy use of the `volatile` keyword to ensure that updates made by one thread are instantly visible to other threads without explicit locking.
* **💻 Code:**
```java
// Internal Node relies on volatile for memory visibility
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;          // Thread-safe reads!
    volatile Node<K,V> next; // Thread-safe traversal!
}

transient volatile Node<K,V>[] table;

```



### 4️⃣ 🔬 Deep Dive: Internal Working (Bucket-Level Locking)

When you call `map.put(key, value)` in Java 8+, it does not lock the whole map. It only locks the **specific bucket** you are writing to.

1. **Empty Bucket (Fast Path):** If the calculated bucket is completely empty, it uses a CPU-level **CAS (Compare-And-Swap)** operation to drop the new node in. *No locks are acquired.*
2. **Occupied Bucket (Slow Path):** If there is already a node in the bucket (a collision), it uses the `synchronized` keyword *only on the first node* (the head) of that specific linked list/tree.
3. **The Result:** Thread A can write to Bucket 2 while Thread B simultaneously writes to Bucket 5. Neither blocks the other.

### 5️⃣ Constructors

* **🧠 Theory:** You can set the initial capacity and load factor. Historically (in Java 7), `concurrencyLevel` determined the number of "Segments." In Java 8+, `concurrencyLevel` is mostly kept for backward compatibility and acts as a hint for initial array sizing.
* **💻 Code:**
```java
// 1. Default (Capacity 16)
ConcurrentMap<String, Integer> map = new ConcurrentHashMap<>();

// 2. Full Constructor
// (initialCapacity, loadFactor, concurrencyLevel)
ConcurrentMap<String, Integer> map = new ConcurrentHashMap<>(32, 0.75f, 16);

```



### 6️⃣ Resizing Mechanism (Multi-Threaded Transfer)

* **🧠 Theory:** Resizing a massive hash table is slow. `ConcurrentHashMap` solves this by allowing **multiple threads to help resize simultaneously**. When a thread notices the map is resizing, it grabs a chunk of buckets (a "stride") and helps transfer them to the new array before doing its own work.
* **💻 Code:**
```java
// Internal marker node indicating a bucket has been moved
// Threads seeing this will assist in the transfer process
static final class ForwardingNode<K,V> extends Node<K,V> { ... }

```



### 7️⃣ Time Complexity

* **🧠 Theory:** Similar to `HashMap`, assuming a good hash function.
* **💻 Code (Operations):**
* `get(key)` → **O(1)** (Reads are entirely lock-free!)
* `put(key, value)` → **O(1)** (Average), **O(\log n)** (If bucket is treeified)
* `remove(key)` → **O(1)**



### 8️⃣ Fail-Safe Behavior (Weakly Consistent Iterators)

* **🧠 Theory:** Unlike `HashMap`, `ConcurrentHashMap` iterators will **never** throw a `ConcurrentModificationException`. They are "Weakly Consistent."
* **Meaning:** The iterator will reflect the state of the map at the moment the iterator was created. It *may or may not* reflect updates made after creation, but it will never crash or return the same element twice.

### 9️⃣ Thread Safety (No External Synchronization Needed)

* **🧠 Theory:** All operations are thread-safe. However, compound operations (like "check-then-act") require special atomic methods provided by the `ConcurrentMap` API.
* **💻 Code:**
```java
// ❌ BAD: Race condition! Two threads could pass the null check.
if (map.get(key) == null) { map.put(key, value); }

// ✅ GOOD: Atomic operation.
map.putIfAbsent(key, value);

// ✅ GOOD: Atomic compute (Java 8+)
map.computeIfAbsent(key, k -> createValue());

```



### 🔟 The "No Nulls" Rule (Crucial)

* **🧠 Theory:** **Neither keys nor values can be `null**`. Attempting to put a `null` key or value will throw a `NullPointerException`. (See Senior Addendum for the "why").

### 1️⃣1️⃣ Memory Behavior

* **🧠 Theory:** Higher memory footprint than `HashMap` due to the use of `volatile` variables (which prevent CPU cache optimizations and force main memory flushes) and complex internal tracking structures like `TreeBin` and `ForwardingNode`.

### 1️⃣2️⃣ When to Use ConcurrentHashMap

* **🧠 Theory:**
✅ In multi-threaded applications where a shared cache or registry is heavily read and written to by many threads concurrently.
✅ As a complete replacement for `Hashtable`.

### 1️⃣3️⃣ When NOT to Use

* **🧠 Theory:**
❌ In strict single-threaded scenarios (use `HashMap` to avoid `volatile` overhead).
❌ When you need strict atomicity across multiple distinct maps or collections (requires external locking).

---

## 🌟 Senior & Lead Developer Addendum (System Design & Internals)

### 1️⃣4️⃣ Architectural Shift: Java 7 Segments vs. Java 8 Bucket Locks

* **🧠 Theory:** Interviewers love asking how `ConcurrentHashMap` changed in Java 8.
* **Java 7 (Lock Striping):** The map was divided into 16 "Segments" (like 16 mini HashTables). Locking a segment locked 1/16th of the map. It was better than locking the whole map, but still blocked threads unnecessarily if they hit the same segment.
* **Java 8+ (CAS & Node Locks):** Segments were completely removed. Java 8 synchronizes directly on the *head node of the specific bucket* being modified. If the table size is 1024, there are 1024 separate locks. This massively increases concurrent throughput.



### 1️⃣5️⃣ The `NullPointerException` Mystery (Ambiguous Reads)

* **🧠 Theory:** Why does `HashMap` allow `null` keys/values, but `ConcurrentHashMap` forbids them? It's due to the **Ambiguity Problem** in concurrent environments.
* **💻 The Logic:** If `map.get("A")` returns `null`, does that mean:
1. The key "A" does not exist in the map?
2. The key "A" exists, but its value was explicitly set to `null`?


* In a single-threaded `HashMap`, you can just call `map.containsKey("A")` to check. In a concurrent environment, Thread B might insert "A" in the microsecond between your `get()` and `containsKey()` calls, making the result entirely unreliable. Forbidding `null` completely eliminates this ABA/ambiguity problem.

### 1️⃣6️⃣ `size()` vs `mappingCount()`

* **🧠 Theory:** Keeping a single volatile `size` integer is a massive bottleneck if 100 threads are constantly adding/removing items (they would constantly fight for CPU cache coherence on that one variable).
* **💻 Mechanics:** `ConcurrentHashMap` uses an array of `CounterCell` objects spread across memory. When a thread adds an item, it increments a random `CounterCell`.
* **The Gotcha:** The `size()` method returns an `int`, but a highly concurrent map might exceed `Integer.MAX_VALUE`. In Java 8+, you should always use `map.mappingCount()`, which returns a `long` by summing up all the distributed `CounterCells`.
```java
// Legacy / Interfaces (Might overflow or bottleneck)
int size = map.size(); 

// ✅ Modern Java 8+ Concurrent way
long accurateSize = map.mappingCount(); 

```



### 1️⃣7️⃣ Unsafe and Hardware-Level Operations

* **🧠 Theory:** Under the hood, `ConcurrentHashMap` heavily relies on `sun.misc.Unsafe` (and later `VarHandle` in Java 9+) to perform atomic, hardware-level CPU instructions like Compare-And-Swap (CAS). This allows the map to bypass the JVM's standard locking monitors entirely for its fastest paths, achieving speeds close to native C++ concurrent structures.

```

---

**Would you like me to tackle the `TreeMap` next to complete the Map hierarchy (which dives into sorting and Red-Black trees), or should we move on to Design Patterns?**

```