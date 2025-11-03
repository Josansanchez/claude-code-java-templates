# Advanced Security

## Overview

This guide covers advanced security topics including OWASP Top 10 vulnerabilities, rate limiting, input sanitization, and security best practices for Spring Boot applications.

## OWASP Top 10 (2021) Protection

### 1. Broken Access Control

**Problem**: Users accessing unauthorized resources

**Solution**: Use @PreAuthorize and proper authorization

```java
@RestController
@RequestMapping("/api/v1/users")
@RequiredArgsConstructor
public class UserController {

    // ✅ CORRECT: Check ownership
    @GetMapping("/{username}")
    @PreAuthorize("#username == authentication.name or hasRole('ADMIN')")
    public ResponseEntity<UserDTO> getUser(@PathVariable String username) {
        return ResponseEntity.ok(userService.getUser(username));
    }

    // ✅ CORRECT: Admin only
    @DeleteMapping("/{username}")
    @PreAuthorize("hasRole('ADMIN')")
    public ResponseEntity<Void> deleteUser(@PathVariable String username) {
        userService.deleteUser(username);
        return ResponseEntity.noContent().build();
    }
}
```

**Service Layer Protection**:
```java
@Service
public class OrderService {

    public OrderDTO getOrder(Long orderId, String username) {
        Order order = orderRepository.findById(orderId)
            .orElseThrow(() -> new ResourceNotFoundException("Order not found"));

        // ✅ Verify ownership
        if (!order.getUser().getUsername().equals(username)) {
            throw new AccessDeniedException("Not authorized to access this order");
        }

        return orderMapper.toDTO(order);
    }
}
```

### 2. Cryptographic Failures

**Problem**: Weak encryption, exposed secrets

**Solution**: Strong encryption and secret management

```java
@Configuration
public class SecurityConfig {

    @Bean
    public PasswordEncoder passwordEncoder() {
        // ✅ Use BCrypt with strength 12
        return new BCryptPasswordEncoder(12);
    }
}

// Encrypt sensitive data
@Service
public class EncryptionService {

    @Value("${encryption.secret}")
    private String secret;

    public String encrypt(String data) {
        try {
            SecretKeySpec key = new SecretKeySpec(
                secret.getBytes(), "AES");
            Cipher cipher = Cipher.getInstance("AES/GCB/PKCS5Padding");
            cipher.init(Cipher.ENCRYPT_MODE, key);
            byte[] encrypted = cipher.doFinal(data.getBytes());
            return Base64.getEncoder().encodeToString(encrypted);
        } catch (Exception e) {
            throw new RuntimeException("Encryption failed", e);
        }
    }
}
```

**Environment Variables**:
```yaml
# application.yml
jwt:
  secret: ${JWT_SECRET}  # From environment, not hardcoded

encryption:
  secret: ${ENCRYPTION_SECRET}
```

### 3. Injection (SQL, JPQL, Command)

**Problem**: Malicious input executed as code

**Solution**: Use JPA Specifications (type-safe), parameterized queries

```java
// ✅ CORRECT: JPA Specifications (type-safe)
public static Specification<Product> hasName(String name) {
    return (root, query, cb) ->
        name == null ? cb.conjunction() :
        cb.equal(root.get("name"), name);
}

// ✅ CORRECT: Parameterized query
@Query("SELECT u FROM User u WHERE u.username = :username")
Optional<User> findByUsername(@Param("username") String username);

// ❌ WRONG: String concatenation
@Query("SELECT u FROM User u WHERE u.username = '" + username + "'")  // VULNERABLE!
Optional<User> findByUsernameBad(String username);

// ❌ WRONG: Native query with concatenation
@Query(value = "SELECT * FROM users WHERE username = '" + username + "'",
       nativeQuery = true)  // SQL INJECTION!
Optional<User> findByUsernameNative(String username);
```

**Command Injection Prevention**:
```java
// ❌ DANGEROUS: Executing user input
Runtime.getRuntime().exec("ls " + userInput);  // VULNERABLE!

// ✅ SAFE: Validate and whitelist
public void executeAllowedCommand(String command) {
    Set<String> allowedCommands = Set.of("status", "info", "health");

    if (!allowedCommands.contains(command)) {
        throw new IllegalArgumentException("Command not allowed");
    }

    // Execute safely
}
```

### 4. Insecure Design

**Solution**: Implement security by design

```java
// ✅ Rate limiting
// ✅ Input validation
// ✅ Fail securely
// ✅ Principle of least privilege

@Service
public class AuthService {

    private static final int MAX_LOGIN_ATTEMPTS = 5;
    private final Map<String, Integer> loginAttempts = new ConcurrentHashMap<>();

    public void login(LoginRequest request) {
        String username = request.getUsername();

        // Check if account is locked
        if (isAccountLocked(username)) {
            throw new AccountLockedException("Account locked due to too many failed attempts");
        }

        try {
            // Attempt authentication
            authenticate(request);
            // Reset attempts on success
            loginAttempts.remove(username);
        } catch (BadCredentialsException e) {
            // Increment failed attempts
            int attempts = loginAttempts.getOrDefault(username, 0) + 1;
            loginAttempts.put(username, attempts);

            if (attempts >= MAX_LOGIN_ATTEMPTS) {
                lockAccount(username);
                throw new AccountLockedException("Account locked");
            }

            throw e;
        }
    }
}
```

### 5. Security Misconfiguration

**Solution**: Secure defaults, disable unnecessary features

```yaml
# application-prod.yml
server:
  error:
    include-stacktrace: never  # ✅ Don't expose stack traces
    include-message: never
    include-binding-errors: never

spring:
  jpa:
    show-sql: false  # ✅ Don't log SQL in production
    properties:
      hibernate:
        format_sql: false

management:
  endpoints:
    web:
      exposure:
        include: health,info  # ✅ Only expose necessary endpoints
```

### 6. Vulnerable and Outdated Components

**Solution**: Keep dependencies updated

```xml
<!-- Use Dependency Check plugin -->
<plugin>
    <groupId>org.owasp</groupId>
    <artifactId>dependency-check-maven</artifactId>
    <version>8.4.0</version>
    <executions>
        <execution>
            <goals>
                <goal>check</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

```bash
# Check for vulnerabilities
./mvnw dependency-check:check

# Update dependencies
./mvnw versions:display-dependency-updates
```

### 7. Identification and Authentication Failures

**Solution**: Strong authentication, JWT, secure sessions

```java
@Configuration
public class JwtConfig {

    @Value("${jwt.secret}")
    private String secret;

    @Value("${jwt.expiration:3600000}")  // 1 hour
    private Long expiration;

    public String generateToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        claims.put("roles", userDetails.getAuthorities());

        return Jwts.builder()
            .setClaims(claims)
            .setSubject(userDetails.getUsername())
            .setIssuedAt(new Date())
            .setExpiration(new Date(System.currentTimeMillis() + expiration))
            .signWith(SignatureAlgorithm.HS512, secret)
            .compact();
    }
}
```

### 8. Software and Data Integrity Failures

**Solution**: Verify integrity, use signed JWTs

```java
// ✅ Verify JWT signature
public boolean validateToken(String token) {
    try {
        Jwts.parser()
            .setSigningKey(secret)
            .parseClaimsJws(token);  // Throws exception if invalid
        return true;
    } catch (JwtException | IllegalArgumentException e) {
        return false;
    }
}
```

### 9. Security Logging and Monitoring Failures

**Solution**: Log security events, monitor anomalies

```java
@Slf4j
@Component
public class SecurityEventLogger {

    public void logLoginSuccess(String username, String ip) {
        log.info("LOGIN_SUCCESS: user={}, ip={}", username, ip);
    }

    public void logLoginFailure(String username, String ip, String reason) {
        log.warn("LOGIN_FAILURE: user={}, ip={}, reason={}", username, ip, reason);
    }

    public void logAccessDenied(String username, String resource) {
        log.warn("ACCESS_DENIED: user={}, resource={}", username, resource);
    }

    public void logSuspiciousActivity(String username, String activity) {
        log.error("SUSPICIOUS_ACTIVITY: user={}, activity={}", username, activity);
        // Trigger alert
    }
}
```

### 10. Server-Side Request Forgery (SSRF)

**Solution**: Validate URLs, whitelist domains

```java
@Service
public class WebhookService {

    private static final Set<String> ALLOWED_DOMAINS = Set.of(
        "api.trusted-partner.com",
        "webhook.example.com"
    );

    public void triggerWebhook(String url, Object payload) {
        // ✅ Validate URL
        if (!isAllowedUrl(url)) {
            throw new SecurityException("URL not whitelisted");
        }

        // Make request
        restTemplate.postForObject(url, payload, String.class);
    }

    private boolean isAllowedUrl(String urlString) {
        try {
            URL url = new URL(urlString);
            String host = url.getHost();

            return ALLOWED_DOMAINS.contains(host);
        } catch (MalformedURLException e) {
            return false;
        }
    }
}
```

## Rate Limiting

### Using Bucket4j

**1. Add Dependency**:
```xml
<dependency>
    <groupId>com.github.vladimir-bukhtoyarov</groupId>
    <artifactId>bucket4j-core</artifactId>
    <version>8.5.0</version>
</dependency>
```

**2. Rate Limit Filter**:
```java
@Component
public class RateLimitFilter extends OncePerRequestFilter {

    private final Map<String, Bucket> cache = new ConcurrentHashMap<>();

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {

        String key = getClientKey(request);
        Bucket bucket = resolveBucket(key);

        if (bucket.tryConsume(1)) {
            filterChain.doFilter(request, response);
        } else {
            response.setStatus(HttpStatus.TOO_MANY_REQUESTS.value());
            response.getWriter().write("Rate limit exceeded");
        }
    }

    private Bucket resolveBucket(String key) {
        return cache.computeIfAbsent(key, k -> createNewBucket());
    }

    private Bucket createNewBucket() {
        Bandwidth limit = Bandwidth.classic(100, Refill.intervally(100, Duration.ofMinutes(1)));
        return Bucket.builder()
            .addLimit(limit)
            .build();
    }

    private String getClientKey(HttpServletRequest request) {
        // Use IP address or user ID
        String ip = request.getRemoteAddr();
        return ip != null ? ip : "unknown";
    }
}
```

**3. Method-Level Rate Limiting**:
```java
@RestController
@RequestMapping("/api/v1/auth")
public class AuthController {

    private final LoadingCache<String, AtomicInteger> requestCountsPerIpAddress;

    public AuthController() {
        this.requestCountsPerIpAddress = CacheBuilder.newBuilder()
            .expireAfterWrite(1, TimeUnit.MINUTES)
            .build(new CacheLoader<String, AtomicInteger>() {
                @Override
                public AtomicInteger load(String key) {
                    return new AtomicInteger(0);
                }
            });
    }

    @PostMapping("/login")
    public ResponseEntity<?> login(@RequestBody LoginRequest request,
                                   HttpServletRequest httpRequest) {
        String clientIp = httpRequest.getRemoteAddr();

        // Check rate limit
        AtomicInteger requests = requestCountsPerIpAddress.get(clientIp);
        if (requests.incrementAndGet() > 5) {
            throw new RateLimitExceededException("Too many login attempts");
        }

        return ResponseEntity.ok(authService.login(request));
    }
}
```

## Input Sanitization

### 1. Validation Annotations

```java
@Data
public class CreateUserRequest {

    @NotBlank(message = "Username is required")
    @Size(min = 3, max = 50)
    @Pattern(regexp = "^[a-zA-Z0-9_-]+$", message = "Username contains invalid characters")
    private String username;

    @NotBlank
    @Email
    private String email;

    @NotBlank
    @Size(min = 8, max = 100)
    @Pattern(regexp = "^(?=.*[0-9])(?=.*[a-z])(?=.*[A-Z])(?=.*[@#$%^&+=]).*$",
             message = "Password must contain uppercase, lowercase, number, and special character")
    private String password;

    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Invalid phone number")
    private String phone;
}
```

### 2. Custom Validators

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = NoSQLInjectionValidator.class)
public @interface NoSQLInjection {
    String message() default "Input contains potential SQL injection";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

public class NoSQLInjectionValidator implements ConstraintValidator<NoSQLInjection, String> {

    private static final Pattern SQL_INJECTION_PATTERN =
        Pattern.compile(".*(\\b(SELECT|INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|EXEC|EXECUTE)\\b).*",
                       Pattern.CASE_INSENSITIVE);

    @Override
    public boolean isValid(String value, ConstraintValidatorContext context) {
        if (value == null) {
            return true;
        }
        return !SQL_INJECTION_PATTERN.matcher(value).matches();
    }
}
```

### 3. HTML Sanitization (XSS Prevention)

**Add Dependency**:
```xml
<dependency>
    <groupId>org.owasp.encoder</groupId>
    <artifactId>encoder</artifactId>
    <version>1.2.3</version>
</dependency>
```

**Sanitize Input**:
```java
@Service
public class CommentService {

    public CommentDTO createComment(CreateCommentRequest request) {
        // ✅ Sanitize HTML to prevent XSS
        String safeContent = Encode.forHtml(request.getContent());

        Comment comment = new Comment();
        comment.setContent(safeContent);
        comment.setAuthor(request.getAuthor());

        Comment saved = commentRepository.save(comment);
        return commentMapper.toDTO(saved);
    }
}
```

### 4. Path Traversal Prevention

```java
@Service
public class FileService {

    private static final String BASE_DIR = "/app/uploads";

    public File getFile(String filename) {
        // ✅ Prevent path traversal
        String sanitized = Paths.get(filename).getFileName().toString();

        if (sanitized.contains("..")) {
            throw new SecurityException("Invalid filename");
        }

        File file = new File(BASE_DIR, sanitized);

        // ✅ Verify file is within base directory
        try {
            if (!file.getCanonicalPath().startsWith(new File(BASE_DIR).getCanonicalPath())) {
                throw new SecurityException("Access denied");
            }
        } catch (IOException e) {
            throw new SecurityException("Invalid file path");
        }

        return file;
    }
}
```

## Security Headers

**Configure Security Headers**:
```java
@Configuration
public class SecurityHeadersConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.headers()
            .contentSecurityPolicy("default-src 'self'")
            .and()
            .xssProtection()
            .and()
            .contentTypeOptions()
            .and()
            .frameOptions().deny()
            .and()
            .httpStrictTransportSecurity()
                .includeSubDomains(true)
                .maxAgeInSeconds(31536000);

        return http.build();
    }
}
```

## Security Best Practices

### 1. Never Log Sensitive Data

```java
// ❌ WRONG
log.info("User login: username={}, password={}", username, password);
log.info("Credit card: {}", creditCard);

// ✅ CORRECT
log.info("User login: username={}", username);
log.info("Payment processed: last4={}", creditCard.getLast4Digits());
```

### 2. Fail Securely

```java
// ✅ Return generic error, log details
try {
    authenticateUser(credentials);
} catch (Exception e) {
    log.error("Authentication failed: {}", e.getMessage(), e);
    throw new BadCredentialsException("Invalid credentials");  // Generic message
}
```

### 3. Principle of Least Privilege

```java
// ✅ Different roles with specific permissions
@PreAuthorize("hasAuthority('USER_READ')")
public UserDTO getUser(Long id) { }

@PreAuthorize("hasAuthority('USER_WRITE')")
public UserDTO updateUser(Long id, UpdateUserRequest request) { }

@PreAuthorize("hasAuthority('USER_DELETE')")
public void deleteUser(Long id) { }
```

### 4. Secure Sessions

```yaml
server:
  servlet:
    session:
      cookie:
        http-only: true  # Prevent JavaScript access
        secure: true     # HTTPS only
        same-site: strict  # CSRF protection
      timeout: 30m       # Session timeout
```

## Security Checklist

✅ **Authentication & Authorization**:
- [ ] Strong password policy enforced
- [ ] BCrypt with strength ≥ 12
- [ ] JWT with strong secret (≥ 512 bits)
- [ ] Token expiration configured
- [ ] @PreAuthorize on all endpoints
- [ ] Rate limiting on auth endpoints

✅ **Input Validation**:
- [ ] @Valid on all request bodies
- [ ] Custom validators for complex rules
- [ ] HTML sanitization for user content
- [ ] SQL injection prevention (Specifications)
- [ ] Path traversal prevention

✅ **Data Protection**:
- [ ] HTTPS enforced
- [ ] Sensitive data encrypted at rest
- [ ] Environment variables for secrets
- [ ] HttpOnly cookies
- [ ] CORS properly configured

✅ **Security Headers**:
- [ ] CSP configured
- [ ] X-XSS-Protection enabled
- [ ] X-Content-Type-Options enabled
- [ ] X-Frame-Options enabled
- [ ] HSTS enabled

✅ **Monitoring**:
- [ ] Security events logged
- [ ] Failed login attempts tracked
- [ ] Anomaly detection in place
- [ ] Alerts configured

## Related Documentation

- See [authentication.md](./authentication.md) for JWT setup
- See [authorization.md](./authorization.md) for roles and permissions
- See [../deployment/monitoring.md](../deployment/monitoring.md) for security monitoring
