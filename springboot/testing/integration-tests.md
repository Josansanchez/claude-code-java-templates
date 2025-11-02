# Integration Tests

## Overview
Integration tests verify the entire application stack working together, including database, security, and all layers.

## @SpringBootTest Structure

### Basic Integration Test
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class UserIntegrationTest {

    @LocalServerPort
    private int port;

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private JwtUtils jwtUtils;

    @BeforeEach
    void setUp() {
        userRepository.deleteAll();
    }

    @Test
    void shouldCreateAndRetrieveUser() {
        // Given
        String token = jwtUtils.generateToken("testuser");
        HttpHeaders headers = new HttpHeaders();
        headers.add(HttpHeaders.COOKIE, jwtUtils.getJwtCookieName() + "=" + token);

        CreateUserRequest request = new CreateUserRequest("Integration User", "password");
        HttpEntity<CreateUserRequest> entity = new HttpEntity<>(request, headers);

        // When - Create
        ResponseEntity<UserDTO> createResponse = restTemplate.exchange(
            "http://localhost:" + port + "/api/v1/user",
            HttpMethod.POST,
            entity,
            UserDTO.class
        );

        // Then - Create
        assertThat(createResponse.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        assertThat(createResponse.getBody()).isNotNull();
        String code = createResponse.getBody().getCode();

        // When - Retrieve
        HttpEntity<Void> getEntity = new HttpEntity<>(headers);
        ResponseEntity<UserDTO> getResponse = restTemplate.exchange(
            "http://localhost:" + port + "/api/v1/user/" + code,
            HttpMethod.GET,
            getEntity,
            UserDTO.class
        );

        // Then - Retrieve
        assertThat(getResponse.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(getResponse.getBody().getName()).isEqualTo("Integration User");
    }

    @Test
    void shouldReturn404WhenUserNotFound() {
        // Given
        String token = jwtUtils.generateToken("testuser");
        HttpHeaders headers = new HttpHeaders();
        headers.add(HttpHeaders.COOKIE, jwtUtils.getJwtCookieName() + "=" + token);

        // When
        HttpEntity<Void> entity = new HttpEntity<>(headers);
        ResponseEntity<String> response = restTemplate.exchange(
            "http://localhost:" + port + "/api/v1/user/NOTFOUND",
            HttpMethod.GET,
            entity,
            String.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NOT_FOUND);
    }

    @Test
    void shouldUpdateUser() {
        // Given - Create user
        User user = User.builder()
            .name("Original Name")
            .code("UPDATE1")
            .build();
        userRepository.save(user);

        String token = jwtUtils.generateToken("testuser");
        HttpHeaders headers = new HttpHeaders();
        headers.add(HttpHeaders.COOKIE, jwtUtils.getJwtCookieName() + "=" + token);

        UpdateUserRequest updateRequest = new UpdateUserRequest("Updated Name");
        HttpEntity<UpdateUserRequest> entity = new HttpEntity<>(updateRequest, headers);

        // When
        ResponseEntity<UserDTO> response = restTemplate.exchange(
            "http://localhost:" + port + "/api/v1/user/UPDATE1",
            HttpMethod.PUT,
            entity,
            UserDTO.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody().getName()).isEqualTo("Updated Name");

        // Verify in database
        User updated = userRepository.findByCode("UPDATE1").orElseThrow();
        assertThat(updated.getName()).isEqualTo("Updated Name");
    }

    @Test
    void shouldDeleteUser() {
        // Given
        User user = User.builder()
            .name("To Delete")
            .code("DELETE1")
            .build();
        userRepository.save(user);

        String token = jwtUtils.generateToken("testuser");
        HttpHeaders headers = new HttpHeaders();
        headers.add(HttpHeaders.COOKIE, jwtUtils.getJwtCookieName() + "=" + token);

        // When
        HttpEntity<Void> entity = new HttpEntity<>(headers);
        ResponseEntity<Void> response = restTemplate.exchange(
            "http://localhost:" + port + "/api/v1/user/DELETE1",
            HttpMethod.DELETE,
            entity,
            Void.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NO_CONTENT);

        // Verify deleted from database
        Optional<User> found = userRepository.findByCode("DELETE1");
        assertThat(found).isEmpty();
    }
}
```

## Using Testcontainers

### MySQL Testcontainer
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
@Testcontainers
class UserIntegrationTestWithContainer {

    @Container
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8.0")
        .withDatabaseName("testdb")
        .withUsername("test")
        .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", mysql::getJdbcUrl);
        registry.add("spring.datasource.username", mysql::getUsername);
        registry.add("spring.datasource.password", mysql::getPassword);
    }

    @Autowired
    private TestRestTemplate restTemplate;

    @LocalServerPort
    private int port;

    @Test
    void shouldWorkWithRealDatabase() {
        // Test with real MySQL container
    }
}
```

### application-test.yml for Testcontainers
```yaml
spring:
  datasource:
    url: jdbc:tc:mysql:8.0:///testdb
    driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver
  jpa:
    hibernate:
      ddl-auto: create-drop
```

## Authentication Integration Tests

### Login Flow Test
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class AuthenticationIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private PasswordEncoder passwordEncoder;

    @LocalServerPort
    private int port;

    @BeforeEach
    void setUp() {
        // Create test user
        User user = User.builder()
            .username("testuser")
            .password(passwordEncoder.encode("password123"))
            .email("test@example.com")
            .build();
        userRepository.save(user);
    }

    @Test
    void shouldLoginSuccessfully() {
        // Given
        LoginRequest loginRequest = new LoginRequest("testuser", "password123");

        // When
        ResponseEntity<UserInfoResponse> response = restTemplate.postForEntity(
            "http://localhost:" + port + "/api/v1/auth/signin",
            loginRequest,
            UserInfoResponse.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody()).isNotNull();
        assertThat(response.getBody().getUsername()).isEqualTo("testuser");

        // Verify JWT cookie is set
        String cookies = response.getHeaders().getFirst(HttpHeaders.SET_COOKIE);
        assertThat(cookies).contains("myapp=");
    }

    @Test
    void shouldFailLoginWithInvalidCredentials() {
        // Given
        LoginRequest loginRequest = new LoginRequest("testuser", "wrongpassword");

        // When
        ResponseEntity<String> response = restTemplate.postForEntity(
            "http://localhost:" + port + "/api/v1/auth/signin",
            loginRequest,
            String.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.UNAUTHORIZED);
    }

    @Test
    void shouldLogoutSuccessfully() {
        // Given - Login first
        String token = loginAndGetToken();
        HttpHeaders headers = new HttpHeaders();
        headers.add(HttpHeaders.COOKIE, "myapp=" + token);

        // When - Logout
        HttpEntity<Void> entity = new HttpEntity<>(headers);
        ResponseEntity<MessageResponse> response = restTemplate.exchange(
            "http://localhost:" + port + "/api/v1/auth/signout",
            HttpMethod.POST,
            entity,
            MessageResponse.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);

        // Verify cookie is cleared
        String cookies = response.getHeaders().getFirst(HttpHeaders.SET_COOKIE);
        assertThat(cookies).contains("Max-Age=0");
    }
}
```

## Authorization Integration Tests

### Role-Based Access Test
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class AuthorizationIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @LocalServerPort
    private int port;

    @Test
    void shouldDenyAccessWhenNotAuthenticated() {
        // When
        ResponseEntity<String> response = restTemplate.getForEntity(
            "http://localhost:" + port + "/api/v1/user",
            String.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.UNAUTHORIZED);
    }

    @Test
    void shouldAllowAccessWithUserRole() {
        // Given
        String userToken = createTokenForRole("USER");
        HttpHeaders headers = new HttpHeaders();
        headers.add(HttpHeaders.COOKIE, "myapp=" + userToken);

        // When
        HttpEntity<Void> entity = new HttpEntity<>(headers);
        ResponseEntity<String> response = restTemplate.exchange(
            "http://localhost:" + port + "/api/v1/user",
            HttpMethod.GET,
            entity,
            String.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
    }

    @Test
    void shouldDenyAccessWithInsufficientRole() {
        // Given
        String userToken = createTokenForRole("USER");
        HttpHeaders headers = new HttpHeaders();
        headers.add(HttpHeaders.COOKIE, "myapp=" + userToken);

        // When - Try to access admin endpoint
        HttpEntity<Void> entity = new HttpEntity<>(headers);
        ResponseEntity<String> response = restTemplate.exchange(
            "http://localhost:" + port + "/api/v1/admin/users",
            HttpMethod.GET,
            entity,
            String.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);
    }
}
```

## Database Integration Tests

### Transaction Rollback Test
```java
@SpringBootTest
@ActiveProfiles("test")
@Transactional
class TransactionIntegrationTest {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private CreateUserService createUserService;

    @Test
    @Rollback  // Default behavior, included for clarity
    void shouldRollbackOnException() {
        // Given
        CreateUserRequest request = new CreateUserRequest("Test", "password");

        // When - Force an exception during creation
        assertThatThrownBy(() -> {
            createUserService.createUser(request);
            throw new RuntimeException("Simulated error");
        });

        // Then - Verify nothing was saved (transaction rolled back)
        List<User> users = userRepository.findAll();
        assertThat(users).isEmpty();
    }
}
```

## Complete CRUD Integration Test

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class CompleteCRUDIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private UserRepository userRepository;

    @LocalServerPort
    private int port;

    private static String createdUserCode;
    private static String authToken;

    @BeforeAll
    static void setupAuth(@Autowired JwtUtils jwtUtils) {
        authToken = jwtUtils.generateToken("testuser");
    }

    @Test
    @Order(1)
    void step1_shouldCreateUser() {
        // Given
        CreateUserRequest request = new CreateUserRequest("Test User", "password");
        HttpHeaders headers = createAuthHeaders();
        HttpEntity<CreateUserRequest> entity = new HttpEntity<>(request, headers);

        // When
        ResponseEntity<UserDTO> response = restTemplate.exchange(
            baseUrl() + "/api/v1/user",
            HttpMethod.POST,
            entity,
            UserDTO.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.CREATED);
        createdUserCode = response.getBody().getCode();
        assertThat(createdUserCode).isNotNull();
    }

    @Test
    @Order(2)
    void step2_shouldRetrieveCreatedUser() {
        // Given
        HttpHeaders headers = createAuthHeaders();
        HttpEntity<Void> entity = new HttpEntity<>(headers);

        // When
        ResponseEntity<UserDTO> response = restTemplate.exchange(
            baseUrl() + "/api/v1/user/" + createdUserCode,
            HttpMethod.GET,
            entity,
            UserDTO.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody().getName()).isEqualTo("Test User");
    }

    @Test
    @Order(3)
    void step3_shouldUpdateUser() {
        // Given
        UpdateUserRequest request = new UpdateUserRequest("Updated User");
        HttpHeaders headers = createAuthHeaders();
        HttpEntity<UpdateUserRequest> entity = new HttpEntity<>(request, headers);

        // When
        ResponseEntity<UserDTO> response = restTemplate.exchange(
            baseUrl() + "/api/v1/user/" + createdUserCode,
            HttpMethod.PUT,
            entity,
            UserDTO.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody().getName()).isEqualTo("Updated User");
    }

    @Test
    @Order(4)
    void step4_shouldDeleteUser() {
        // Given
        HttpHeaders headers = createAuthHeaders();
        HttpEntity<Void> entity = new HttpEntity<>(headers);

        // When
        ResponseEntity<Void> response = restTemplate.exchange(
            baseUrl() + "/api/v1/user/" + createdUserCode,
            HttpMethod.DELETE,
            entity,
            Void.class
        );

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.NO_CONTENT);

        // Verify deletion
        Optional<User> found = userRepository.findByCode(createdUserCode);
        assertThat(found).isEmpty();
    }

    private HttpHeaders createAuthHeaders() {
        HttpHeaders headers = new HttpHeaders();
        headers.add(HttpHeaders.COOKIE, "myapp=" + authToken);
        return headers;
    }

    private String baseUrl() {
        return "http://localhost:" + port;
    }
}
```

## Best Practices

1. ✅ Use `@SpringBootTest` for integration tests
2. ✅ Use `RANDOM_PORT` for web environment
3. ✅ Use `TestRestTemplate` for HTTP calls
4. ✅ Clean database before each test (`@BeforeEach`)
5. ✅ Test complete workflows (create → read → update → delete)
6. ✅ Use Testcontainers for real database testing
7. ✅ Test authentication and authorization flows
8. ✅ Verify database state after operations
9. ✅ Use `@ActiveProfiles("test")` for test configuration
10. ❌ NO integration tests for every scenario (use unit tests)
11. ❌ NO shared state between tests
12. ❌ NO relying on test execution order (unless using `@TestMethodOrder`)

## When to Use Integration Tests

### Use Integration Tests For:
- ✅ Complete API workflows
- ✅ Authentication/Authorization flows
- ✅ Database transactions
- ✅ Cross-layer interactions
- ✅ External service integrations

### Use Unit Tests For:
- ✅ Business logic
- ✅ Validation rules
- ✅ Mappers
- ✅ Individual methods
- ✅ Error handling

## Related Documentation
- See [unit-tests.md](./unit-tests.md) for unit testing patterns
- See [test-conventions.md](./test-conventions.md) for naming conventions
- See [dependencies.md](../configuration/dependencies.md) for test dependencies
