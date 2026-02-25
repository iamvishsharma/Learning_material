# 02 Linked List

## Notes

# 📘 Complete Java LinkedList Reference

### 1️⃣ Overview
* **🧠 Theory:** `LinkedList` is a doubly-linked list implementation of the `List` and `Deque` interfaces present in `java.util`. It maintains insertion order, allows duplicates, acts as both a list and a queue, and is **not thread-safe**.

### 2️⃣ Class Hierarchy
* **🧠 Theory:** It extends `AbstractSequentialList` and implements `List<E>`, `Deque<E>` (Double Ended Queue), `Cloneable`, and `Serializable`.
* **💻 Code (Architecture):**
  ```text
  Object
    ↳ AbstractCollection<E>
         ↳ AbstractList<E>
              ↳ AbstractSequentialList<E>
                   ↳ LinkedList<E>

```

### 3️⃣ Internal Data Structure

* **🧠 Theory:** Unlike `ArrayList`, it is NOT backed by an array. Instead, it consists of individual `Node` objects linked together via pointers. It maintains references to the `first` (head) and `last` (tail) nodes.
* **💻 Code:**
```java
// Internal Node class representation
private static class Node<E> {
    E item;        // The actual data
    Node<E> next;  // Pointer to the next node
    Node<E> prev;  // Pointer to the previous node
}

transient int size = 0;
transient Node<E> first;
transient Node<E> last;

```



### 4️⃣ Constructors

* **🧠 Theory:** Since there is no underlying array to size, there is no "initial capacity" constructor. You can only create an empty list or copy from an existing collection.
* **💻 Code:**
```java
// 1. Default (Empty list)
LinkedList<Integer> list = new LinkedList<>();

// 2. From Collection
LinkedList<String> list = new LinkedList<>(existingCollection);

```



### 5️⃣ Node Linkage (Instead of Resizing)

* **🧠 Theory:** `LinkedList` never needs to "resize". When you add an element, it simply creates a new `Node` object in memory and updates the `next` and `prev` pointers of the surrounding nodes. This takes constant time once the position is found.
* **💻 Code:**
```java
// Internal logic for adding to the end (simplified)
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null) first = newNode;
    else l.next = newNode;
    size++;
}

```



### 6️⃣ Time Complexity

* **🧠 Theory:** Adding/removing at the extreme ends is instant. However, retrieving an element by its index is slow because the JVM must sequentially traverse the nodes from the beginning (or end) until it reaches the index.
* **💻 Code (Operations):**
* `addFirst(E)` / `addLast(E)` → **O(1)**
* `removeFirst()` / `removeLast()` → **O(1)**
* `get(index)` / `set(index)` → **O(n)** *(Technically O(n/2) because it searches from whichever end is closer, but simplifies to O(n))*
* `add(index, e)` / `remove(index)` → **O(n)** *(Time is spent traversing to the index)*



### 7️⃣ Fail-Fast Behavior

* **🧠 Theory:** Just like `ArrayList`, it uses `modCount` to track structural changes. Modifying the links directly during an iteration loop throws an exception.
* **💻 Code:**
```java
// ❌ BAD: Throws ConcurrentModificationException
for(Integer i : list) {
    if (i == 2) list.remove(i); 
}

// ✅ GOOD: Safe deletion using Iterator
Iterator<Integer> it = list.iterator();
while(it.hasNext()) {
    if(it.next() == 5) {
        it.remove(); // Safely unlinks the node
    }
}

```



### 8️⃣ Thread Safety

* **🧠 Theory:** `LinkedList` is strictly single-threaded. If multiple threads alter the pointers simultaneously, the list will become corrupted or broken.
* **💻 Code:**
```java
// Option 1: Synchronized Wrapper
List<Integer> list = Collections.synchronizedList(new LinkedList<>());

// Option 2: Better alternatives for concurrent Queues/Deques
ConcurrentLinkedQueue<Integer> queue = new ConcurrentLinkedQueue<>();
ConcurrentLinkedDeque<Integer> deque = new ConcurrentLinkedDeque<>();

```



### 9️⃣ Deque & Queue Operations

* **🧠 Theory:** Because it implements `Deque`, `LinkedList` is frequently used as a Queue (FIFO) or a Stack (LIFO).
* **💻 Code:**
```java
// Queue operations (FIFO)
list.offer("A"); // Adds to tail
list.poll();     // Retrieves and removes from head
list.peek();     // Retrieves head without removing

// Stack operations (LIFO)
list.push("B");  // Pushes onto head
list.pop();      // Pops from head

```



### 🔟 Lack of RandomAccess Marker

* **🧠 Theory:** `LinkedList` does **NOT** implement the `RandomAccess` marker interface. If you pass a `LinkedList` to `Collections.binarySearch()`, the framework detects the missing marker and safely switches to an `Iterator`-based search instead of a standard `for` loop to prevent catastrophic performance drops.

### 1️⃣1️⃣ Memory Behavior

* **🧠 Theory:** Has very **high memory overhead**. Every single element must be wrapped in a `Node` object, requiring extra memory for the object header and two memory pointers (`next` and `prev`). Furthermore, nodes are scattered randomly across heap memory, leading to **poor CPU cache locality**.

### 1️⃣2️⃣ When to Use LinkedList

* **🧠 Theory:**
✅ When implementing a Queue or Deque.
✅ Applications with frequent insertions/deletions at the very beginning or end of the collection.
✅ When the total size of the collection is entirely unknown and you want to avoid arbitrary array resizing overhead.

### 1️⃣3️⃣ When NOT to Use

* **🧠 Theory:**
❌ When memory efficiency is required (due to node overhead).
❌ Heavy read operations or frequent access by index.
❌ When CPU cache performance and fast iteration are critical.

---

## 🌟 Senior & Lead Developer Addendum (JVM & System Internals)

### 1️⃣4️⃣ Garbage Collection (GC) Impact & Object Churn

* **🧠 Theory (The Senior Perspective):** `LinkedList` is notorious for causing high memory churn. Every single time you call `.add()`, the JVM must allocate a new `Node` object in the **Eden Space** of the heap. In high-throughput applications, this rapidly fills the Eden space, triggering frequent **Minor GC (Garbage Collection)** pauses.
* By contrast, `ArrayList` only creates garbage when it resizes (discarding the old array). For long-lived data, `ArrayList` is vastly more GC-friendly.

### 1️⃣5️⃣ Parallel Streams & Spliterators (Java 8+)

* **🧠 Theory:** If you use `.parallelStream()`, the data structure must be split into chunks for different CPU threads. `ArrayList` dominates here.
* **💻 Mechanics:** `ArrayList` uses an index-based `Spliterator`. It can look at a list of 10,000 items and instantly divide it down the middle in $O(1)$ time by passing index ranges to different threads. `LinkedList` has a terrible `Spliterator`; to split it in half, it must physically traverse the nodes one by one ($O(n)$), completely bottlenecking the parallelization.

### 1️⃣6️⃣ The "Middle Insertion Myth" (Mechanical Sympathy)

* **🧠 Theory:** Textbooks claim `LinkedList` is better for middle insertions because re-linking nodes is $O(1)$, while `ArrayList` must shift elements ($O(n)$). **In the real world, this is often false.**
* **💻 Mechanics:** To insert into the middle of a `LinkedList`, you still have to *traverse* to the middle ($O(n)$), suffering cache misses the whole way. `ArrayList` shifts elements using `System.arraycopy()`, which is a highly optimized, native C++ method that moves contiguous blocks of memory directly in the L1/L2 CPU cache. For lists up to a few thousand elements, `ArrayList` middle insertions are frequently much faster than `LinkedList` in practical benchmarks.

### 1️⃣7️⃣ Custom Serialization (The `transient` Trick)

* **🧠 Theory:** As noted, `ArrayList` marks its internal array as `transient`. Senior engineers should know this is an implementation of the **Serialization Proxy Pattern**.
* **💻 Code:** `ArrayList` implements `private void writeObject(java.io.ObjectOutputStream s)`. Instead of serializing the massive underlying array (which might have thousands of empty slots), it writes out a single `int` for the capacity, and then loops through to serialize *only* the populated elements, saving massive amounts of network bandwidth and disk space.

```

---
