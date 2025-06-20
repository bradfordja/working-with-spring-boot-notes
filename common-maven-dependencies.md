## 🧱 dependencies used in most Spring Boot applications
Common Maven pom.xml dependencies
Certainly! Here’s a shortlist of common Maven pom.xml dependencies used in most Spring Boot applications, with a brief description and use case for each.

⸻

✅ Common Spring Boot Maven Dependencies

1. Spring Boot Starter Web

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

	•	Use Case: Build REST APIs or MVC web apps (includes Tomcat, Jackson, etc.)

⸻

2. Spring Boot Starter Data JPA

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
```

	•	Use Case: Integrate JPA (Hibernate) for database access and ORM

⸻

3. Spring Boot Starter Security

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

	•	Use Case: Add authentication and authorization (Basic Auth, JWT, OAuth2)

⸻

4. Spring Boot DevTools

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-devtools</artifactId>
  <scope>runtime</scope>
  <optional>true</optional>
</dependency>
```

	•	Use Case: Enable hot reload during development

⸻

5. Spring Boot Starter Test

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-test</artifactId>
  <scope>test</scope>
</dependency>
```

	•	Use Case: Unit and integration testing (JUnit, Mockito, MockMvc)

⸻

6. Spring Boot Starter Validation

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

	•	Use Case: Bean validation using @Valid, @NotNull, @Size, etc.

⸻

7. Spring Boot Starter Thymeleaf

```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

	•	Use Case: Render dynamic HTML pages (for MVC or admin UIs)

⸻

8. H2 Database (in-memory)

```xml
<dependency>
  <groupId>com.h2database</groupId>
  <artifactId>h2</artifactId>
  <scope>runtime</scope>
</dependency>
```

	•	Use Case: Lightweight in-memory database for development/testing

⸻

9. MySQL Driver

```xml
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <scope>runtime</scope>
</dependency>
```

	•	Use Case: Connect Spring Boot app to a MySQL database

⸻

10. MapStruct (DTO Mapping)

```xml
<dependency>
  <groupId>org.mapstruct</groupId>
  <artifactId>mapstruct</artifactId>
  <version>1.5.5.Final</version>
</dependency>
```

	•	Use Case: Generate mappers between entities and DTOs

⸻
