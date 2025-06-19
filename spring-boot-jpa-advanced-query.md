# 🎯 Advanced Query Customization in Spring Data JPA

Spring Data JPA allows you to define advanced custom queries using annotations, JPQL, native SQL, SpEL expressions, and projections. These are critical for handling complex data retrieval beyond basic CRUD operations.

---

## 🔧 Common Patterns for Advanced Queries

### 1. JPQL (Java Persistence Query Language)

```java
@Query("SELECT u FROM User u WHERE u.loginName = :login")
User findByLogin(@Param("login") String login);
```


	•	Uses entity and field names (not table or column names).
	•	Type-safe and portable.

⸻

2. Native SQL Queries

@Query(value = "SELECT * FROM users WHERE login_name = ?1", nativeQuery = true)
User findNativeByLogin(String login);

	•	Used when you need DB-specific SQL features.
	•	Must use column names instead of field names.

⸻

3. Update/Delete with @Modifying

```java
@Modifying
@Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
void updateStatus(@Param("id") Long id, @Param("status") String status);
```

	•	Required for queries that modify data (UPDATE/DELETE).
	•	Typically used with @Transactional.

⸻

4. SpEL (Spring Expression Language) in Queries

```java
@Query("SELECT u FROM User u WHERE u.createdBy = ?#{principal.username}")
List<User> findUsersCreatedByCurrentUser();
```


	•	Useful for multi-tenant or security-aware queries.
	•	Injects runtime variables (like logged-in user).

⸻

5. DTO Projections

```java
@Query("SELECT new com.example.dto.UserDTO(u.id, u.loginName) FROM User u")
List<UserDTO> fetchUserSummaries();
```


	•	Returns lightweight DTOs instead of full entities.
	•	Optimizes performance and avoids lazy-loading issues.

⸻

6. Derived Queries (Method Name Strategy)

```java
User findByFirstNameAndLastName(String firstName, String lastName);
List<User> findByStatusIn(List<String> statuses);
```


	•	Spring auto-generates queries based on method name.
	•	Great for simple filters without @Query.

⸻

✅ Best Practices
	•	Use JPQL for portability and safety.
	•	Use native queries only when absolutely necessary.
	•	Use DTO projections for large result sets.
	•	Always annotate update/delete queries with @Modifying.

⸻

❓ Interview Questions
	1.	What’s the difference between JPQL and native queries?
	•	JPQL is entity-based and portable; native SQL is DB-specific and column-based.
	2.	Can you return a DTO from a query?
	•	Yes, using JPQL constructor expressions or interface-based projections.
	3.	When should you use @Modifying?
	•	For UPDATE or DELETE queries — always paired with @Transactional.

⸻