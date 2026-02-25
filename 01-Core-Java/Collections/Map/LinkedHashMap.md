# 05 LinkedHashMap

## Notes

# 📘 Complete Java LinkedHashMap Reference

### 1️⃣ Overview
* **🧠 Theory:** `LinkedHashMap` is an implementation of the `Map` interface that extends `HashMap`. It maintains a **doubly-linked list** running through all of its entries. This defines the iteration ordering, which is normally the order in which keys were inserted (**Insertion Order**).
* **Key Feature:** Unlike `HashMap`, the order is predictable.

### 2️⃣ Class Hierarchy
* **🧠 Theory:** It extends `HashMap` and implements `Map`. It reuses most of `HashMap`'s non-structural logic.
* **💻 Code (Architecture):**
  ```text
  Object
    ↳ AbstractMap<K,V>
         ↳ HashMap<K,V>
              ↳ LinkedHashMap<K,V>

```

### 3️⃣ Internal Data Structure (The Hybrid)

* **🧠 Theory:** It uses the exact same hash table (buckets) as `HashMap` for fast $O(1)$ access. **However**, every node also has `before` and `after` pointers that connect it to the node inserted previously and the node inserted subsequently.
* **💻 Code:**
```java
// Extends the standard HashMap Node to add pointers
static class Entry<K,V> extends HashMap.Node<K,V> {
    Entry<K,V> before, after;

    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}

// Pointers to the head (eldest) and tail (youngest) of the list
transient LinkedHashMap.Entry<K,V> head;
transient LinkedHashMap.Entry<K,V> tail;

```



### 4️⃣ 🔬 Deep Dive: Internal Working Explained Simply

Imagine a standard `HashMap`, but with a "Red String" connecting every single item.

1. **The Bucket (Vertical):** When you call `put("A", 1)`, it calculates the hash and drops "A" into a bucket (just like `HashMap`). This gives you $O(1)$ lookup speed.
2. **The List (Horizontal):** Simultaneously, it updates the `tail` pointer. "A" points back to the previous item, and the previous item points forward to "A".
3. **The Result:** You have a collection that *looks* like a Hash Table for speed, but *acts* like a LinkedList for iteration.

### 5️⃣ Constructors & Ordering Modes

* **🧠 Theory:** You can configure `LinkedHashMap` to maintain **Access Order** instead of Insertion Order. This is the foundation of LRU Caches.
* **💻 Code:**
```java
// 1. Default (Insertion Order)
LinkedHashMap<String, String> map = new LinkedHashMap<>();

// 2. Access Order Enabled (Crucial for LRU)
// accessOrder = true: Last accessed item moves to the end of the list
LinkedHashMap<String, String> map = new LinkedHashMap<>(16, 0.75f, true);

```



### 6️⃣ Time Complexity

* **🧠 Theory:**
* **Get/Put:** $O(1)$ - Same as `HashMap`.
* **Iteration:** $O(N)$ - where $N$ is the number of elements.
* **Performance Note:** Iteration is actually **faster** than `HashMap` if the map is sparse (large capacity, few elements), because it follows the linked list and skips empty buckets.


* **💻 Code (Operations):**
* `get(key)` → **O(1)**
* `put(key, value)` → **O(1)** (slightly slower constant time than HashMap due to linking pointers)
* `containsKey(key)` → **O(1)**



### 7️⃣ Fail-Fast Behavior

* **🧠 Theory:** Like all standard `java.util` collections, iterators are fail-fast. Modifying the map structure (adding/removing keys) while iterating throws `ConcurrentModificationException`.
* **Note:** In **Access Order** mode, simply calling `get()` modifies the structure (moves the item to the end). Calling `get()` while iterating with a standard Iterator **will** throw an exception!

### 8️⃣ Thread Safety

* **🧠 Theory:** `LinkedHashMap` is **not** thread-safe.
* **💻 Code:**
```java
// Synchronized Wrapper
Map<K,V> m = Collections.synchronizedMap(new LinkedHashMap<>());

```



### 9️⃣ Insertion Order vs Access Order

* **Insertion Order (Default):** The order keys were added. Updating the value of a key does *not* change its position.
* **Access Order (`true` in constructor):** Every time you call `get(key)` or `put(key, val)`, that entry is moved to the **end** of the list (it becomes the "youngest"). The "eldest" entry is at the head.

### 🔟 Memory Overhead

* **🧠 Theory:** Higher than `HashMap`. Every entry requires two additional references (`before` and `after`) (64 bits * 2 = 128 bits extra per entry on 64-bit JVMs).

### 1️⃣1️⃣ The LRU Cache Hook (`removeEldestEntry`)

* **🧠 Theory:** `LinkedHashMap` has a protected method `removeEldestEntry` that returns `false` by default. If you override it to return `true`, the map will automatically delete the oldest item when a new one is added.
* **💻 Code:**
```java
// Simple LRU Cache Implementation
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return size() > MAX_ENTRIES; // Returns true to delete eldest
}

```



### 1️⃣2️⃣ When to Use LinkedHashMap

* **🧠 Theory:**
✅ When you need a map but strictly require **Insertion Order** (e.g., parsing JSON where key order matters).
✅ When implementing a **Cache** (LRU).
✅ When you need faster iteration over a sparse map.

### 1️⃣3️⃣ When NOT to Use

* **🧠 Theory:**
❌ If you just need mappings and don't care about order (`HashMap` is lighter on memory).
❌ In multi-threaded environments without external synchronization.

---

## 🌟 Senior & Lead Developer Addendum (System Design & Internals)

### 1️⃣4️⃣ Building a Real LRU Cache (Interview Gold)

* **🧠 Theory:** A standard interview question is "Design an LRU Cache". You can do this in Java by extending `LinkedHashMap`.
* **💻 Code (The 5-Line Solution):**
```java
public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    private final int capacity;

    public LRUCache(int capacity) {
        // true = Enable Access Order
        super(capacity, 0.75f, true); 
        this.capacity = capacity;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        // Automatically removes the "head" (oldest) when size exceeds capacity
        return size() > capacity; 
    }
}

```



### 1️⃣5️⃣ Iteration Performance Paradox

* **🧠 Theory:** `LinkedHashMap` adds overhead to insertions (updating pointers), but it speeds up iteration.
* **The Math:**
* `HashMap` Iteration Cost: $O(Capacity + Size)$. If you initialize with Capacity 10,000 but only have 5 items, it still scans 10,000 buckets.
* `LinkedHashMap` Iteration Cost: $O(Size)$. It ignores capacity and just follows the 5 links.
* **Result:** `LinkedHashMap` is efficient for caching and event listeners where iteration happens frequently but the map might be sparse.



### 1️⃣6️⃣ Inheritance & "View" Mechanics

* **🧠 Theory:** `LinkedHashMap` does not rewrite the `put()` method of `HashMap`. Instead, it overrides a small hook method called `newNode()`.
* **Internal Magic:** When `HashMap`'s `put` logic decides to create a node, it calls `newNode`. `LinkedHashMap` intercepts this call, creates its special Double-Linked Node, and attaches it to the list. This is a perfect example of the **Template Method Pattern**.

### 1️⃣7️⃣ Key Mutable Danger

* **🧠 Theory:** Since `LinkedHashMap` relies on `HashMap` for lookups, the standard warning applies: **Never use mutable objects as keys.** If the key's hash code changes after insertion, the map loses track of the bucket, but the Linked List pointers persist, leading to a "Zombie Entry" that exists in the list but cannot be retrieved via `get()`.

```

```