
# Date and Time API (java.time)

## Notes

# 📘 Complete Java 8 Date and Time API Reference

### 1️⃣ Overview
* **🧠 Theory:** Before Java 8, we relied on `java.util.Date` and `java.util.Calendar`. These were poorly designed, confusing to use, and—most importantly—**not thread-safe**.
* **The Solution:** Java 8 introduced the `java.time` package (based on Joda-Time).
* **Key Traits:**
    * **Immutable:** All classes are immutable and thread-safe.
    * **Clear Separation:** Distinguishes between machine-readable time and human-readable time.
    * **Fluent API:** Uses a pattern of `of`, `now`, `plus`, `minus`, and `with` methods.

### 2️⃣ The Core Classes (Human Time)
* **🧠 Theory:** These classes represent time from a human perspective (Year, Month, Day).
* **💻 Code:**
  ```java
  // 1. Date only (2026-02-26)
  LocalDate date = LocalDate.now();
  LocalDate birthday = LocalDate.of(1995, Month.JANUARY, 15);

  // 2. Time only (14:30:05)
  LocalTime time = LocalTime.now();

  // 3. Both (2026-02-26T14:30:05)
  LocalDateTime dateTime = LocalDateTime.now();

```

### 3️⃣ Instant (Machine Time)

* **🧠 Theory:** Represents a single point in time on the timeline (UTC). Internally, it is the number of nanoseconds since the "Unix Epoch" (Jan 1, 1970).
* **Usage:** Used for logging, timestamps, and measuring application performance.
* **💻 Code:**
```java
Instant start = Instant.now();
// ... perform operation ...
Instant end = Instant.now();

```



### 4️⃣ ZonedDateTime (Global Time)

* **🧠 Theory:** `LocalDateTime` + `ZoneId`. Essential for applications running across multiple regions.
* **💻 Code:**
```java
ZonedDateTime set = ZonedDateTime.now(ZoneId.of("America/New_York"));

```



### 5️⃣ Duration vs. Period

* **🧠 Theory:**
* **Duration:** Measures time in **seconds and nanoseconds** (Machine time). Best for `Instant` or `LocalTime`.
* **Period:** Measures time in **years, months, and days** (Human time). Best for `LocalDate`.


* **💻 Code:**
```java
// Duration example
Duration duration = Duration.between(startTime, endTime);

// Period example
Period age = Period.between(birthDate, today);
System.out.println("Age is: " + age.getYears() + " years.");

```



### 6️⃣ DateTimeFormatter

* **🧠 Theory:** Replaces the thread-unsafe `SimpleDateFormat`. It is **immutable and thread-safe**.
* **💻 Code:**
```java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm");
String formatted = dateTime.format(formatter);
LocalDateTime parsed = LocalDateTime.parse("26-02-2026 10:30", formatter);

```



---

## 🌟 Senior & Lead Developer Addendum (Architecture & Pitfalls)

### 7️⃣ Why `java.util.Date` was a Disaster

* **🧠 Theory:** 1. **Mutability:** Anyone could call `date.setTime()`, making it dangerous to share across threads.
2. **Zero-based Months:** In the old API, January was `0`. In `java.time`, January is `1`.
3. **Confusing Names:** `java.util.Date` actually represents both date and time, while `java.sql.Date` (its subclass) represents only the date.

### 8️⃣ Handling Daylight Saving Time (DST)

* **🧠 Theory:** Senior developers must know that `LocalDateTime` is "dangerously" unaware of DST. If you add 1 hour to a `LocalDateTime` during a DST transition, the clock might be physically wrong.
* **The Fix:** Always use `ZonedDateTime` or `OffsetDateTime` for calculations involving actual physical time passages in specific regions.

### 9️⃣ TemporalAdjusters (Complex Logic)

* **🧠 Theory:** How do you find "The next working day" or "The last Sunday of the month"?
* **💻 Code:**
```java
LocalDate today = LocalDate.now();
LocalDate nextSunday = today.with(TemporalAdjusters.next(DayOfWeek.SUNDAY));
LocalDate lastDayOfMonth = today.with(TemporalAdjusters.lastDayOfMonth());

```



### 🔟 Precision and Storage (Database Best Practices)

* **🧠 Theory:** When storing dates in a database (like PostgreSQL or MySQL):
* Use **UTC** for storage (`Instant` or `OffsetDateTime`).
* Convert to the user's local timezone **only** at the UI/Presentation layer.
* Avoid storing `LocalDateTime` in the DB if the application serves global users, as you lose the context of *when* that event actually happened globally.



### 1️⃣1️⃣ ChronoUnit for Math

* **🧠 Theory:** A clean way to perform date math without manually calculating milliseconds.
* **💻 Code:**
```java
long daysBetween = ChronoUnit.DAYS.between(startDate, endDate);
LocalDate nextYear = today.plus(1, ChronoUnit.YEARS);

```



```

---

**This completes the essential Java 8 deep-dives! Since you're targeting senior roles, would you like to pivot to Design Patterns (Factory, Strategy, Observer) or perhaps a look at Java 9-17 features (Records, Sealed Classes, Var)?**

```