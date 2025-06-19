# 🚀 Advanced Spring Data JPA & Caching Techniques

---

## 📄 1. Pagination and Sorting

Spring Data provides built-in support for pagination and sorting using the `Pageable` and `Sort` interfaces.

### 🔹 Use Case
Retrieve large datasets in pages (e.g., display 10 users per page in UI).

### ✅ Code Example

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByStatus(String status, Pageable pageable);
}
```

🔸 Controller Example

```java
@GetMapping("/users")
public Page<User> getUsers(@RequestParam int page, @RequestParam int size) {
    Pageable pageable = PageRequest.of(page, size, Sort.by("firstName").ascending());
    return userRepository.findByStatus("ACTIVE", pageable);
}
```

⸻

📦 2. DTO Projections

DTO (Data Transfer Object) projection allows you to select only required fields rather than the entire entity.

🔹 Use Case

Return a lightweight user list (id, name) without loading unnecessary fields or relationships.

### ✅ DTO Class

```java
public class UserDTO {
    private Long id;
    private String loginName;

    public UserDTO(Long id, String loginName) {
        this.id = id;
        this.loginName = loginName;
    }
}
```

### ✅ Query in Repository

```java
@Query("SELECT new com.example.dto.UserDTO(u.id, u.loginName) FROM User u WHERE u.status = :status")
List<UserDTO> findUserSummaries(@Param("status") String status);
```



	•	Improves performance.
	•	Avoids lazy loading of unneeded relationships.

⸻

🔄 3. Spring Caching Strategies

Spring supports method-level caching using annotations like @Cacheable, @CachePut, and @CacheEvict.

🔹 Use Case

Cache frequently-accessed data like lookup tables, product catalogs, or user settings.

### ✅ Enable Caching

```java
@SpringBootApplication
@EnableCaching
public class MyApp {}
```

### ✅ @Cacheable Example

```java
@Cacheable("users")
public User getUserById(Long id) {
    return userRepository.findById(id).orElse(null);
}
```



	•	Caches the result of the method after the first call.
	•	Subsequent calls with the same id use the cached value.

### ✅ @CachePut Example

```java
@CachePut(value = "users", key = "#user.id")
public User updateUser(User user) {
    return userRepository.save(user);
}
```


	•	Always executes the method and updates the cache.

### ✅ @CacheEvict Example

```java
@CacheEvict(value = "users", key = "#id")
public void deleteUser(Long id) {
    userRepository.deleteById(id);
}
```


	•	Removes entry from the cache after deletion.

⸻

❓ Interview Questions

Q1: What’s the difference between Page, Slice, and List?
	•	Page: Includes metadata like total pages.
	•	Slice: Only knows if a next page exists.
	•	List: Returns all items.

Q2: How do DTO projections improve performance?
	•	They reduce data retrieval and memory footprint by selecting only needed fields.

Q3: When should you use caching in a Spring Boot app?
	•	When data is frequently read, rarely updated, and performance is critical (e.g., settings, categories).

⸻

### ✅ Summary Table

Feature	Benefit	Key API/Annotation
Pagination	Handles large data efficiently	Pageable, Page<T>
DTO Projections	Optimizes performance, avoids N+1 issues	@Query, Constructor DTO
Caching	Boosts performance for repeated queries	@Cacheable, @CacheEvict, @CachePut

⸻
