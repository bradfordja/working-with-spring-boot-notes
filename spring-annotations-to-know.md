## 🧱 Spring Annotations to know
```java
@Component
@Service, @Repository and @Controller
@Autowired
@Value
@Configuration
@Bean
@Transactional
@RequesMappings
@RestController
@SpringBootApplication
@EnableAutoConfiguration
@ComponentScan
```
⸻

### ✅ 1. @Component

🔹 Purpose:

Marks a class as a Spring-managed component (bean). Spring auto-detects it during component scanning.

🔹 Use Case:

Generic component—business logic, utility class, or configuration loader.

🔹 Example:
```java
@Component
public class EmailValidator {
    public boolean isValid(String email) {
        return email != null && email.contains("@");
    }
}
```

⸻

### ✅ 2. @Service, @Repository, and @Controller

🔹 All are specialized versions of @Component used for semantic clarity and tool support.

⸻

### 🟢 @Service

Used for business logic layer.
```java
@Service
public class UserService {
    public String getUser() {
        return "John";
    }
}
```

⸻

### 🟢 @Repository

Used for data access layer. Enables exception translation via PersistenceExceptionTranslationPostProcessor.
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {}
```

⸻

### 🟢 @Controller

Marks a class as a Spring MVC controller for returning views (e.g., JSP/Thymeleaf).
```java
@Controller
public class WebController {
    @GetMapping("/")
    public String home() {
        return "index"; // resolves to index.html or index.jsp
    }
}
```

⸻

### ✅ 3. @Autowired

🔹 Purpose:

Automatically injects Spring-managed beans via constructor, field, or setter injection.

🔹 Use Case:

Dependency injection for loosely-coupled components.
```java
@Service
public class NotificationService {

    private final EmailValidator validator;

    @Autowired
    public NotificationService(EmailValidator validator) {
        this.validator = validator;
    }
}
```

⸻

### ✅ 4. @Value

🔹 Purpose:

Injects primitive values or expressions from properties files.

🔹 Use Case:

Read config properties or SpEL (Spring Expression Language).
```java
@Value("${server.port}")
private int serverPort;

@Value("#{2 * 10}")
private int calculatedValue;
```

⸻

### ✅ 5. @Configuration

🔹 Purpose:

Marks a class as a source of bean definitions (replaces XML config).
```java
@Configuration
public class AppConfig {
    @Bean
    public EmailValidator emailValidator() {
        return new EmailValidator();
    }
}
```

⸻

### ✅ 6. @Bean

🔹 Purpose:

Declares a bean definition inside a @Configuration class.
```java
@Bean
public UserService userService() {
    return new UserService();
}
```

⸻

### ✅ 7. @Transactional

🔹 Purpose:

Manages transaction boundaries declaratively.

🔹 Use Case:

Wrap service methods that read/write DB within a transaction.
```java
@Service
public class OrderService {

    @Transactional
    public void placeOrder(Order order) {
        // Save to DB
        // Send message
    }
}
```
## ✅ Works with Spring Data and JPA/Hibernate seamlessly.

⸻

### ✅ 8. @RequestMapping

🔹 Purpose:

Maps HTTP requests to handler methods in a controller.
```java
@Controller
@RequestMapping("/users")
public class UserController {

    @RequestMapping(value = "/{id}", method = RequestMethod.GET)
    public ResponseEntity<User> getUser(@PathVariable Long id) {
        return ResponseEntity.ok(new User(id, "John"));
    }
}
```

⸻

### ✅ 9. @RestController

🔹 Purpose:

Combines @Controller and @ResponseBody for building RESTful web services.
```java
@RestController
@RequestMapping("/api")
public class ConfigController {

    @GetMapping("/config")
    public Map<String, String> getConfig() {
        return Map.of("mode", "dev");
    }
}
```

⸻

### ✅ 10. @SpringBootApplication

🔹 Purpose:

Meta-annotation that includes:
	•	@Configuration
	•	@EnableAutoConfiguration
	•	@ComponentScan
```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

⸻

### ✅ 11. @EnableAutoConfiguration

🔹 Purpose:

Tells Spring Boot to automatically configure beans based on classpath settings, properties, and beans.
```java
@EnableAutoConfiguration
public class LegacyApp {
    public static void main(String[] args) {
        SpringApplication.run(LegacyApp.class, args);
    }
}
```
## ✅ Typically not used directly—prefer @SpringBootApplication.

⸻

### ✅ 12. @ComponentScan

🔹 Purpose:

Tells Spring where to scan for @Component, @Service, @Repository, etc.
```java
@ComponentScan(basePackages = "com.example.service")
@Configuration
public class AppConfig {}
```
## ✅ Used to customize scanning in non-boot or modular applications.

⸻

🧠 Best Practices Summary for Interviews

￼

⸻

⸻

### ✅ Code:
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {}
```

⸻

### ✅ Explanation

🔹 1. @Repository
	•	This is a Spring stereotype annotation used to indicate that this interface is part of the data access layer (DAO).
	•	It enables:
	•	Component scanning (so Spring can detect and inject it)
	•	Exception translation: Spring converts JPA-specific exceptions into Spring’s DataAccessException hierarchy.

⸻

🔹 2. public interface UserRepository
	•	This declares an interface named UserRepository.
	•	In Spring Data JPA, repositories are usually interfaces, not classes.

⸻

🔹 3. extends JpaRepository<User, Long>

This is the core part that connects the interface to Spring Data JPA.

Part	Meaning
JpaRepository	A JPA-specific extension of PagingAndSortingRepository and CrudRepository. Provides rich CRUD + pagination + sorting methods.
<User, Long>	Tells Spring:

	•	User is the entity type (i.e., the class mapped to a table)
	•	Long is the type of the primary key (@Id field) |

### ✅ By extending JpaRepository, you automatically get dozens of ready-to-use methods, such as:
```java
findAll()
findById(Long id)
save(User user)
deleteById(Long id)
existsById(Long id)
```
You can also define custom query methods like:
```java
List<User> findByLastName(String lastName);
Optional<User> findByEmail(String email);
```

⸻

### ✅ Example Entity (User)
```java
@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;
    private String email;

    // Getters and setters
}
```

⸻

### ✅ Usage in a Service
```java
@Service
public class UserService {
    private final UserRepository userRepo;

    @Autowired
    public UserService(UserRepository userRepo) {
        this.userRepo = userRepo;
    }

    public User getUser(Long id) {
        return userRepo.findById(id)
                       .orElseThrow(() -> new RuntimeException("User not found"));
    }
}
```

⸻

### ✅ Summary

Component	Role
@Repository	Marks the interface as a DAO layer component
UserRepository	DAO interface for the User entity
extends JpaRepository<User, Long>	Inherits all CRUD, pagination, and sorting methods for the User entity with Long ID type


⸻