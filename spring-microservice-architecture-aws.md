## 🧱 Spring Boot microservices architecture with AWS and RDS integration

This API will be broken down into 3 Spring Boot microservices and use AWS services:

### 📦 Microservices
1. **UserService** — Handles user registration and login
2. **RoleService** — Handles CRUD operations for roles
3. **UserRoleService** — Handles user-role associations

Each service will:
- Expose REST endpoints via **AWS API Gateway**
- Persist to a **MySQL database** on **Amazon RDS**
- Be deployed to **AWS Lambda** using **Spring Cloud Function** or to **ECS/Fargate**

---

## 🛠️ Required Spring Boot Files per Microservice

### 1. **`pom.xml`**
**Purpose:** Maven dependencies
```xml
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  <dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
  </dependency>
</dependencies>
```

---

### 2. **`application.properties`**
**Purpose:** Connect to Amazon RDS MySQL
```properties
spring.datasource.url=jdbc:mysql://your-rds-endpoint:3306/userdb
spring.datasource.username=admin
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
```

---

### 3. **`User.java`**
```java
@Entity
public class User {
  @Id @GeneratedValue
  private Long id;
  @Column(unique = true, nullable = false)
  private String loginName;
  private String password;
  private String firstName;
  private String lastName;
  // getters and setters
}
```

### 4. **`Role.java`**
```java
@Entity
public class Role {
  @Id @GeneratedValue
  private Long id;
  @Column(unique = true)
  private String roleName;
}
```

### 5. **`UserRole.java`**
```java
@Entity
public class UserRole {
  @Id @GeneratedValue
  private Long id;
  private Long userId;
  private Long roleId;
}
```

---

### 6. **Repository Interfaces**
```java
public interface UserRepository extends JpaRepository<User, Long> {}
public interface RoleRepository extends JpaRepository<Role, Long> {}
public interface UserRoleRepository extends JpaRepository<UserRole, Long> {}
```

---

### 7. **Service Classes**
```java
@Service
public class UserService {
  @Autowired private UserRepository repo;
  public User saveUser(User user) { return repo.save(user); }
}
```

Repeat similarly for `RoleService`, `UserRoleService`

---

### 8. **Controller Classes**
```java
@RestController
@RequestMapping("/users")
public class UserController {
  @Autowired private UserService userService;

  @PostMapping
  public ResponseEntity<User> create(@RequestBody User user) {
    return new ResponseEntity<>(userService.saveUser(user), HttpStatus.CREATED);
  }
}
```

Repeat for `RoleController` and `UserRoleController`

---

### 9. **Deployment**
- Use **AWS Lambda adapter** (e.g., Spring Cloud Function) or **Dockerized Spring Boot** to ECS/Fargate
- Each service should be built separately and exposed through **API Gateway** paths:
  - `/users/**` → UserService
  - `/roles/**` → RoleService
  - `/user-roles/**` → UserRoleService
✅ The complete breakdown of your Spring Boot microservices architecture with AWS and RDS integration has been documented in markdown format. You can now export, edit, or continue building from this starting point.

### TODO:
	•	.zip folder structure for each service
	•	AWS Lambda handler integration
	•	Terraform or CDK deployment config
	•	Swagger/OpenAPI support
	•	Secrets Manager/IAM integration