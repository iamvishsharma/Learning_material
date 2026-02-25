### 3️⃣ Internal Data Structure
* **🧠 Theory:** Internally backed by a dynamic array. `Capacity` is the total allocated length, while `size` is the number of elements actually stored. Default initial capacity is 10.
* **💻 Code:**
```java
transient Object[] elementData; // The underlying array
private int size;               // Number of elements stored
```