# Default Methods

## Notes

# 📘 Complete Java 8 Default Methods Reference

### 1️⃣ Overview
* **🧠 Theory:** Before Java 8, interfaces could only contain *abstract* methods (signatures without bodies). Java 8 introduced **Default Methods**, allowing developers to add new methods to interfaces *with* an implementation.
* **The Keyword:** They are declared using the `default` keyword.

### 2️⃣ The "Why": Backward Compatibility
* **🧠 Theory:** Why did Java break the 20-year-old rule of interfaces? **Backward Compatibility.**
* **The Problem:** Java 8 wanted to add the `stream()` and `forEach()` methods to the `Collection` interface. If they added them as standard abstract methods, millions of custom collection classes written by developers worldwide would suddenly break (fail to compile) because they didn't implement the new methods.
* **The Solution:** By making them `default` methods, the `Collection` interface could provide a fallback implementation. Old code continued to compile and run perfectly while automatically inheriting the new features.

### 3️⃣ Syntax & Basic Usage
* **🧠 Theory:** A default method is just a regular method inside an interface marked with `default`. Classes implementing the interface can use it as-is, or override it.
* **💻 Code:**
  ```java
  public interface Vehicle {
      void startEngine(); // Abstract method (must be implemented)
      
      // Default method (inherited automatically)
      default void turnOnAlarm() {
          System.out.println("Turning on vehicle alarm...");
      }
  }

  public class Car implements Vehicle {
      @Override
      public void startEngine() {
          System.out.println("Car engine started.");
      }
      // turnOnAlarm() is automatically inherited!
  }

```

### 4️⃣ The Diamond Problem (Multiple Inheritance of Behavior)

* **🧠 Theory:** Because Java classes can implement *multiple* interfaces, what happens if two interfaces provide a default method with the exact same signature? This is known as the **Diamond Problem**.
* **The Rule:** The Java compiler will force you to resolve the conflict manually by overriding the method in your implementation class.
* **💻 Code:**
```java
interface Camera {
    default void record() { System.out.println("Camera recording"); }
}

interface Phone {
    default void record() { System.out.println("Phone recording"); }
}

// ❌ Compilation Error: inherits unrelated defaults for record()
class SmartPhone implements Camera, Phone { 

    // ✅ The Fix: You MUST override the method to resolve ambiguity
    @Override
    public void record() {
        // You can write your own logic, OR explicitly call one of the interfaces:
        Phone.super.record(); 
    }
}

```



### 5️⃣ Resolution Rules (The 3 Golden Rules)

If there is a conflict, the JVM resolves it using these priorities:

1. **Classes always win:** A method defined in a Class or Superclass takes highest priority over any default method.
2. **Sub-interfaces win:** If Interface B extends Interface A, and both provide a default for the same method, Interface B's default wins (it is "more specific").
3. **Manual Override:** If neither 1 nor 2 applies (the interfaces are parallel), the implementing class *must* override the method.

### 6️⃣ Static Methods in Interfaces

* **🧠 Theory:** Along with default methods, Java 8 also allowed `static` methods in interfaces.
* **The Goal:** To eliminate the need for companion utility classes. (e.g., placing utility methods directly inside `Collection` instead of needing a separate `Collections` class).
* **💻 Code:**
```java
public interface MathUtils {
    static int add(int a, int b) {
        return a + b;
    }
}

// Called directly on the interface, NOT the instance
int sum = MathUtils.add(5, 10); 

```



---

## 🌟 Senior & Lead Developer Addendum (Architecture & Gotchas)

### 7️⃣ Interface vs. Abstract Class (The Modern Distinction)

* **🧠 Theory:** If interfaces can now have implemented methods, do we still need Abstract Classes? **Yes.**
* **The Architectural Difference:** **State.**
* **Abstract Classes:** Can hold *State* (instance variables/fields), have constructors, and maintain internal object state.
* **Interfaces:** Are entirely *Stateless*. They cannot hold instance variables (all fields in an interface are implicitly `public static final` constants). They define *Behavior*, not State.


* **Interview Answer:** "Use an abstract class when you need to share state and code among closely related classes. Use an interface with default methods when you need to define a contract and provide optional default behavior across unrelated classes."

### 8️⃣ The `Object` Class Override Restriction

* **🧠 Theory:** Interviewers love asking: *Can you provide a default implementation for `equals()`, `hashCode()`, or `toString()` inside an interface?*
* **The Answer:** **NO. It causes a compilation error.**
* **The "Why":** Remember Golden Rule #1: *Classes always win*. Every class in Java inherits from `java.lang.Object`. Therefore, the `equals()`, `hashCode()`, and `toString()` methods from the `Object` class would *always* take priority over an interface's default method. To prevent developers from writing default methods that literally can never be executed, the Java compiler explicitly forbids it.

### 9️⃣ Interface Evolution vs. "Trait" Pattern

* **🧠 Theory:** Default methods were designed strictly for *interface evolution* (backward compatibility), not for multiple inheritance. However, developers quickly began using them as **Traits** (mixins) to compose behaviors.
* **Example:** Creating interfaces like `Flyable`, `Swimmable`, `Walkable`, each with default methods, and then having a `Duck` class implement all three to compose its behavior without writing boilerplate. While powerful, senior architects caution against overusing this, as it can lead to deeply confusing inheritance hierarchies.

```

---

**Would you like to move into the true powerhouse of Java 8 next: The `Streams API`? Or would you prefer to cover `Optional` to handle nulls gracefully?**

```