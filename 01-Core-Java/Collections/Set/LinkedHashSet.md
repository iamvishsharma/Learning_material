# 09 LinkedHashSet

## Notes

# 📘 Complete Java LinkedHashSet Reference

### 1️⃣ Overview
* **🧠 Theory:** `LinkedHashSet` is an ordered version of `HashSet`. It maintains a **doubly-linked list** across all elements. This guarantees that the **iteration order** is the same as the **insertion order**.
* **Key Difference:** unlike `HashSet` (random order) and `TreeSet` (sorted order), `LinkedHashSet` keeps elements exactly as they were added.

### 2️⃣ Class Hierarchy
* **🧠 Theory:** It extends `HashSet` and implements `Set<E>`, `Cloneable`, and `Serializable`.
* **💻 Code (Architecture):**
  ```text
  Object
    ↳ AbstractCollection<E>
         ↳ AbstractSet<E>
              ↳ HashSet<E>
                   ↳ LinkedHashSet<E>

```

### 3️⃣ Internal Data Structure (The Hybrid)

* **🧠 Theory:** It uses a **Hash Table** for fast $O(1)$ lookups, combined with a **Doubly-Linked List** for order.
* **The Secret:** It is powered entirely by a `LinkedHashMap` underneath (just like `HashSet` uses `HashMap`).
* **The Node:** Every entry has the standard hash fields (`hash`, `key`, `next`) plus two extra pointers: `before` and `after`.

### 4️⃣ Constructors

* **🧠 Theory:** Calls the super constructor in `HashSet`, which initializes the backing map as a `LinkedHashMap` instead of a standard `HashMap`.
* **💻 Code:**
```java
// 1. Default (Capacity 16, Load Factor 0.75)
Set<String> set = new LinkedHashSet<>();

// 2. From Collection (Deduplicates while keeping order)
List<String> rawList = Arrays.asList("A", "B", "A", "C");
Set<String> deduped = new LinkedHashSet<>(rawList); 
// Result: "A", "B", "C" (Order preserved)

```



### 5️⃣ Insertion Order Guarantee

* **🧠 Theory:**
* **New Element:** Added to the end of the linked list.
* **Re-inserting Existing:** If you add an element that already exists (`set.add("A")` when "A" is there), the order **does not change**. The existing position is preserved.
* **Removal:** Removing an element unlinks it from the list. Re-adding it later puts it at the end.



### 6️⃣ Time Complexity

* **🧠 Theory:**
* **Add/Remove/Contains:** $O(1)$ - Same as `HashSet`.
* **Performance Note:** Operations are slightly slower than `HashSet` (constant time overhead) because it must update the `before` and `after` pointers.



### 7️⃣ Fail-Fast Behavior

* **🧠 Theory:** Like all standard collections, the `Iterator` is fail-fast. Modifying the set structure (adding/removing) during iteration throws `ConcurrentModificationException`.

### 8️⃣ Thread Safety

* **🧠 Theory:** `LinkedHashSet` is **not** thread-safe.
* **💻 Code:**
```java
// Synchronized Wrapper
Set<String> s = Collections.synchronizedSet(new LinkedHashSet<>());

```



### 9️⃣ Memory Behavior (The Trade-off)

* **🧠 Theory:** Has a **higher memory footprint** than `HashSet`.
* **The Cost:** Every single element requires two extra references (`before` and `after`). On a 64-bit JVM, this adds 16 bytes (or 8 bytes with compressed oops) per element compared to a standard `HashSet`.

### 🔟 When to Use LinkedHashSet

* **🧠 Theory:**
✅ When you need to **deduplicate** a list but strictly maintain the original sequence.
✅ When you need a cache where iteration order matters (e.g., oldest inserted items first).
✅ When iterating over the set is a frequent operation (see Senior Addendum).

### 1️⃣1️⃣ When NOT to Use

* **🧠 Theory:**
❌ If memory is tight (use `HashSet`).
❌ If you need sorting (use `TreeSet`).
❌ If order is completely irrelevant (use `HashSet` for slightly faster insertions).

---

## 🌟 Senior & Lead Developer Addendum (System Design & Internals)

### 1️⃣2️⃣ The "Package-Private" Constructor Hack

* **🧠 Theory:** How does `LinkedHashSet` reuse `HashSet` logic without rewriting it?
* **💻 Implementation:** `HashSet` has a specific, package-private constructor designed *only* for `LinkedHashSet`.
```java
// Inside HashSet.java
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}

```


* **Why this matters:** It shows the tight coupling between these classes in the JDK. `LinkedHashSet` is essentially a "config option" for `HashSet`.

### 1️⃣3️⃣ The Iteration Performance Paradox

* **🧠 Theory:** This is a favorite senior interview nuance.
* **HashSet Iteration:** $O(Capacity + Size)$. If you have a huge capacity (10,000) but only 1 element, `HashSet` iterates slowly because it must scan all 10,000 empty buckets to find that 1 item.
* **LinkedHashSet Iteration:** $O(Size)$. It ignores the empty buckets and just follows the linked list.
* **Conclusion:** For **sparse sets** (high capacity, low population), `LinkedHashSet` iteration is significantly faster than `HashSet`.



### 1️⃣4️⃣ Implementation of `Spliterator` (ORDERED characteristic)

* **🧠 Theory:** When using Java Streams (`set.stream()`), the `Spliterator` for `LinkedHashSet` reports the `ORDERED` characteristic.
* **Impact:** `HashSet` reports `DISTINCT`. `LinkedHashSet` reports `DISTINCT | ORDERED`. This implies that stream operations like `.limit()` or `.skip()` are deterministic on a `LinkedHashSet`, whereas on a `HashSet`, `.findFirst()` essentially becomes "find any".

```

```