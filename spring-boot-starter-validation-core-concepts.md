## 🧱 spring-boot-starter-validation - Core Concepts

### ✅ What Is spring-boot-starter-validation?

spring-boot-starter-validation is a Spring Boot starter that includes Hibernate Validator, the reference implementation of the Bean Validation 2.0 (JSR-380) API. It allows you to declaratively validate Java objects using annotations like @NotNull, @Size, @Email, etc.

⸻

🧱 Maven Dependency
```xml
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

### ✅ Spring Boot 2.3+ no longer includes validation by default—you must add this dependency explicitly.

⸻

🎯 Core Concepts

Concept	Description
Bean Validation	Use annotations to enforce constraints on fields or methods
JSR-380/JSR-303	Java validation standards used by Hibernate Validator
@Valid / @Validated	Used to trigger validation checks
Constraint Violations	Automatically handled and converted into HTTP 400 responses in Spring MVC


⸻

🧩 Common Validation Annotations

Annotation	Description
@NotNull	Must not be null
@NotBlank	Must not be null or empty string
@Size(min, max)	Length constraint
@Min / @Max	Numeric range validation
@Email	Must be a valid email
@Pattern	Regex pattern
@Past / @Future	Date constraints


⸻

### ✅ Sample Use Case: Validating a User Registration API

⸻

🔹 1. DTO Class with Validation Annotations
```java
public class UserRequest {

    @NotBlank(message = "Username is required")
    private String username;

    @Email(message = "Email must be valid")
    private String email;

    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;

    // Getters and setters
}
```

⸻

🔹 2. REST Controller with @Valid
```java
@RestController
@RequestMapping("/api/users")
public class UserController {

    @PostMapping
    public ResponseEntity<String> registerUser(@Valid @RequestBody UserRequest request) {
        // Process user
        return ResponseEntity.ok("User registered!");
    }
}
```

⸻

🔹 3. Automatic Error Handling

When a constraint is violated, Spring automatically returns a 400 Bad Request with a JSON body like:
```java
{
  "timestamp": "2025-05-22T18:00:00",
  "status": 400,
  "errors": [
    "Username is required",
    "Email must be valid"
  ]
}
```

⸻

🔹 4. Customize Error Response with @ControllerAdvice
```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<List<String>> handleValidationErrors(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult().getFieldErrors()
            .stream()
            .map(FieldError::getDefaultMessage)
            .collect(Collectors.toList());

        return new ResponseEntity<>(errors, HttpStatus.BAD_REQUEST);
    }
}
```

⸻

🧠 Best Practices

Practice	Why
✅ Use DTOs for validation	Keeps entities clean and separate
✅ Add messages to annotations	Improves client error feedback
✅ Use @Valid in controller params	Triggers validation automatically
✅ Centralize error handling	Avoid repetitive try-catch logic
❌ Don’t validate entities directly	Leads to tight coupling with DB schema


⸻

### ✅ Summary

Feature	Description
Purpose	Validates user input (API requests, form data, etc.)
Standard	Based on Bean Validation (JSR-380 / Hibernate Validator)
Trigger	Use @Valid or @Validated in controller or service
Errors	Automatically translated to HTTP 400 unless customized


⸻

Would you like a working Spring Boot project repo for validation, or examples of cross-field validation (@AssertTrue, @ScriptAssert)?