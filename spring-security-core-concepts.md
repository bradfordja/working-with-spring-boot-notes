## 🧱 Spring Security - Core Concepts

Below is an in-depth overview of the most important core concepts of Spring Security, including simple explanations, practical use-cases, and clear code snippets to guide you.

🚀 Spring Security - Core Concepts (with Examples)

### ✅ 1. Authentication

🚀 Explanation:
Authentication verifies who the user is. It confirms users’ identities typically via credentials (username/password, JWT tokens, OAuth tokens, etc.).

Use-case:
	•	Logging into web applications.

Example (Basic Authentication Config):

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
      http
        .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
        .httpBasic(Customizer.withDefaults());
      return http.build();
  }

  @Bean
  public UserDetailsService users() {
      UserDetails user = User.builder()
          .username("user")
          .password(passwordEncoder().encode("password"))
          .roles("USER")
          .build();
      return new InMemoryUserDetailsManager(user);
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
      return new BCryptPasswordEncoder();
  }
}
```

### ✅ 2. Authorization

🚀 Explanation:
Authorization verifies what actions a user can perform once authenticated, based on roles or permissions.

Use-case:
	•	Restricting access to admin endpoints.

Example (Role-Based Authorization):

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
      http
        .authorizeHttpRequests(auth -> auth
          .requestMatchers("/admin/**").hasRole("ADMIN")
          .requestMatchers("/user/**").hasAnyRole("USER", "ADMIN")
          .anyRequest().permitAll())
        .formLogin(Customizer.withDefaults());
      return http.build();
  }
}
```

### ✅ 3. UserDetails & UserDetailsService

🚀 Explanation:
	•	UserDetails: Defines user credentials and authorities.
	•	UserDetailsService: Loads user-specific data during authentication.

Use-case:
	•	Loading users from a database.

Example (Custom UserDetailsService):

```java
@Service
public class MyUserDetailsService implements UserDetailsService {

  @Autowired
  private UserRepository userRepository;

  @Override
  public UserDetails loadUserByUsername(String username) {
      User user = userRepository.findByUsername(username)
          .orElseThrow(() -> new UsernameNotFoundException("User not found"));

      return new org.springframework.security.core.userdetails.User(
          user.getUsername(), user.getPassword(), List.of(new SimpleGrantedAuthority("ROLE_USER")));
  }
}
```

### ✅ 4. Password Encoding

🚀 Explanation:
Secure storage of passwords by hashing (e.g., BCryptPasswordEncoder).

Use-case:
	•	Safely storing passwords in databases.

Example:

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

// Usage:
String hashedPassword = passwordEncoder.encode("password123");

### ✅ 5. SecurityContext & SecurityContextHolder

🚀 Explanation:
Stores information about the currently authenticated user.

Use-case:
	•	Fetching authenticated user details.

Example:

Authentication auth = SecurityContextHolder.getContext().getAuthentication();
String username = auth.getName();
Collection<? extends GrantedAuthority> roles = auth.getAuthorities();

### ✅ 6. JWT Authentication (Stateless Security)

🚀 Explanation:
JSON Web Token-based authentication provides stateless, scalable authentication ideal for REST APIs.

Use-case:
	•	REST APIs or microservices authentication.

Example (JWT token parsing in Spring Security Filter):

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    private JwtService jwtService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
        String authHeader = request.getHeader("Authorization");
        if(authHeader != null && authHeader.startsWith("Bearer ")) {
            String token = authHeader.substring(7);
            String username = jwtService.extractUsername(token);
            if(username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                UserDetails userDetails = jwtService.loadUserByUsername(username);
                if(jwtService.validateToken(token, userDetails)) {
                    UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(userDetails, null, userDetails.getAuthorities());
                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        }
        chain.doFilter(request, response);
    }
}
```

### ✅ 7. OAuth2 & OpenID Connect

🚀 Explanation:
Industry-standard protocols for delegated authentication/authorization (OAuth2) and authentication (OpenID Connect).

Use-case:
	•	Login via Google, Facebook, GitHub, etc.

Example (OAuth2 Login configuration):

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

  @Bean
  SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
      return http
        .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
        .oauth2Login(Customizer.withDefaults())
        .build();
  }
}
```

### ✅ 8. Method-level Security

🚀 Explanation:
Secures methods based on roles/permissions.

Use-case:
	•	Restricting method execution based on roles.

Example:

```java
@EnableMethodSecurity
@Configuration
public class SecurityConfig {
    //...
}

@Service
public class UserService {

    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long userId) {
        // delete logic
    }
}
```

### ✅ 9. Cross Site Request Forgery (CSRF) Protection

🚀 Explanation:
Protects against CSRF attacks by validating tokens.

Use-case:
	•	Web form submissions protection.

Example (CSRF configuration):

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
        .csrf(Customizer.withDefaults()) // enabled by default
        .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
        .build();
}
```

### ✅ 10. Security Filters & FilterChain

🚀 Explanation:
Spring Security uses a filter-chain pattern to apply security to HTTP requests sequentially.

Use-case:
	•	Adding custom security checks.

Example (custom filter):

```java
@Component
public class CustomLoggingFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws ServletException, IOException {
        logger.info("Request URI: " + request.getRequestURI());
        chain.doFilter(request, response);
    }
}
```

Configure in SecurityFilterChain:

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http, CustomLoggingFilter customLoggingFilter) throws Exception {
    return http.addFilterBefore(customLoggingFilter, UsernamePasswordAuthenticationFilter.class)
        .authorizeHttpRequests(auth -> auth.anyRequest().authenticated())
        .build();
}
```

### 🎯 Summary of Core Concepts

Concept	Role/Usage
Authentication	           Identify users
Authorization	           Control user permissions
UserDetailsService	      Load users from persistent store
Password Encoding	        Securely store passwords
SecurityContextHolder	   Access current authentication state
JWT	                     Stateless token-based authentication
OAuth2/OpenID Connect	   Third-party identity providers
Method-Level Security	   Restrict methods execution
CSRF Protection	Prevent   CSRF attacks
Security Filter Chain	   Extend security logic

Final Notes

Mastering these concepts demonstrates your expertise as a Java Architect. Understanding how to implement these in real-world scenarios ensures your applications are secure, scalable, and high-performance.
🚀 