# 03 Vector

## Notes

# 📘 Complete Java Vector Reference

### 1️⃣ Overview
* **🧠 Theory:** `Vector` is a legacy class (introduced in JDK 1.0, before the Collections Framework) that implements a growable array of objects. It is very similar to `ArrayList` but is **synchronized** (thread-safe) by default.
* **Note:** It was "retrofitted" in Java 1.2 to implement the `List` interface.

### 2️⃣ Class Hierarchy
* **🧠 Theory:** It extends `AbstractList` and implements `List<E>`, `RandomAccess`, `Cloneable`, and `Serializable`.
* **💻 Code (Architecture):**
  ```text
  Object
    ↳ AbstractCollection<E>
         ↳ AbstractList<E>
              ↳ Vector<E>

```

### 3️⃣ Internal Data Structure

* **🧠 Theory:** Like `ArrayList`, it is backed by an `Object[]`. However, it includes a specific field `capacityIncrement` to control growth size, which `ArrayList` lacks.
* **💻 Code:**
```java
protected Object[] elementData;   // The buffer
protected int elementCount;       // The size
protected int capacityIncrement;  // Growth step (if 0, it doubles)

```



### 4️⃣ Constructors

* **🧠 Theory:** You can specify an initial capacity *and* a "capacity increment". This allows fine-grained control over how the array grows.
* **💻 Code:**
```java
// 1. Default (Capacity 10)
Vector<String> v = new Vector<>();

// 2. Capacity + Increment (Unique to Vector)
// Starts at 20. When full, adds exactly 5 slots (20 -> 25 -> 30).
Vector<String> v = new Vector<>(20, 5);

```



### 5️⃣ Resizing Mechanism (2x Growth)

* **🧠 Theory:** This is a major difference from `ArrayList`.
* If `capacityIncrement` is defined (> 0), it grows by that amount.
* If `capacityIncrement` is 0 (default), it **Doubles (2x)** its size.
* *Comparison:* `ArrayList` grows by 1.5x. `Vector` is more aggressive with memory.


* **💻 Code:**
```java
int newCapacity = oldCapacity + ((capacityIncrement > 0) ? 
                                 capacityIncrement : oldCapacity);

```



### 6️⃣ Time Complexity

* **🧠 Theory:** Theoretically the same as `ArrayList` ($O(1)$ for get/add), but in practice, every single operation has the overhead of acquiring and releasing a **synchronization lock**.
* **💻 Code (Operations):**
* `get(index)` / `add(e)` → **O(1)** (plus lock cost)
* `contains(e)` / `remove(e)` → **O(n)**



### 7️⃣ Fail-Fast Behavior

* **🧠 Theory:** Despite being thread-safe for individual operations, its Iterators are **fail-fast**. If the vector is structurally modified after the Iterator is created, it throws an exception.
* **💻 Code:**
```java
// Throws ConcurrentModificationException
Iterator<String> it = v.iterator();
v.add("New"); // Modification outside iterator
it.next();    // Crash

```



### 8️⃣ Thread Safety (Synchronized Methods)

* **🧠 Theory:** `Vector` is thread-safe because almost every method is modified with the `synchronized` keyword. This locks the entire object for every single operation, which is a performance bottleneck in modern multi-core systems.
* **💻 Code:**
```java
// Source code snippet example
public synchronized void addElement(E obj) {
    modCount++;
    add(obj, elementData, elementCount);
}

```



### 9️⃣ Enumeration (Legacy Iterator)

* **🧠 Theory:** Before `Iterator` existed (Java 1.2), `Vector` used `Enumeration`. It is functionally similar to Iterator but has shorter method names and does **not** support `remove()`.
* **💻 Code:**
```java
Enumeration<String> e = v.elements();
while(e.hasMoreElements()) {
    System.out.println(e.nextElement());
}

```



### 🔟 RandomAccess Marker Interface

* **🧠 Theory:** Like `ArrayList`, `Vector` implements `RandomAccess`. Algorithms like `Collections.binarySearch` will use indexed loops rather than iterators.

### 1️⃣1️⃣ Memory Behavior

* **🧠 Theory:** Because it doubles in size (100% growth) vs `ArrayList` (50% growth), `Vector` can waste more memory if the list is large but not fully populated after a resize.

### 1️⃣2️⃣ When to Use Vector

* **🧠 Theory:**
✅ **Almost never** in new development.
✅ When maintaining legacy code (pre-Java 1.2).
✅ If you need the specific "capacity increment" logic (rare).

### 1️⃣3️⃣ When NOT to Use

* **🧠 Theory:**
❌ **Always prefer `ArrayList**` for non-threaded scenarios.
❌ **Always prefer `CopyOnWriteArrayList**` or `Collections.synchronizedList()` for threaded scenarios, as they offer more modern locking strategies.

---

## 🌟 Senior & Lead Developer Addendum (Legacy & Internals)

### 1️⃣4️⃣ The "Compound Operation" Trap

* **🧠 Theory:** Just because `Vector` methods are synchronized doesn't mean your *logic* is thread-safe. A "Check-Then-Act" sequence is still dangerous without an external lock.
* **💻 Code (The Race Condition):**
```java
// Even with Vector, this is NOT safe!
// Thread A and B could both pass the 'if' before either calls 'add'
if (!vector.contains("A")) {
    vector.add("A"); 
}

// FIX: Must synchronize the block manually
synchronized(vector) {
    if (!vector.contains("A")) { ... }
}

```



### 1️⃣5️⃣ Why is `Stack` a Vector? (Design Flaw)

* **🧠 Theory:** The `java.util.Stack` class extends `Vector`. This is widely considered a design mistake (Inheritance instead of Composition).
* **The Problem:** Because `Stack` *is-a* `Vector`, you can call `stack.add(0, "element")` (inserting at the bottom), breaking the LIFO (Last-In-First-Out) contract.
* **Recommendation:** Use `ArrayDeque` for stack operations instead.

### 1️⃣6️⃣ Performance: Monitor Overhead

* **🧠 Theory:** Every call to a synchronized method involves acquiring the intrinsic lock (monitor). Even with "Biased Locking" optimizations in the JVM, this adds CPU cycle overhead and prevents multiple threads from reading simultaneously. `CopyOnWriteArrayList` or `ReentrantReadWriteLock` are vastly superior for read-heavy workloads.

### 1️⃣7️⃣ The Enumeration vs Iterator Fail-Safe Difference

* **🧠 Theory:** `Enumeration` is **not** fail-fast. If one thread modifies the Vector while another is traversing via Enumeration, it might throw a standard index exception, skip elements, or repeat elements, but it won't explicitly warn you with a `ConcurrentModificationException`. This makes debugging legacy code using Enumerations notoriously difficult.

```

```