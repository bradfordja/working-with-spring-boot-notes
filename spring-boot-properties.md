# ⚙️ Spring Boot Properties — Interview Prep Guide

## 📌 What Are Spring Boot Properties?

Spring Boot uses the `application.properties` or `application.yml` file to externalize configuration. This allows managing environment-specific settings like server port, data source credentials, serialization behavior, and security controls from a central place.

---

## 🔧 Core Properties with Use Cases & Snippets

### 🔹 `server.port`

**Purpose:** Change the default HTTP port.

```properties
server.port=8081
```

**Use Case:** Run multiple apps on different ports locally.

---

### 🔹 `spring.application.name`

**Purpose:** Identifies your app in logs, monitoring tools, and service registries.

```properties
spring.application.name=inventory-service
```

---

### 🔹 `server.servlet.context-path`

**Purpose:** Sets the base URI for all routes in the app.

```properties
server.servlet.context-path=/api/v1
```

**Use Case:** Versioning your REST APIs.

---

### 🔹 `spring.datasource.*`

**Purpose:** Configure database connection details.

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=secret
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

---

### 🔹 `spring.jpa.*`

#### a) `spring.jpa.hibernate.ddl-auto`

**Purpose:** Auto create/update DB schema.

```properties
spring.jpa.hibernate.ddl-auto=update
```

**Options:** `none`, `validate`, `update`, `create`, `create-drop`

#### b) `spring.jpa.show-sql`

**Purpose:** Log generated SQL statements.

```properties
spring.jpa.show-sql=true
```

---

### 🔹 `spring.jackson.*` — JSON Serialization Settings

#### a) Pretty-printed JSON

```properties
spring.jackson.serialization.indent-output=true
```

#### b) ISO date formatting instead of timestamps

```properties
spring.jackson.serialization.write-dates-as-timestamps=false
```

#### c) Custom date format

```properties
spring.jackson.date-format=yyyy-MM-dd HH:mm:ss
```

#### d) Set default time zone

```properties
spring.jackson.time-zone=America/Denver
```

---

### 🔹 `spring.security.*`

#### a) Google OAuth2 Login

```properties
spring.security.oauth2.client.registration.google.client-id=your-client-id
spring.security.oauth2.client.registration.google.client-secret=your-client-secret
```

#### b) Stateless session policy (for REST APIs)

```properties
spring.security.session-management.session-creation-policy=stateless
```

---

### 🔹 `management.endpoints.web.exposure.include`

**Purpose:** Control which actuator endpoints are available over HTTP.

```properties
management.endpoints.web.exposure.include=health,info
```

---

### 🔹 `spring.threads.virtual.enabled`

**Purpose:** Enables Java virtual threads (from Project Loom).

```properties
spring.threads.virtual.enabled=true
```

**Use Case:** Improve concurrency with minimal thread overhead (Java 21+).

---

## 📘 Common Interview Questions

### 1. How do you change your Spring Boot application's default port?

```properties
server.port=9090
```

---

### 2. How do you enable logging of SQL statements?

```properties
spring.jpa.show-sql=true
```

---

### 3. How can you make your REST API stateless?

```properties
spring.security.session-management.session-creation-policy=stateless
```

---

### 4. How do you pretty-print JSON in REST responses?

```properties
spring.jackson.serialization.indent-output=true
```

---

### 5. How do you expose specific actuator endpoints?

```properties
management.endpoints.web.exposure.include=health,info
```

---

### 6. What does `spring.jpa.hibernate.ddl-auto=update` do?

It automatically updates the database schema to match entity definitions.

---

## ✅ Summary Table

| Property Key                                                   | Purpose                              |
|----------------------------------------------------------------|--------------------------------------|
| `server.port`                                                  | Set HTTP port                        |
| `spring.application.name`                                      | App identity                         |
| `spring.datasource.*`                                          | DB configuration                     |
| `spring.jpa.hibernate.ddl-auto`                                | Auto schema generation               |
| `spring.jpa.show-sql`                                          | Log SQL queries                      |
| `spring.jackson.*`                                             | Control JSON formatting              |
| `server.servlet.context-path`                                  | Base path for all endpoints          |
| `spring.security.oauth2.client.registration.google.*`          | Google OAuth login                   |
| `spring.security.session-management.session-creation-policy`   | REST stateless session               |
| `management.endpoints.web.exposure.include`                    | Expose specific actuator endpoints   |
| `spring.threads.virtual.enabled`                               | Enable virtual threads               |

---
