Markdown# 01 Array List

## Notes

# 📘 Complete Java ArrayList Reference

### 1️⃣ Overview
`ArrayList` is a resizable array implementation of the `List` interface present in `java.util`.

**It maintains:**
* Insertion order
* Allows duplicates
* Provides fast random access
* Not thread-safe

### 2️⃣ Class Hierarchy
```text
Object
  ↳ AbstractCollection<E>
       ↳ AbstractList<E>
            ↳ ArrayList<E>
Implements: List<E>, RandomAccess, Cloneable, Serializable
3️⃣ Internal Data StructureInternally backed by a dynamic array:Javatransient Object[] elementData;
private int size;
elementData → underlying arraysize → number of elements actually storedCapacity → length of underlying arrayDefault initial capacity = 10
4️⃣ ConstructorsDefault ConstructorJavaArrayList<Integer> list = new ArrayList<>();
Creates an empty list with a default capacity of 10.With Initial CapacityJavaArrayList<Integer> list = new ArrayList<>(20);
Avoids frequent resizing if the size is known in advance.From CollectionJavaArrayList<String> list = new ArrayList<>(existingCollection);
Copies elements from a given collection.
5️⃣ Resizing Mechanism (Very Important)When capacity is exceeded, it grows by 1.5x:JavanewCapacity = oldCapacity + (oldCapacity >> 1)
Example:10 → 1515 → 2222 → 33Resize involves:Creating a new array.Copying old elements (Arrays.copyOf()).Time complexity during resize → O(n)
6️⃣ Time ComplexityOperationComplexityget(index)O(1)set(index)O(1)add(E e)O(1) amortizedadd(index, e)O(n)remove(index)O(n)contains()O(n)Why is add() O(1) amortized? > Because resizing happens only occasionally. Most adds are instant.
7️⃣ Fail-Fast BehaviorArrayList uses modCount internally. If the list is structurally modified during iteration:→ Throws ConcurrentModificationExceptionException Example:Javafor(Integer i : list) {
    list.add(100); // Throws Exception
}
Safe way:JavaIterator<Integer> it = list.iterator();
while(it.hasNext()) {
    if(it.next() == 5) {
        it.remove();
    }
}
8️⃣ Thread SafetyArrayList is NOT thread-safe.Options:Collections.synchronizedList()JavaList<Integer> list = Collections.synchronizedList(new ArrayList<>());
(Note: Must use a synchronized(list) block during iteration).CopyOnWriteArrayListBetter for concurrent read-mostly scenarios.
9️⃣ Capacity vs Sizesize() → actual number of elements present.capacity → internal array length.Useful methods:Javalist.ensureCapacity(100);
list.trimToSize();
🔟 RandomAccess Marker InterfaceArrayList implements RandomAccess.Meaning: It acts as a marker to indicate that the class is optimized for fast random access (O(1)).Note: LinkedList does NOT implement this because its random access is O(n).
1️⃣1️⃣ Memory BehaviorStores references (not the primitives directly).Uses contiguous memory.Has significantly better cache locality compared to LinkedList.
1️⃣2️⃣ When to Use ArrayList
✅ Heavy read operations
✅ Frequent random access
✅ Append-heavy usage
✅ When memory efficiency is required
1️⃣3️⃣ When NOT to Use
❌ Frequent middle insertions
❌ Frequent deletions
❌ Heavy concurrent modifications