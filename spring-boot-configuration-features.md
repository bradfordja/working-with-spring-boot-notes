## 🧱 Spring Boot - Configuration features 

Understanding Spring Boot's configuration mechanisms is crucial for any Spring developer. Here's a breakdown of common features, their use cases, and examples:

### 1. @Value Annotation
The @Value annotation is a simple way to inject a single property value from various sources (like application.properties or application.yml files, environment variables, or system properties) into a bean's field or method parameter.

Use Cases:
* Injecting simple configuration values like a database URL, a port number, or a feature flag.
* Providing default values if a property is not found.
* Using Spring Expression Language (SpEL) for more complex value injections.

Code Snippet:
Let's say you have a property in your application.properties:

```properties
app.name=My Awesome Application
app.version=1.0.0
default.greeting=Hello
```

You can inject these values into a Spring component like this:
```java
import org.springframework.beans.factory.annotation.Value;
import org.springframework.stereotype.Component;

@Component
public class AppConfig {

    @Value("${app.name}")
    private String appName;

    @Value("${app.version}")
    private String appVersion;

    // Providing a default value
    @Value("${app.greeting:${default.greeting}}") // Injects 'app.greeting' if present, otherwise 'default.greeting'
    private String greeting;

    // SpEL for default value
    @Value("${some.port:8080}") // Injects 'some.port' if present, otherwise 8080
    private int serverPort;

    public String getAppName() {
        return appName;
    }

    public String getAppVersion() {
        return appVersion;
    }

    public String getGreeting() {
        return greeting;
    }

    public int getServerPort() {
        return serverPort;
    }

    public void displayConfig() {
        System.out.println("App Name: " + appName);
        System.out.println("App Version: " + appVersion);
        System.out.println("Greeting: " + greeting);
        System.out.println("Server Port: " + serverPort);
    }
}
```

🔹 Key Interview Points:
* Simplicity: Good for injecting individual, simple values.
* Type Conversion: Spring automatically converts the property string to the target type (e.g., String, int, boolean).
* Default Values: The :${defaultValue} syntax is handy.
* SpEL: Can be used for more dynamic value injection (#{systemProperties['user.home']}).
* Limitations: Can become cumbersome for managing many related properties or complex hierarchical configurations. This is where @ConfigurationProperties shines.

### 2. @ConfigurationProperties Annotation
@ConfigurationProperties provides a powerful and type-safe way to bind external configuration properties (typically hierarchical) to a Java bean. It's often preferred over @Value for managing groups of related properties.

🔹 Use Cases:
* Binding a group of related properties to a POJO (Plain Old Java Object).
* Handling complex, nested configuration structures.
* Validating configuration properties using JSR 303 bean validation annotations (e.g., @NotNull, @Min, @Max).
* Improved type safety and IDE support (autocompletion for properties).

🔹 Code Snippet:
Suppose you have the following properties in application.yml:
```YAML
app:
  name: My Configured App
  host: localhost
  port: 8080
  security:
    username: admin
    roles:
      - USER
      - ADMIN
```

You can create a configuration properties class:

```java
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component; // Or @Configuration
import org.springframework.validation.annotation.Validated; // For validation

import jakarta.validation.constraints.NotEmpty; // JSR 303/380
import jakarta.validation.constraints.Positive;
import java.util.List;
import java.util.Map;

@Component // Makes it a Spring bean and eligible for property binding
@ConfigurationProperties(prefix = "app") // Binds properties starting with 'app'
@Validated // Enables validation
public class AppProperties {

    @NotEmpty // Example validation
    private String name;
    private String host;
    @Positive // Example validation
    private int port;
    private Security security;

    // Standard getters and setters are required for binding

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getHost() { return host; }
    public void setHost(String host) { this.host = host; }
    public int getPort() { return port; }
    public void setPort(int port) { this.port = port; }
    public Security getSecurity() { return security; }
    public void setSecurity(Security security) { this.security = security; }

    public static class Security { // Nested class for nested properties
        private String username;
        private List<String> roles;
        // Standard getters and setters

        public String getUsername() { return username; }
        public void setUsername(String username) { this.username = username; }
        public List<String> getRoles() { return roles; }
        public void setRoles(List<String> roles) { this.roles = roles; }
    }

    public void display() {
        System.out.println("App Name: " + name);
        System.out.println("Host: " + host);
        System.out.println("Port: " + port);
        System.out.println("Security Username: " + (security != null ? security.getUsername() : "N/A"));
        System.out.println("Security Roles: " + (security != null ? security.getRoles() : "N/A"));
    }
}
```

To enable @ConfigurationProperties, you typically need to either:
* Annotate the class with @Component (or @Service, @Configuration, etc.).
* Or, use @EnableConfigurationProperties(AppProperties.class) on a @Configuration class.

🔹 Key Interview Points:
* Type Safety: Converts properties to the correct types in your POJO.
* Hierarchical Binding: Easily maps nested YAML/properties structures.
* Validation: Supports JSR 303/380 validation annotations.
* Relaxed Binding: Spring Boot uses relaxed rules for matching property names to field names (e.g., app.my-property maps to myProperty).
* IDE Support: Often provides better autocompletion and metadata if spring-boot-configuration-processor is on the classpath.
* Constructor Binding: Immutable configuration properties can be achieved by using constructor binding (available since Spring Boot 2.2).

### 3. Spring Boot Actuator (Configuration Aspects)
Spring Boot Actuator exposes operational information about your running application via HTTP endpoints or JMX. Several Actuator endpoints are highly relevant for understanding and debugging configuration.

🔹 Use Cases:
* Viewing effective configuration: Seeing what properties are loaded and their values in a running application.
* Debugging configuration issues: Identifying why a specific configuration isn't being applied as expected.
* Understanding property sources: Seeing the order and precedence of loaded property sources.

🔹 Key Actuator Endpoints for Configuration:
* /actuator/configprops: Displays a collated list of all @ConfigurationProperties beans and their current property values.
* /actuator/env: Exposes properties from Spring's ConfigurableEnvironment. It shows all active profiles, system properties, environment variables, and properties from application.properties/application.yml files, along with their precedence.
* /actuator/beans: Displays the complete list of all Spring beans in your application, which can help understand how configuration classes are being instantiated.

🔹 Configuration (in application.properties):
By default, not all Actuator endpoints are exposed over HTTP for security reasons.
Properties

# Expose specific Actuator endpoints over HTTP (use '*' for all, but be cautious in production)
management.endpoints.web.exposure.include=health,info,env,configprops,beans
# To enable all endpoints (not recommended for production without security)
# management.endpoints.web.exposure.include=*

# You can also change the base path for actuator endpoints
# management.endpoints.web.base-path=/manage
Example Usage (Accessing via HTTP):
* GET http://localhost:8080/actuator/configprops
* GET http://localhost:8080/actuator/env
* GET http://localhost:8080/actuator/env/app.name (to get a specific property)
Key Interview Points:
* Visibility: Actuator provides runtime insight into your application's configuration.

* Debugging: Invaluable for troubleshooting why a certain property isn't taking effect or has an unexpected value.
* Property Sources: /env endpoint clearly shows the hierarchy and overriding of properties from different sources.
* Security: Be mindful of which endpoints are exposed, especially in production, as they can reveal sensitive configuration data. Secure them appropriately using Spring Security.

### 4. Spring Profiles
Spring Profiles allow you to define different bean configurations or property values for different environments (e.g., development, testing, staging, production). This enables you to tailor your application's behavior without changing your core codebase.

🔹 Use Cases:
* Environment-Specific Database Configuration: Using an H2 in-memory database for development/testing and a PostgreSQL database for production.
* Feature Toggling: Enabling or disabling certain features based on the active profile.
* Mocking Services: Using mock implementations of external services in a testing profile.
* Logging Levels: Setting different logging verbosity for different environments.

🔹 Defining Profiles:
### 1. Property Files: Create profile-specific property files:
    * application.properties (default)
    * application-dev.properties (for 'dev' profile)
    * application-prod.properties (for 'prod' profile)
    * Similarly for YAML: application.yml, application-dev.yml, application-prod.yml

### 2. Properties in profile-specific files override those in the default application.properties when that profile is active.

### 3. @Profile Annotation: Conditionally register beans or configuration classes based on the active profile.

🔹 Code Snippet (@Profile):

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Profile;

interface DataSourceConfig {
    void setup();
}

@Configuration
public class AppDataSourceConfig {

    @Bean
    @Profile("dev") // This bean will only be created if 'dev' profile is active
    public DataSourceConfig devDataSourceConfig() {
        return new DevDataSourceConfig();
    }

    @Bean
    @Profile("prod") // This bean will only be created if 'prod' profile is active
    public DataSourceConfig prodDataSourceConfig() {
        return new ProdDataSourceConfig();
    }

    @Bean
    @Profile("!prod") // Active if 'prod' profile is NOT active
    public DataSourceConfig nonProdDataSourceConfig() {
        return new NonProdDataSourceConfig();
    }
}

class DevDataSourceConfig implements DataSourceConfig {
    @Override
    public void setup() { System.out.println("Setting up DEV H2 Database"); }
}

class ProdDataSourceConfig implements DataSourceConfig {
    @Override
    public void setup() { System.out.println("Setting up PRODUCTION PostgreSQL Database"); }
}

class NonProdDataSourceConfig implements DataSourceConfig {
    @Override
    public void setup() { System.out.println("Setting up NON-PROD specific configuration (e.g., for dev or test)"); }
}
```

🔹 Activating Profiles:
application.properties: 
# Propertiesspring.profiles.active=dev

🔹 Command Line Argument: 
```Bashjava -jar myapp.jar --spring.profiles.active=prod
```
🔹 Environment Variable: 
```java
SPRING_PROFILES_ACTIVE=staging
```
🔹 Programmatically: 
```java
SpringApplication.setAdditionalProfiles("dev");
```
🔹 Testing (@ActiveProfiles): 
```java
@SpringBootTest
@ActiveProfiles("test")
class MyServiceTests { /* ... */ }```
```

🔹 Key Interview Points:
* Environment Segregation: Key for managing different configurations across deployment stages.
* Property Overriding: Profile-specific properties take precedence.
* Conditional Bean Registration: @Profile annotation is powerful for defining environment-specific beans.
* Multiple Profiles: You can activate multiple profiles (e.g., spring.profiles.active=dev,local-db). The last one often takes precedence in case of conflicting properties defined directly in profile-specific files, but for @Profile annotations, a bean is created if any of its specified profiles are active (unless using complex profile expressions).
* Default Profile: If no profiles are active, beans not marked with @Profile or marked with @Profile("default") are registered.

5. Spring Environment Abstraction
The Environment abstraction in Spring is a central interface representing the environment in which the current application is running. It provides access to profiles and properties from various sources.

🔹 Use Cases:
* Programmatically accessing property values.
* Checking which profiles are currently active.
* Resolving placeholders in strings.

🔹 Property Sources Hierarchy (Simplified):
Spring loads properties from various sources with a defined precedence (higher on the list overrides lower):

1. Devtools global settings (~/.spring-boot-devtools.properties)
2. @TestPropertySource annotations in tests.
3. properties attribute on @SpringBootTest.
4. Command-line arguments (--my.prop=value).
5. SPRING_APPLICATION_JSON (inline JSON embedded in an environment variable).
6. ServletConfig init parameters.
7. ServletContext init parameters.
8. JNDI attributes (java:comp/env/...).
9. Java System properties (System.getProperties()).
10. OS environment variables.
11. Profile-specific application-{profile}.properties (or .yml) files (outside packaged jar).
12. Profile-specific application-{profile}.properties (or .yml) files (inside packaged jar).
13. Application application.properties (or .yml) files (outside packaged jar).
14. Application application.properties (or .yml) files (inside packaged jar).
15. @PropertySource annotations on @Configuration classes.
16. Default properties (specified using SpringApplication.setDefaultProperties).

🔹 Code Snippet (Accessing Environment):
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.env.Environment;
import org.springframework.stereotype.Service;

@Service
public class MyService {

    private final Environment environment;

    @Autowired
    public MyService(Environment environment) {
        this.environment = environment;
    }

    public void displayAppDetails() {
        String appName = environment.getProperty("app.name");
        String defaultPort = environment.getProperty("server.port", "8080"); // Get property with a default value

        System.out.println("Application Name from Environment: " + appName);
        System.out.println("Server Port from Environment (default 8080): " + defaultPort);

        if (environment.acceptsProfiles(org.springframework.core.env.Profiles.of("dev"))) {
            System.out.println("Running with 'dev' profile.");
        }

        if (environment.acceptsProfiles(org.springframework.core.env.Profiles.of("prod"))) {
            System.out.println("Running with 'prod' profile.");
        }

        // You can also check for multiple active profiles
        String[] activeProfiles = environment.getActiveProfiles();
        if (activeProfiles.length > 0) {
            System.out.println("Active profiles: " + String.join(", ", activeProfiles));
        } else {
            System.out.println("No active profiles. Running with default profile(s).");
        }
    }
}
```

🔹 Key Interview Points:
* Centralized Access: Provides a unified way to access properties and profile information regardless of where they are defined.
* Property Resolution: Use getProperty() to fetch values. It can also resolve placeholders like ${some.other.property}.
* Profile Checking: getActiveProfiles() and acceptsProfiles() are useful for conditional logic based on active profiles.
* Understanding Precedence: Knowing the property source order is vital for debugging overridden properties.
* ConfigurableEnvironment: This sub-interface (which ApplicationContext usually provides) allows for modification of property sources, though it's less common in typical application code.
