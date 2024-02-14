# manages user login with JWT

Setting up a Spring Boot application with JWT authentication requires several steps. Here's a simplified breakdown with sample code for each step:

### 1. Setup a new Spring Boot Project:

You can initialize a Spring Boot project using Spring Initializr (https://start.spring.io/). Ensure you add the following dependencies:

- Spring Web
- Spring Security
- Spring Data JPA (assuming you want to store user details in a database)
- Your preferred database driver (like H2 for simplicity or MySQL for something more real-world)

### 2. Add Dependencies for JWT:

Include the Java JWT (jjwt) dependency in your `pom.xml`:

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

### 3. Define your User Entity and Repository:

User.java (simplified):
```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String password;

    // getters, setters, constructors...
}
```

UserRepository.java:
```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
}
```

### 4. Implement JWT Utility:

JWTUtil.java:
```java
@Component
public class JWTUtil {
    
    private String secret = "mySecretKey";  // In real apps, keep this secret and not hardcoded
    private int expiration = 3600000;      // 1 hour in milliseconds

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date(System.currentTimeMillis()))
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    }
    
    public String getUsernameFromToken(String token) {
        return getClaimsFromToken(token).getSubject();
    }

    private Claims getClaimsFromToken(String token) {
        return Jwts.parser().setSigningKey(secret).parseClaimsJws(token).getBody();
    }
    
    // Other utility methods like validation can also be added here.
}
```

### 5. Setup Spring Security Configuration:

SecurityConfig.java:
```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    private UserDetailsService userDetailsService;

    @Autowired
    private JWTUtil jwtUtil;

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
            .csrf().disable()
            .authorizeRequests()
            .antMatchers("/login").permitAll()
            .anyRequest().authenticated()
            .and()
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        
        // Add JWT filter
        http.addFilter(new JWTAuthenticationFilter(authenticationManager(), jwtUtil, userDetailsService));
    }

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(userDetailsService).passwordEncoder(passwordEncoder());
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
}
```

### 6. Implement JWT Authentication Filter:

JWTAuthenticationFilter.java:
```java
public class JWTAuthenticationFilter extends UsernamePasswordAuthenticationFilter {

    private JWTUtil jwtUtil;
    private UserDetailsService userDetailsService;

    public JWTAuthenticationFilter(AuthenticationManager authenticationManager, JWTUtil jwtUtil, UserDetailsService userDetailsService) {
        super(authenticationManager);
        this.jwtUtil = jwtUtil;
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain, Authentication authResult) throws IOException, ServletException {
        UserDetails userDetails = (UserDetails) authResult.getPrincipal();
        String token = jwtUtil.generateToken(userDetails);

        response.addHeader("Authorization", "Bearer " + token);
    }
}
```

### 7. Implement a Login endpoint:

AuthController.java:
```java
@RestController
public class AuthController {

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody AuthenticationRequest request) {
        try {
            authenticate(request.getUsername(), request.getPassword());
            final UserDetails userDetails = userDetailsService.loadUserByUsername(request.getUsername());
            final String jwt = jwtUtil.generateToken(userDetails);

            return ResponseEntity.ok(new AuthenticationResponse(jwt));
        } catch (BadCredentialsException e) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body("Incorrect username or password");
        }
    }
    
    private void authenticate(String username, String password) throws Exception {
        try {
            authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(username, password));
        } catch (DisabledException e) {
            throw new Exception("USER_DISABLED", e);
        } catch (BadCredentialsException e) {
            throw new Exception("INVALID_CREDENTIALS", e);
        }
    }
}
```

**Explanation**:
- Spring Security handles most of the security logic for us.
- We use JWT to generate a token for a successful login and send it back to the client.
- The client sends this token with each request, and Spring Security ensures it's valid.

**Use-case**:
A user sends their credentials (username & password) to the `/login` endpoint. If the credentials are valid, the server responds with a JWT token. The client stores this token and sends it in the `Authorization` header with each subsequent request to access protected resources.

This is a very basic example, and in a real-world scenario, you might want to expand on this by adding features such as token refresh mechanisms, more granular role-based authorizations, and so forth.