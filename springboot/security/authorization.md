# Authorization with @PreAuthorize

## Overview
Authorization controls access to resources based on user roles using Spring Security's `@PreAuthorize` annotation.

## Enable Method Security

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // Enable @PreAuthorize
public class WebSecurityConfig {
    // configuration
}
```

## Role System

### Role Enumeration
```java
public enum ERole {
    ROLE_USER,
    ROLE_MODERATOR,
    ROLE_ADMIN
}
```

### Role Entity
```java
@Entity
@Table(name = "roles")
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Role {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    @Enumerated(EnumType.STRING)
    @Column(length = 20)
    private ERole name;
}
```

### User-Role Relationship
```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
}
```

## Role Constants

Create a constants class for role references:

```java
public final class RolesConstants {
    public static final String USER = "ROLE_USER";
    public static final String MODERATOR = "ROLE_MODERATOR";
    public static final String ADMIN = "ROLE_ADMIN";

    private RolesConstants() {
        // Prevent instantiation
    }
}
```

## Using @PreAuthorize in Controllers

### Basic Usage
```java
@RestController
@RequestMapping("/api/v1/user")
public class CreateUserController {

    private final CreateUserService createUserService;

    @PostMapping
    @PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
        return ResponseEntity.ok(createUserService.createUser(request));
    }
}
```

### Multiple Roles (OR)
```java
@GetMapping("/admin")
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).ADMIN) or " +
              "hasRole(T(com.example.myapp.constants.RolesConstants).MODERATOR)")
public ResponseEntity<List<UserDTO>> getAllUsersAdmin() {
    return ResponseEntity.ok(userService.getAllUsers());
}
```

### Multiple Roles (AND)
```java
@DeleteMapping("/force/{id}")
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).ADMIN) and " +
              "hasRole(T(com.example.myapp.constants.RolesConstants).MODERATOR)")
public ResponseEntity<Void> forceDelete(@PathVariable Long id) {
    userService.forceDelete(id);
    return ResponseEntity.noContent().build();
}
```

### Any Role
```java
@GetMapping("/profile")
@PreAuthorize("hasAnyRole(" +
              "T(com.example.myapp.constants.RolesConstants).USER, " +
              "T(com.example.myapp.constants.RolesConstants).ADMIN)")
public ResponseEntity<UserDTO> getProfile() {
    return ResponseEntity.ok(userService.getCurrentUser());
}
```

## Authorization Patterns

### Pattern 1: User-Only Access
```java
@PostMapping
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
    return ResponseEntity.ok(createUserService.createUser(request));
}
```

### Pattern 2: Admin-Only Access
```java
@DeleteMapping("/admin/{id}")
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).ADMIN)")
public ResponseEntity<Void> adminDelete(@PathVariable Long id) {
    adminService.deleteUser(id);
    return ResponseEntity.noContent().build();
}
```

### Pattern 3: Public Endpoints
```java
@GetMapping("/public/users")
// No @PreAuthorize - publicly accessible
public ResponseEntity<List<UserDTO>> getPublicUsers() {
    return ResponseEntity.ok(userService.getPublicUsers());
}
```

But configure in SecurityFilterChain:
```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.authorizeHttpRequests(auth -> auth
        .requestMatchers("/api/v1/auth/**").permitAll()
        .requestMatchers("/api/v1/public/**").permitAll()
        .anyRequest().authenticated()
    );
    return http.build();
}
```

## Resource-Level Authorization

### Check Ownership in Service
```java
@Service
@Slf4j
public class DeleteUserService {

    private final UserRepository userRepository;
    private final UserService userService;

    @Transactional
    public void deleteUser(String code) {
        String username = Utils.getLoggedUsername();
        User user = userService.findByUsername(username);

        User user = userRepository.findByCode(code)
            .orElseThrow(() -> new ResourceNotFoundException("User not found"));

        // Check ownership
        if (!user.getCreatedBy().equals(username)) {
            throw new ForbiddenException("You don't have permission to delete this user");
        }

        userRepository.delete(user);
        log.info("User deleted: {}", code);
    }
}
```

### Custom Permission Checks
```java
@PreAuthorize("@userSecurityService.canAccessUser(#code)")
@GetMapping("/{code}")
public ResponseEntity<UserDTO> getUser(@PathVariable String code) {
    return ResponseEntity.ok(getUserService.getUser(code));
}
```

```java
@Service("userSecurityService")
public class UserSecurityService {

    private final UserRepository userRepository;

    public boolean canAccessUser(String code) {
        String username = Utils.getLoggedUsername();
        return userRepository.existsByCodeAndUsername(code, username);
    }
}
```

## Global Method Security

Apply security at service level if needed:

```java
@Service
public class AdminService {

    @PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).ADMIN)")
    public void performAdminOperation() {
        // Only admins can call this
    }

    @PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
    public void performUserOperation() {
        // Any user can call this
    }
}
```

## Exception Handling

When authorization fails, Spring Security throws `AccessDeniedException`.

Handle in GlobalExceptionHandler:

```java
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException e) {
        log.error("Access denied: {}", e.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.FORBIDDEN.value(),
            "You don't have permission to access this resource",
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }
}
```

## Testing with Roles

### Unit Tests
```java
@WebMvcTest(CreateUserController.class)
class CreateUserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private CreateUserService createUserService;

    @Test
    @WithMockUser(roles = "USER")
    void shouldCreateUserWhenUserRole() throws Exception {
        // Test with USER role
        mockMvc.perform(post("/api/v1/user")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
            .andExpect(status().isCreated());
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    void shouldCreateUserWhenAdminRole() throws Exception {
        // Test with ADMIN role
        mockMvc.perform(post("/api/v1/user")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
            .andExpect(status().isCreated());
    }

    @Test
    @WithAnonymousUser
    void shouldReturn401WhenNotAuthenticated() throws Exception {
        // Test without authentication
        mockMvc.perform(post("/api/v1/user")
                .contentType(MediaType.APPLICATION_JSON)
                .content(requestJson))
            .andExpect(status().isUnauthorized());
    }
}
```

### Integration Tests
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class UserAuthorizationIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void shouldReturn403WhenInsufficientRole() {
        // Create user with USER role
        String userToken = createUserAndGetToken("user", "USER");

        // Try to access admin endpoint
        HttpHeaders headers = new HttpHeaders();
        headers.add(HttpHeaders.COOKIE, "myapp=" + userToken);

        ResponseEntity<String> response = restTemplate.exchange(
            "/api/v1/admin/users",
            HttpMethod.GET,
            new HttpEntity<>(headers),
            String.class
        );

        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);
    }
}
```

## Best Practices

1. ✅ Use `@PreAuthorize` on controller methods
2. ✅ Reference role constants using `T(...)` syntax
3. ✅ Check resource ownership in services
4. ✅ Use custom security services for complex checks
5. ✅ Handle `AccessDeniedException` in global handler
6. ✅ Test with different roles using `@WithMockUser`
7. ✅ Use `hasRole()` or `hasAnyRole()` appropriately
8. ✅ Enable method security with `@EnableMethodSecurity`
9. ❌ NO hardcoded role strings in `@PreAuthorize`
10. ❌ NO authorization logic in controllers
11. ❌ NO skipping authorization checks

## Common Role Patterns

### Public Endpoint
```java
// In SecurityFilterChain
.requestMatchers("/api/v1/auth/**").permitAll()
```

### Authenticated Users
```java
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
```

### Admin Only
```java
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).ADMIN)")
```

### Admin or Moderator
```java
@PreAuthorize("hasAnyRole(" +
              "T(com.example.myapp.constants.RolesConstants).ADMIN, " +
              "T(com.example.myapp.constants.RolesConstants).MODERATOR)")
```

### Custom Permissions
```java
@PreAuthorize("@securityService.canAccess(#id)")
```

## Related Documentation
- See [authentication.md](./authentication.md) for JWT setup
- See [cors.md](./cors.md) for CORS configuration
- See [controllers.md](../backend/controllers.md) for controller patterns
- See [global-handler.md](../error-handling/global-handler.md) for exception handling
