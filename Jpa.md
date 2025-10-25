Ah! I see your concern. This is a **common point of confusion** when moving from Java to Spring Data JPA. Let me clarify carefully.

---

### **1. Why we can use `findById()` and `save()` without explicitly defining them**

In Spring Data JPA:

* `JpaRepository<T, ID>` (and `CrudRepository<T, ID>`) **already provides standard methods** for your entity `T` with ID type `ID`.

Some of the important methods inherited from `JpaRepository`:

| Method                          | Description                                   |
| ------------------------------- | --------------------------------------------- |
| `save(entity: T): T`            | Saves a new entity or updates an existing one |
| `findById(id: ID): Optional<T>` | Finds an entity by its ID                     |
| `findAll(): List<T>`            | Returns all entities                          |
| `deleteById(id: ID)`            | Deletes entity by ID                          |
| `count(): Long`                 | Number of entities                            |

So, you **don’t need to define `findById()` or `save()`** in your repository—they’re automatically available.

---

### **2. Example**

```kotlin
interface ReportRepository : JpaRepository<Report, Long> {
    fun findByStatus(status: String): List<Report>  // custom query
}
```

Now you can do in service:

```kotlin
val report = reportRepo.findById(reportId)
    .orElseThrow { RuntimeException("Report not found") }

report.status = ReportStatus.Resolved
reportRepo.save(report)  // save updates
```

* `findById()` → fetches the entity
* `save()` → persists changes (new entity or updated entity)

---

### **3. Why this works in Kotlin too**

* Kotlin uses the **same Spring Data JPA interfaces** as Java.
* Even though the methods aren’t explicitly written, your repository inherits **all CRUD methods from `JpaRepository`**.

---

### ✅ **Takeaway**

* `save()`, `findById()`, `findAll()`, `deleteById()` → all built-in.
* You only need to **define custom queries** (like `findByStatus`) in the repository.
* That’s why in the service we could call `reportRepo.findById()` and `reportRepo.save()` directly without defining them.

---

If you want, I can **also explain how to make `findById` return `ReportStatus` properly from DB**, so it works seamlessly with the **sealed class**.

Do you want me to do that next?
