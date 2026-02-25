# CompletableFuture

## Notes

# 📘 Complete Java 8+ CompletableFuture Reference

### 1️⃣ Overview
* **🧠 Theory:** `CompletableFuture` is an advanced implementation of the `Future` interface introduced in Java 8. It represents a future result of an asynchronous computation.
* **The "Why":** The old `Future` interface was limited—you couldn't manually complete it, you couldn't chain tasks together, and `future.get()` was a blocking call. `CompletableFuture` solves this by being **Non-Blocking** and **Composable**.

### 2️⃣ Creating & Running Tasks
* **🧠 Theory:** You can run tasks that don't return a value (`runAsync`) or tasks that do (`supplyAsync`).
* **💻 Code:**
  ```java
  // 1. Returns a result
  CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
      return "Data from API";
  });

  // 2. Performs an action only
  CompletableFuture<Void> run = CompletableFuture.runAsync(() -> {
      System.out.println("Background task running");
  });

```

### 3️⃣ Chaining Callbacks (Non-Blocking)

* **🧠 Theory:** Instead of waiting for a result, you attach "callbacks" that trigger automatically when the task completes.
* **Common Methods:**
* `thenApply(Function)`: Transforms the result (Like `map`).
* `thenAccept(Consumer)`: Consumes the result (returns `void`).
* `thenRun(Runnable)`: Executes an action regardless of the result.


* **💻 Code:**
```java
CompletableFuture.supplyAsync(() -> "User123")
    .thenApply(id -> fetchUserEmail(id)) // Transforms ID to Email
    .thenAccept(email -> sendWelcomeEmail(email)); // Sends the email

```



### 4️⃣ Combining Multiple Futures

* **🧠 Theory:** This is the most powerful feature for Senior devs—orchestrating multiple async calls.
* **Combiners:**
* `thenCompose()`: Used when the second task depends on the first (Flattening).
* `thenCombine()`: Used when two independent tasks need to be merged.
* `allOf()`: Waits for multiple futures to finish.
* `anyOf()`: Returns as soon as the first future in a list finishes.



### 5️⃣ Exception Handling

* **🧠 Theory:** In async programming, you can't use standard `try-catch` across threads. `CompletableFuture` provides specialized methods to handle errors.
* **💻 Code:**
```java
CompletableFuture.supplyAsync(() -> { throw new RuntimeException("DB Fail"); })
    .exceptionally(ex -> "Fallback Data") // Recovers from error
    .handle((res, ex) -> { // Handles both result and error
        if(ex != null) return "Error Handled";
        return res;
    });

```



---

## 🌟 Senior & Lead Developer Addendum (Concurrency & Tuning)

### 6️⃣ Thread Pool Management (The ForkJoinPool)

* **🧠 Theory:** By default, `CompletableFuture` uses the **ForkJoinPool.commonPool()**.
* **Lead Perspective:** Never use the common pool for **I/O bound tasks** (DB calls, API requests) in production. If one task blocks, it can starve all other parts of your app using the common pool.
* **The Fix:** Always pass a custom `Executor`.
```java
Executor executor = Executors.newFixedThreadPool(10);
CompletableFuture.supplyAsync(() -> fetch(), executor);

```



### 7️⃣ `thenApply` vs `thenApplyAsync`

* **🧠 Theory:** A very common interview question.
* `thenApply`: Runs in the **same thread** that completed the previous stage (or the main thread if already complete).
* `thenApplyAsync`: Submits the task to the **thread pool** again, potentially running on a different thread.


* **Rule of Thumb:** Use `Async` versions for heavy computations to avoid blocking the thread that completed the previous task.

### 8️⃣ Handling Timeouts (Java 9+)

* **🧠 Theory:** Java 8 didn't have a built-in timeout for `CompletableFuture`. Java 9 added `orTimeout()` and `completeOnTimeout()`.
* **💻 Code:**
```java
future.orTimeout(2, TimeUnit.SECONDS) // Throws TimeoutException if slow
      .completeOnTimeout("Default", 2, TimeUnit.SECONDS); // Returns "Default" if slow

```



### 9️⃣ Async vs Parallel Streams

* **🧠 Theory:** When should you use which?
* **Parallel Streams:** For **CPU-intensive** tasks (heavy math, data processing) where the work is divided into equal chunks.
* **CompletableFuture:** For **I/O-intensive** tasks (microservice calls) where tasks have different latencies and need complex orchestration.



### 🔟 The Pipeline State Machine

* **🧠 Theory:** Internally, `CompletableFuture` acts as a state machine. It transitions from **Pending** to **Completed** or **Exceptionally**. Once completed, it triggers a "stack" of dependents. If you add a callback to an *already completed* future, it executes immediately in the caller's thread.

```

---

**This concludes the core "Modern Java" technical stack. Would you like to proceed to Design Patterns now (e.g., Factory, Observer, or Singleton), or should we look at Spring Boot internals?**

```