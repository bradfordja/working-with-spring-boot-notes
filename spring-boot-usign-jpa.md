### JPA to create CRUD for  employees table using Microsoft SQL Server

Below are the steps for adding JPA to a Spring Boot project to create CRUD operations for an `employees` table using Microsoft SQL Server:

1. **Add Dependencies in pom.xml**:

    Start by adding necessary dependencies to your `pom.xml` file.

    ```xml
    <!-- Spring Data JPA Dependency -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- SQL Server JDBC Driver -->
    <dependency>
        <groupId>com.microsoft.sqlserver</groupId>
        <artifactId>mssql-jdbc</artifactId>
        <version>8.4.1.jre8</version> <!-- Use the latest version or the version compatible with your SQL Server -->
    </dependency>
    ```

2. **Configure Database in `application.properties` or `application.yml`**:

    ```properties
    # For application.properties
    spring.datasource.url=jdbc:sqlserver://[server-address];databaseName=[database-name]
    spring.datasource.username=[username]
    spring.datasource.password=[password]
    spring.jpa.hibernate.ddl-auto=update
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.SQLServer2012Dialect
    ```

3. **Create the Entity Class**:

    You need to define the `Employee` entity, which will represent the `employees` table in your database.

    ```java
    package com.example.demo.entity;

    import javax.persistence.Entity;
    import javax.persistence.GeneratedValue;
    import javax.persistence.GenerationType;
    import javax.persistence.Id;

    @Entity
    public class Employee {

        @Id
        @GeneratedValue(strategy = GenerationType.IDENTITY)
        private Long id;
        private String name;
        private String position;

        // Getters, setters, constructors, etc.
    }
    ```

4. **Create the Repository**:

    You can create a repository interface to perform CRUD operations using JPA.

    ```java
    package com.example.demo.repository;

    import com.example.demo.entity.Employee;
    import org.springframework.data.jpa.repository.JpaRepository;

    public interface EmployeeRepository extends JpaRepository<Employee, Long> {
        // The basic CRUD operations are already provided by JpaRepository
    }
    ```

5. **Service and Controller**:

    Typically, for a clean architecture, you'd create a service layer between your controller and repository. This service would contain the business logic.

    - **Service**:

        ```java
        package com.example.demo.service;

        import com.example.demo.entity.Employee;
        import com.example.demo.repository.EmployeeRepository;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.stereotype.Service;

        @Service
        public class EmployeeService {
            
            @Autowired
            private EmployeeRepository employeeRepository;

            public Employee save(Employee employee) {
                return employeeRepository.save(employee);
            }

            // Other CRUD operations
        }
        ```

    - **Controller**:

        ```java
        package com.example.demo.controller;

        import com.example.demo.entity.Employee;
        import com.example.demo.service.EmployeeService;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.web.bind.annotation.*;

        @RestController
        @RequestMapping("/employees")
        public class EmployeeController {

            @Autowired
            private EmployeeService employeeService;

            @PostMapping
            public Employee save(@RequestBody Employee employee) {
                return employeeService.save(employee);
            }

            // Other CRUD endpoints
        }
        ```

6. **Run the Application**: 

    When you start your Spring Boot application, JPA, with the provided configurations, will connect to your Microsoft SQL Server database, and due to the `spring.jpa.hibernate.ddl-auto=update` configuration, if the `employees` table doesn't exist, it will be created.

That's a basic setup for adding JPA to a Spring Boot project targeting Microsoft SQL Server. However, in real-world scenarios, more configurations and considerations (like transactions, error handling, validation, etc.) would be needed.


Certainly! Let's continue from where we left off:

Create all CRUD service methods and controller methods please

### Service:

Extend the `EmployeeService` to cover all CRUD operations:

```java
package com.example.demo.service;

import com.example.demo.entity.Employee;
import com.example.demo.repository.EmployeeRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class EmployeeService {
    
    @Autowired
    private EmployeeRepository employeeRepository;

    // Create
    public Employee save(Employee employee) {
        return employeeRepository.save(employee);
    }

    // Read (get one by ID)
    public Optional<Employee> findById(Long id) {
        return employeeRepository.findById(id);
    }

    // Read (get all)
    public List<Employee> findAll() {
        return employeeRepository.findAll();
    }

    // Update
    public Employee update(Employee employee) {
        // Ideally, you'd include some logic here to check if the ID exists before saving
        return employeeRepository.save(employee);
    }

    // Delete
    public void deleteById(Long id) {
        employeeRepository.deleteById(id);
    }
}
```

### Controller:

Now, create corresponding endpoints in the `EmployeeController`:

```java
package com.example.demo.controller;

import com.example.demo.entity.Employee;
import com.example.demo.service.EmployeeService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/employees")
public class EmployeeController {

    @Autowired
    private EmployeeService employeeService;

    // Create
    @PostMapping
    public Employee save(@RequestBody Employee employee) {
        return employeeService.save(employee);
    }

    // Read (get one by ID)
    @GetMapping("/{id}")
    public Employee findById(@PathVariable Long id) {
        return employeeService.findById(id)
                .orElseThrow(() -> new RuntimeException("Employee not found with id: " + id));
    }

    // Read (get all)
    @GetMapping
    public List<Employee> findAll() {
        return employeeService.findAll();
    }

    // Update
    @PutMapping("/{id}")
    public Employee update(@PathVariable Long id, @RequestBody Employee employee) {
        // Ensure the ID from the path matches the ID of the employee
        if (!id.equals(employee.getId())) {
            throw new RuntimeException("ID in the path should match the ID in the employee object");
        }
        return employeeService.update(employee);
    }

    // Delete
    @DeleteMapping("/{id}")
    public void deleteById(@PathVariable Long id) {
        employeeService.deleteById(id);
    }
}
```

Now, you have a fully operational CRUD API for your `Employee` entity. Make sure to handle exceptions properly in a real-world application, possibly by using a global exception handler with `@ControllerAdvice`.