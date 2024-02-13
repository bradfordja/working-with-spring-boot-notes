Spring Security to Spring Boot project using JWT

As interviewing for a Senior Java developer role.  Given the set the of following questions please give simple answers that  senior Core Java, Java 8, Spring Boot, Microservices developer would give and if possible include sample code snipped with comments.

1. Give me the steps for adding Spring Security to Spring Boot project using JWT and using it to  create and validate/login users

Certainly! Integrating JWT with Spring Security in a Spring Boot application involves a series of steps. Here's a simplified guide:

**1. Add Dependencies to `pom.xml`**:

```xml
<!-- Spring Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- JWT Java library -->
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt</artifactId>
    <version>0.9.1</version>
</dependency>
```

**2. Define the JWT Util Class**:
This utility class provides methods to create and validate JWTs.

```java
@Component
public class JwtUtil {
    
    private String SECRET_KEY = "secret"; // This should be more complex and ideally outside the code

    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    private Claims extractAllClaims(String token) {
        return Jwts.parser().setSigningKey(SECRET_KEY).parseClaimsJws(token).getBody();
    }

    private Boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    public String generateToken(String username) {
        Map<String, Object> claims = new HashMap<>();
        return createToken(claims, username);
    }

    private String createToken(Map<String, Object> claims, String subject) {
        return Jwts.builder().setClaims(claims).setSubject(subject).setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60 * 10))  // 10 hours token validity
                .signWith(SignatureAlgorithm.HS256, SECRET_KEY).compact();
    }

    public Boolean validateToken(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername()) && !isTokenExpired(token));
    }
}
```

**3. Create a Custom UserDetailsService**:

```java
@Service
public class MyUserDetailsService implements UserDetailsService {

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // You should retrieve user details from your database. This is a hardcoded example.
        return new User("admin", new BCryptPasswordEncoder().encode("password"), new ArrayList<>());
    }
}
```

**4. Configure Spring Security**:

```java
@EnableWebSecurity
public class SecurityConfigurer extends WebSecurityConfigurerAdapter {

    @Autowired
    private MyUserDetailsService myUserDetailsService;

    @Autowired
    private JwtRequestFilter jwtRequestFilter;

    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        auth.userDetailsService(myUserDetailsService);
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeRequests().antMatchers("/authenticate").permitAll()
            .anyRequest().authenticated()
            .and().sessionManagement()
            .sessionCreationPolicy(SessionCreationPolicy.STATELESS);
        http.addFilterBefore(jwtRequestFilter, UsernamePasswordAuthenticationFilter.class);
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

    @Override
    @Bean
    public AuthenticationManager authenticationManagerBean() throws Exception {
        return super.authenticationManagerBean();
    }
}
```

**5. Add Authentication Endpoints and Filters**:

You would create an endpoint `/authenticate` where users would send their credentials. On successful authentication, they would receive a JWT which should be included in the headers of subsequent requests.

You would also create a `JwtRequestFilter` to intercept incoming requests, extract the JWT, validate it, and set the authentication in the Spring Security context.

**6. Login & Generate JWT**:

A login API endpoint (`/authenticate`) would be used where users send their username and password. Upon successful authentication, a JWT is generated using `JwtUtil` and returned to the user.

This is a basic setup to get JWT and Spring Security working. In a production environment, you'd also want to add exception handling, refresh tokens, and other important security measures.
