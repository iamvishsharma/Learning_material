# ArrayList in Java

## Theory

ArrayList is a part of the Java Collection Framework. It provides a resizable array, which can be found in the java.util package. The ArrayList class implements the List interface.

It can be used to store a dynamically-sized collection of elements. Unlike arrays, ArrayLists can grow as you add more elements to them.

## Code Example

```java
import java.util.ArrayList;

public class Main {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("Hello");
        list.add("World");
        System.out.println(list);
    }
}
```