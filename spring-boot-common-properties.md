# 📚 Most Common Spring Boot Properties — Interview Prep Guide

## 📌 Overview

Spring Boot offers a flexible and powerful configuration system through `application.properties` or `application.yml`. As a senior Spring Boot developer, understanding and using these properties effectively is crucial for building scalable, maintainable, and environment-specific applications.

---

## 🧩 Most Common Spring Boot Properties (with Use Cases)

### 🔹 `server.port`

**Purpose:** Sets the HTTP server port.

```properties
server.port=8081
```

**Use Case:** Useful when running multiple apps locally.

---

### 🔹 `spring.application.name`

**Purpose:** Defines the app name used in logs, monitoring, and service discovery.

```properties
spring.application.name=order-service
```

**Use Case:** Important in microservices or distributed tracing.

---

### 🔹 `spring.datasource.*`

**Purpose:** Configure your database connection.

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=postgres
spring.datasource.password=secret
```

**Use Case:** Set database credentials and URL.

---

### 🔹 `spring.jpa.hibernate.ddl-auto`

**Purpose:** Controls schema generation.

```properties
spring.jpa.hibernate.ddl-auto=update
```

**Options:** `create`, `update`, `validate`, `none`

**Use Case:** Auto-generate or validate DB schema in dev/test environments.

---

### 🔹 `spring.jpa.show-sql`

**Purpose:** Logs SQL statements to the console.

```properties
spring.jpa.show-sql=true
```

**Use Case:** Debug JPA queries during development.

---

### 🔹 `spring.jackson.serialization.*`

**Purpose:** Configures JSON serialization globally.

```properties
spring.jackson.serialization.indent-output=true
spring.jackson.serialization.write-dates-as-timestamps=false
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.time-zone=UTC
```

**Use Case:** Format REST API JSON responses and dates properly.

---

### 🔹 `server.servlet.context-path`

**Purpose:** Sets base path for all endpoints.

```properties
server.servlet.context-path=/api/v1
```

**Use Case:** Useful for API versioning or namespacing.

---

### 🔹 `spring.security.*`

**Purpose:** Configures Spring Security.

```properties
spring.security.oauth2.client.registration.google.client-id=your-client-id
spring.security.oauth2.client.registration.google.client-secret=your-client-secret
spring.security.session-management.session-creation-policy=stateless
```

**Use Case:** Secures your app with OAuth2 and stateless session for REST APIs.

---

### 🔹 `management.endpoints.web.exposure.include`

**Purpose:** Enables specific actuator endpoints.

```properties
management.endpoints.web.exposure.include=health,info
```

**Use Case:** Allow certain endpoints to be accessed for monitoring.

---

### 🔹 `spring.threads.virtual.enabled`

**Purpose:** Enables Java Virtual Threads (Project Loom support).

```properties
spring.threads.virtual.enabled=true
```

**Use Case:** Boosts concurrency in high-load applications (Java 21+).

---

## ❓ Common Interview Questions

### 1. How do you change the default port in Spring Boot?

```properties
server.port=9090
```

---

### 2. What does `spring.jpa.hibernate.ddl-auto=update` mean?

It updates the DB schema automatically to match the entity classes.

---

### 3. How do you print SQL to the console?

```properties
spring.jpa.show-sql=true
```

---

### 4. How do you format date/time in JSON responses?

```properties
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
spring.jackson.serialization.write-dates-as-timestamps=false
```

---

### 5. How do you enable only specific actuator endpoints?

```properties
management.endpoints.web.exposure.include=health,info
```

---

### 6. How do you secure a REST API to be stateless?

```properties
spring.security.session-management.session-creation-policy=stateless
```

---

## ✅ Summary Table

| Property                                                  | Purpose                              |
|-----------------------------------------------------------|--------------------------------------|
| `server.port`                                              | Change server port                   |
| `spring.application.name`                                 | Application name                     |
| `spring.datasource.*`                                     | DB connection settings               |
| `spring.jpa.hibernate.ddl-auto`                           | Schema generation                    |
| `spring.jpa.show-sql`                                     | Show SQL queries                     |
| `spring.jackson.*`                                        | Control JSON formatting              |
| `server.servlet.context-path`                             | API base path                        |
| `spring.security.oauth2.client.registration.google.*`     | Google login setup                   |
| `spring.security.session-management.*`                    | REST session handling                |
| `management.endpoints.web.exposure.include`               | Actuator endpoint control            |
| `spring.threads.virtual.enabled`                          | Enable virtual threads               |

---
