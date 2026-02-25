# 📘 Complete Java ArrayList Reference

### 1️⃣ Overview
* **🧠 Theory:** `ArrayList` is a resizable array implementation of the `List` interface present in `java.util`. It maintains insertion order, allows duplicates, provides fast random access, and is **not thread-safe**.

### 2️⃣ Class Hierarchy
* **🧠 Theory:** It implements `List<E>`, `RandomAccess` (marker for fast retrieval), `Cloneable`, and `Serializable`.
* **💻 Code (Architecture):**
  ```text
  Object
    ↳ AbstractCollection<E>
         ↳ AbstractList<E>
              ↳ ArrayList<E>

```

### 3️⃣ Internal Data Structure

* **🧠 Theory:** Internally backed by a dynamic array. `Capacity` is the total allocated length, while `size` is the number of elements actually stored. Default initial capacity is 10.

* **💻 Code:**
```java
transient Object[] elementData; // The underlying array
private int size;               // Number of elements stored

```

### 4️⃣ Constructors

* **🧠 Theory:** You can initialize it empty, with a pre-defined capacity to save memory overhead, or by copying another collection.
* **💻 Code:**
```java
// 1. Default (Capacity 10)
ArrayList<Integer> list = new ArrayList<>();

// 2. Initial Capacity (Avoids resizing overhead)
ArrayList<Integer> list = new ArrayList<>(20);

// 3. From Collection
ArrayList<String> list = new ArrayList<>(existingCollection);

```

### 5️⃣ Resizing Mechanism (Very Important)

* **🧠 Theory:** When capacity is exceeded, it creates a new array, copies the old elements over (`O(n)` time), and grows by **1.5x**.
* **💻 Code:**
```java
// Internal growth formula using a bitwise shift
newCapacity = oldCapacity + (oldCapacity >> 1)

```

*(Example: 10 → 15 → 22 → 33)*

### 6️⃣ Time Complexity

* **🧠 Theory:** Adding elements is generally instant, but occasionally triggers a resize, making it an "amortized" cost. Searching requires a linear scan.
* **💻 Code (Operations):**
* `get(index)` / `set(index)` → **O(1)**
* `add(E e)` → **O(1)** amortized
* `add(index, e)` / `remove(index)` → **O(n)**
* `contains()` → **O(n)**


### 7️⃣ Fail-Fast Behavior

* **🧠 Theory:** `ArrayList` uses an internal `modCount` to track structural changes. Modifying the list during a standard loop breaks this count and throws an error.
* **💻 Code:**
```java
// ❌ BAD: Throws ConcurrentModificationException
for(Integer i : list) {
    list.add(100); 
}

// ✅ GOOD: Safe deletion using Iterator
Iterator<Integer> it = list.iterator();
while(it.hasNext()) {
    if(it.next() == 5) {
        it.remove(); 
    }
}

```

### 8️⃣ Thread Safety

* **🧠 Theory:** `ArrayList` is strictly single-threaded. For concurrent access, you must use a wrapper or a specialized concurrent collection.
* **💻 Code:**
```java
// Option 1: Synchronized Wrapper (Requires synchronized block on iteration)
List<Integer> list = Collections.synchronizedList(new ArrayList<>());

// Option 2: Concurrent Collection (Great for read-heavy systems)
List<Integer> list = new CopyOnWriteArrayList<>();

```

### 9️⃣ Capacity vs Size

* **🧠 Theory:** `size()` returns populated slots; `capacity` is the hidden array length. You can manually manipulate capacity to optimize memory.
* **💻 Code:**
```java
list.ensureCapacity(100); // Forces growth to 100 before bulk additions
list.trimToSize();        // Shrinks underlying array to match current size

```

### 🔟 RandomAccess Marker Interface

* **🧠 Theory:** `ArrayList` implements `RandomAccess`. This acts as a marker telling the JVM and other frameworks that this class is optimized for fast, constant-time `O(1)` random access. *(Note: `LinkedList` does NOT implement this).* 

### 1️⃣1️⃣ Memory Behavior

* **🧠 Theory:** Stores object *references*, not primitives. Because it uses a contiguous block of memory, it benefits from high CPU cache locality, making iterations noticeably faster than node-based structures.

### 1️⃣2️⃣ When to Use ArrayList

* **🧠 Theory:**
✅ Heavy read operations
✅ Frequent random access
✅ Append-heavy usage
✅ When memory efficiency is required

### 1️⃣3️⃣ When NOT to Use

* **🧠 Theory:**
❌ Frequent middle insertions
❌ Frequent deletions
❌ Heavy concurrent modifications