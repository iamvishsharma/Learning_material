# 04 Hash Set

## Notes

# 📘 Complete Java HashSet Reference

### 1️⃣ Overview
* **🧠 Theory:** `HashSet` is an implementation of the `Set` interface backed by a hash table. It guarantees **no duplicate elements**, permits exactly **one `null` element**, makes **no guarantees about insertion order**, and is **not thread-safe**.

### 2️⃣ Class Hierarchy
* **🧠 Theory:** It extends `AbstractSet` and implements `Set<E>`, `Cloneable`, and `Serializable`.
* **💻 Code (Architecture):**
  ```text
  Object
    ↳ AbstractCollection<E>
         ↳ AbstractSet<E>
              ↳ HashSet<E>

```

### 3️⃣ Internal Data Structure (The Secret)

* **🧠 Theory:** A `HashSet` is literally just a wrapper around a `HashMap`. The elements you add to the `HashSet` become the **keys** in the underlying `HashMap`. Since a Map requires key-value pairs, `HashSet` uses a dummy, constant `Object` called `PRESENT` as the value for every key.
* **💻 Code:**
```java
private transient HashMap<E,Object> map;

// Dummy value to associate with an Object in the backing Map
private static final Object PRESENT = new Object();

```



### 4️⃣ 🔬 Deep Dive: Internal Working Explained Simply

To understand `HashSet`, you must understand how it physically places your data in memory. Here is the exact step-by-step process when you call `set.add("Apple")`:

1. **The Hash Code:** Java calls `"Apple".hashCode()`. It returns a large integer (e.g., `63055984`).
2. **The Index (Bitmasking):** The underlying array (bucket array) is small (default size 16). Java uses a bitwise AND operation `(n - 1) & hash` to instantly shrink that massive hash code down to a valid array index (e.g., Index `4`).
3. **The Bucket:** Java goes to Index `4` in the array.
* *If it's empty:* It creates a `Node` holding your `"Apple"` and drops it in.
* *If it's NOT empty (A Collision):* Suppose `"Banana"` is already at Index `4`. Java calls `"Apple".equals("Banana")`.
* If `true`: It rejects the addition (no duplicates allowed).
* If `false`: It attaches `"Apple"` to the end of `"Banana"`, creating a **Linked List** inside that single bucket.




4. **Treeification:** If that Linked List gets too long (8 or more items), Java 8+ automatically converts it into a **Red-Black Tree** to keep search times blazingly fast.

### 5️⃣ Constructors

* **🧠 Theory:** Constructors initialize the backing `HashMap`. You can define the **initial capacity** (default 16) and the **load factor** (default 0.75).
* **💻 Code:**
```java
// 1. Default (Capacity 16, Load Factor 0.75)
HashSet<String> set = new HashSet<>();

// 2. Custom Capacity and Load Factor
// Will resize when size > (100 * 0.50) = 50 elements
HashSet<String> set = new HashSet<>(100, 0.50f);

```



### 6️⃣ Resizing Mechanism (Load Factor)

* **🧠 Theory:** Resizing is handled entirely by the backing `HashMap`. When the number of elements exceeds the `capacity * loadFactor`, the underlying array **doubles** in size (e.g., 16 → 32). The elements are then "rehashed" into the new array.
* **💻 Code:**
```java
// Threshold formula
threshold = capacity * loadFactor;

```



### 7️⃣ Time Complexity

* **🧠 Theory:** Assuming a good hash function that disperses elements properly across buckets, operations are incredibly fast. If there are severe hash collisions, performance degrades.
* **💻 Code (Operations):**
* `add(e)` / `remove(e)` / `contains(e)` → **O(1)** (Average Case)
* `add(e)` / `remove(e)` / `contains(e)` → **O(n)** or **O(\log n)** (Worst Case - Heavy Collisions)



### 8️⃣ Fail-Fast Behavior

* **🧠 Theory:** Iterators are fail-fast, relying on the `modCount` of the underlying `HashMap`. Modifying the set while iterating throws an exception.
* **💻 Code:**
```java
// ✅ GOOD: Safe deletion using Iterator
Iterator<String> it = set.iterator();
while(it.hasNext()) {
    if(it.next().equals("RemoveMe")) {
        it.remove(); 
    }
}

```



### 9️⃣ Thread Safety

* **🧠 Theory:** `HashSet` is strictly single-threaded. Concurrent modification can result in data corruption or infinite loops during resizing.
* **💻 Code:**
```java
// Option 1: Synchronized Wrapper
Set<String> syncSet = Collections.synchronizedSet(new HashSet<>());

// Option 2: Modern Concurrent Alternative (Java 8+)
Set<String> concurrentSet = ConcurrentHashMap.newKeySet();

```



### 🔟 No Random Access or Indexing

* **🧠 Theory:** Because elements are placed into buckets based on their hash code, the concept of an "index" does not exist. You cannot do `set.get(0)`. You must use an `Iterator` or an enhanced `for-loop`.

### 1️⃣1️⃣ Memory Behavior

* **🧠 Theory:** `HashSet` has a very **high memory overhead**. For every element you add, the JVM must allocate memory for:
1. Your actual object.
2. The `HashMap.Node` wrapper object (which holds the hash, key, value, and next pointer).
3. The `PRESENT` dummy object reference.



### 1️⃣2️⃣ When to Use HashSet

* **🧠 Theory:**
✅ When you need a collection with **no duplicates**.
✅ When you need extremely fast O(1) lookups (e.g., checking if an item exists).
✅ When you do not care about the order of elements.

### 1️⃣3️⃣ When NOT to Use

* **🧠 Theory:**
❌ When you need to maintain insertion order (Use `LinkedHashSet`).
❌ When you need the elements sorted naturally (Use `TreeSet`).
❌ In highly memory-constrained environments.

---

## 🌟 Senior & Lead Developer Addendum (JVM & System Internals)

### 1️⃣4️⃣ The `equals()` and `hashCode()` Contract

* **🧠 Theory:** This is the most frequently asked senior interview question regarding Sets. If you put a custom object into a `HashSet`, you **must** override both `equals()` and `hashCode()`.
* **The Rule:** 1. If `obj1.equals(obj2)` is true, their `hashCode()` **must** be identical.
2. If their `hashCode()` is identical, they do *not* necessarily have to be `equals()` (this is just a collision).
* **The Gotcha:** If you override `equals` but forget `hashCode`, two logically identical objects will hash to different buckets, and the `HashSet` will allow duplicates, breaking the fundamental rule of a `Set`.

### 1️⃣5️⃣ Why is `PRESENT` an Object and not `null`?

* **🧠 Theory:** You might wonder why `HashSet` doesn't just pass `null` as the value to the `HashMap` to save memory.
* **The Answer:** `HashMap` allows `null` values. The `put()` method returns `null` if a key was absent, OR if the key was present but its value was `null`. If `HashSet` used `null` as the dummy value, it wouldn't be able to tell the difference between "this key is new" and "this key was already here," breaking the `add(E e)` boolean return logic.

### 1️⃣6️⃣ The `LinkedHashSet` Optimization (Iteration Speed)

* **🧠 Theory:** Iterating over a standard `HashSet` requires scanning the *entire* backing array (capacity), even if it's mostly empty.
* **The Alternative:** `LinkedHashSet` maintains a doubly-linked list running through all inserted elements. Iterating over a `LinkedHashSet` is proportional *only* to the number of elements (size), ignoring the array capacity. For iteration-heavy workloads on sparse sets, `LinkedHashSet` is faster.

```