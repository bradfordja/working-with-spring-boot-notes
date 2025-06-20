## 🧱 Spring Boot MVC + JPA transactional
Here is a complete Spring Boot MVC + JPA transactional example that updates users’ manager ID with error handling and rollback. The components include:
	•	DTO: UserDTO
	•	Entity: User
	•	Repository: UserRepository
	•	Service: UserService with @Transactional for rollback
	•	Controller: UserController

This will update all users where active = true and managerId = 5675 to managerId = 4737. On error, changes are rolled back.

### UserDTO.java

```java
package com.example.demo.dto;

public class UserDTO {
    private Long id;
    private String username;
    private Long managerId;

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }

    public Long getManagerId() { return managerId; }
    public void setManagerId(Long managerId) { this.managerId = managerId; }
}
```

### User.java (Entity)

```java
package com.example.demo.model;

import jakarta.persistence.*;

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    @Column(name = "manager_id")
    private Long managerId;

    private boolean active;

    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }

    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }

    public Long getManagerId() { return managerId; }
    public void setManagerId(Long managerId) { this.managerId = managerId; }

    public boolean isActive() { return active; }
    public void setActive(boolean active) { this.active = active; }
}
```

### UserRepository.java

```java
package com.example.demo.repository;

import com.example.demo.model.User;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

import java.util.List;

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    List<User> findByActiveTrueAndManagerId(Long managerId);
}
```

### UserService.java

```java
package com.example.demo.service;

import com.example.demo.model.User;
import com.example.demo.repository.UserRepository;
import jakarta.transaction.Transactional;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;

@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

    @Transactional
    public int updateManagerForActiveUsers(Long oldManagerId, Long newManagerId) {
        try {
            List<User> users = userRepository.findByActiveTrueAndManagerId(oldManagerId);
            for (User user : users) {
                user.setManagerId(newManagerId);
            }
            userRepository.saveAll(users);
            return users.size();
        } catch (Exception e) {
            throw new RuntimeException("Transaction failed. Rolled back.", e);
        }
    }
}
```

### UserController.java

```java
package com.example.demo.controller;

import com.example.demo.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/users")
public class UserController {

    @Autowired
    private UserService userService;

    @PutMapping("/update-manager")
    public ResponseEntity<String> updateManager() {
        Long oldManagerId = 5675L;
        Long newManagerId = 4737L;
        int updatedCount = userService.updateManagerForActiveUsers(oldManagerId, newManagerId);
        return ResponseEntity.ok(updatedCount + " users updated.");
    }
}
```