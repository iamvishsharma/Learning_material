# 08 HashMap

## Notes

# 📘 Complete Java HashMap Reference

### 1️⃣ Overview
* **🧠 Theory:** `HashMap` is a hash table based implementation of the `Map` interface. It stores data in **Key-Value pairs**. It permits exactly **one `null` key** and multiple `null` values. It makes no guarantees about order, and it is **not thread-safe**.

### 2️⃣ Class Hierarchy
* **🧠 Theory:** It extends `AbstractMap` and implements `Map<K,V>`, `Cloneable`, and `Serializable`.
* **💻 Code (Architecture):**
  ```text
  Object
    ↳ AbstractMap<K,V>
         ↳ HashMap<K,V>

```

### 3️⃣ Internal Data Structure (Buckets & Nodes)

* **🧠 Theory:** Internally, it is an array of `Node<K,V>` objects (often called "buckets"). When collisions occur, these nodes link together to form a Linked List or a Red-Black Tree.
* **💻 Code:**
```java
transient Node<K,V>[] table; // The bucket array (size is ALWAYS a power of 2)

static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next; // Pointer to the next node in case of collision
}

```



### 4️⃣ 🔬 Deep Dive: Internal Working (`put` operation)

When you call `map.put("Key", "Value")`, a highly optimized sequence occurs:

1. **Calculate Hash:** Java calls `"Key".hashCode()`.
2. **Supplemental Hash:** The `HashMap` runs its own `hash()` method to spread the bits out (see Senior Addendum).
3. **Index Calculation:** It uses a bitwise AND operator `(n - 1) & hash` to find the array index.
4. **Placement:** * If the bucket is empty, drop the Node in.
* If occupied (Collision), check if keys are `equals()`. If yes, overwrite the value.
* If keys are different, append to the Linked List.
* If the list length hits 8, convert it to a Red-Black Tree.



### 5️⃣ Constructors & Core Variables

* **🧠 Theory:** You control memory and performance via **Capacity** and **Load Factor**.
* **💻 Code:**
```java
// Default: Capacity 16, Load Factor 0.75
HashMap<String, String> map = new HashMap<>();

// Custom: Will resize when items exceed (32 * 0.80) = 25
HashMap<String, String> map = new HashMap<>(32, 0.80f);

```



### 6️⃣ Resizing Mechanism (`resize()`)

* **🧠 Theory:** When the `size` crosses the `threshold` (Capacity * LoadFactor), the array **doubles** in size. The map then iterates through every single node and recalculates its new bucket index. This is extremely CPU-intensive.

### 7️⃣ Time Complexity

* **🧠 Theory:** Performance is heavily dependent on a good `hashCode()` implementation that prevents collisions.
* **💻 Code (Operations):**
* `get(key)` / `put(key, value)` / `remove(key)` → **O(1)** (Average Case)
* Collisions (Linked List) → **O(n)**
* Collisions (Red-Black Tree) → **O(\log n)**



### 8️⃣ Fail-Fast Behavior

* **🧠 Theory:** It maintains a `modCount`. If the map is structurally modified (keys added or removed) while an Iterator is active, it throws a `ConcurrentModificationException`. Updating an *existing* key's value does not trigger this.

### 9️⃣ Thread Safety

* **🧠 Theory:** Completely thread-unsafe. In Java 7, concurrent resizing could cause an **infinite loop** (a circular linked list). In Java 8+, it causes data loss.
* **💻 Code:**
```java
// Use ConcurrentHashMap for multi-threading!
Map<String, String> safeMap = new ConcurrentHashMap<>();

```



### 🔟 Why Null Keys Work

* **🧠 Theory:** `HashMap` explicitly checks for `null`. If you put a `null` key, it completely bypasses the `hashCode()` calculation and hardcodes the index to `0` (Bucket 0).

### 1️⃣1️⃣ Memory Behavior

* **🧠 Theory:** Has noticeable overhead. It allocates an empty array, and every entry requires a `Node` object. However, because the array holds references, the nodes themselves are scattered across the heap (poor cache locality for iteration compared to `ArrayList`).

### 1️⃣2️⃣ When to Use HashMap

* **🧠 Theory:**
✅ General-purpose caching, lookups, and dictionary needs.
✅ When you need $O(1)$ fast retrieval and do not care about the order of keys.

### 1️⃣3️⃣ When NOT to Use

* **🧠 Theory:**
❌ Multi-threaded environments (`ConcurrentHashMap`).
❌ When you need keys sorted (`TreeMap`).
❌ When you need insertion order (`LinkedHashMap`).

---

## 🌟 Senior & Lead Developer Addendum (System Design & Internals)

### 1️⃣4️⃣ The Power of 2 & The Bitwise Index Hack

* **🧠 Theory:** Why is the `HashMap` capacity *always* a power of 2 (16, 32, 64...)?
* **The Reason:** Modulo arithmetic (`hash % array_length`) is mathematically correct for finding an index, but CPU division is slow. Because the length is a power of 2, Java can use a **bitwise AND mask**: `(n - 1) & hash`.
* **Example:** If size is 16, `n - 1` is 15 (Binary `00001111`). Masking the hash with `1111` instantly extracts the last 4 bits, effectively doing modulo 16 in a single, lightning-fast CPU cycle.

### 1️⃣5️⃣ The Supplemental `hash()` Function (Bit Shifting)

* **🧠 Theory:** Look at the Java 8 source code for `HashMap`. It doesn't just use your object's `hashCode()`. It runs it through this:
```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

```


* **Why?** Because the bucket array is usually small (e.g., 16), the bitwise mask `(n-1) & hash` only looks at the **lowest bits** of the hash code. If multiple objects have different high bits but the same low bits, they would all collide. By XORing (`^`) the top 16 bits with the bottom 16 bits, Java forces the high bits to influence the bucket index, massively reducing collisions for poorly written hash functions.

### 1️⃣6️⃣ Treeification Thresholds (8 and 6)

* **🧠 Theory:** Why does a bucket convert to a Red-Black Tree at 8 items, but doesn't convert back to a Linked List until it drops to 6?
* **The Answer:** **Hysteresis**. If the threshold for both was 8, a map hovering around 8 collisions would constantly treeify and untreeify on every `add` and `remove` operation, destroying CPU performance. The gap prevents this thrashing.

### 1️⃣7️⃣ Immutable Keys (The Golden Rule)

* **🧠 Theory:** You should **never** use a mutable object as a key in a `HashMap`.
* **The Danger:** If you use a custom `Employee` object as a key, insert it, and then change the employee's `name` (which alters their `hashCode`), the `HashMap` will never be able to find the object again. The hash code will point to Bucket X, but the object is physically sitting in Bucket Y. It becomes a permanent memory leak. Always use immutable classes like `String` or `Integer` for keys.

```

---

**Would you like to wrap up the core collections here, or are you ready to jump into the Java Design Patterns (like Strategy, Factory, or Adapter) for the system design portion of your interviews?**

```