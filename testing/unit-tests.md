# Unit Tests

## Overview
Unit tests verify individual components in isolation using mocks. They are fast and focused on specific functionality.

## Testing Services with Mockito

### Basic Service Test Structure
```java
@ExtendWith(MockitoExtension.class)
class CreateUserServiceTest {

    @Mock
    private UserRepository userRepository;

    @Mock
    private UserMapper userMapper;

    @Mock
    private UserService userService;

    @InjectMocks
    private CreateUserService createUserService;

    @Test
    void shouldCreateUserWhenValidRequest() {
        // Given
        CreateUserRequest request = new CreateUserRequest("My User", "password");
        User user = User.builder().id(1L).username("testuser").build();
        User user = User.builder().id(1L).name("My User").code("ABC12").build();
        UserDTO expected = new UserDTO(1L, "My User", "ABC12");

        when(userService.findByUsername(anyString())).thenReturn(user);
        when(userMapper.toEntity(request)).thenReturn(user);
        when(userRepository.save(any(User.class))).thenReturn(user);
        when(userMapper.toDTO(user)).thenReturn(expected);

        // When
        UserDTO result = createUserService.createUser(request);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getName()).isEqualTo("My User");
        assertThat(result.getCode()).isEqualTo("ABC12");

        verify(userRepository, times(1)).save(any(User.class));
        verify(userMapper, times(1)).toDTO(user);
    }

    @Test
    void shouldThrowExceptionWhenUserAlreadyExists() {
        // Given
        CreateUserRequest request = new CreateUserRequest("Existing User", "password");
        when(userRepository.existsByName(request.getName())).thenReturn(true);

        // When & Then
        assertThatThrownBy(() -> createUserService.createUser(request))
            .isInstanceOf(ResourceAlreadyExistsException.class)
            .hasMessageContaining("User already exists");

        verify(userRepository, never()).save(any());
    }
}
```

## Testing Controllers with @WebMvcTest

### Basic Controller Test Structure
```java
@WebMvcTest(CreateUserController.class)
@ActiveProfiles("test")
class CreateUserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private CreateUserService createUserService;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @WithMockUser(roles = "USER")
    void shouldCreateUserWhenValidRequest() throws Exception {
        // Given
        CreateUserRequest request = new CreateUserRequest("My User", "password123");
        UserDTO response = new UserDTO(1L, "My User", "ABC12");

        when(createUserService.createUser(any())).thenReturn(response);

        // When & Then
        mockMvc.perform(post("/api/v1/user")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated())
            .andExpect(header().exists("Location"))
            .andExpect(jsonPath("$.name").value("My User"))
            .andExpect(jsonPath("$.code").value("ABC12"));

        verify(createUserService, times(1)).createUser(any());
    }

    @Test
    @WithMockUser(roles = "USER")
    void shouldReturnBadRequestWhenInvalidInput() throws Exception {
        // Given
        CreateUserRequest request = new CreateUserRequest("", "");  // Invalid

        // When & Then
        mockMvc.perform(post("/api/v1/user")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest());

        verify(createUserService, never()).createUser(any());
    }

    @Test
    @WithAnonymousUser
    void shouldReturn401WhenNotAuthenticated() throws Exception {
        // Given
        CreateUserRequest request = new CreateUserRequest("My User", "password");

        // When & Then
        mockMvc.perform(post("/api/v1/user")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isUnauthorized());
    }

    @Test
    @WithMockUser(roles = "ADMIN")
    void shouldCreateUserWhenAdminRole() throws Exception {
        // Given
        CreateUserRequest request = new CreateUserRequest("Admin User", "password");
        UserDTO response = new UserDTO(1L, "Admin User", "XYZ99");

        when(createUserService.createUser(any())).thenReturn(response);

        // When & Then
        mockMvc.perform(post("/api/v1/user")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isCreated());
    }
}
```

## Testing Repositories with @DataJpaTest

### Basic Repository Test Structure
```java
@DataJpaTest
@ActiveProfiles("test")
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Autowired
    private TestEntityManager entityManager;

    @Test
    void shouldFindUserByCode() {
        // Given
        User user = User.builder()
            .name("Test User")
            .code("TEST1")
            .build();
        entityManager.persist(user);
        entityManager.flush();

        // When
        Optional<User> found = userRepository.findByCode("TEST1");

        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("Test User");
    }

    @Test
    void shouldReturnEmptyWhenCodeNotFound() {
        // When
        Optional<User> found = userRepository.findByCode("NOTEXIST");

        // Then
        assertThat(found).isEmpty();
    }

    @Test
    void shouldCheckExistenceByCode() {
        // Given
        User user = User.builder()
            .name("Test User")
            .code("EXISTS")
            .build();
        entityManager.persist(user);
        entityManager.flush();

        // When
        boolean exists = userRepository.existsByCode("EXISTS");
        boolean notExists = userRepository.existsByCode("NOTEXIST");

        // Then
        assertThat(exists).isTrue();
        assertThat(notExists).isFalse();
    }

    @Test
    void shouldDeleteByCode() {
        // Given
        User user = User.builder()
            .name("Test User")
            .code("DELETE")
            .build();
        entityManager.persist(user);
        entityManager.flush();

        // When
        userRepository.deleteByCode("DELETE");

        // Then
        Optional<User> found = userRepository.findByCode("DELETE");
        assertThat(found).isEmpty();
    }
}
```

## Mockito Patterns

### Pattern 1: Simple Mock
```java
@Mock
private UserRepository userRepository;

when(userRepository.findByCode("ABC")).thenReturn(Optional.of(user));
```

### Pattern 2: Argument Matchers
```java
when(userRepository.save(any(User.class))).thenReturn(savedUser);
when(userService.findByUsername(anyString())).thenReturn(user);
when(userRepository.existsByCodeAndUserId(eq("ABC"), anyLong())).thenReturn(true);
```

### Pattern 3: Verify Interactions
```java
verify(userRepository).save(any(User.class));
verify(userRepository, times(1)).save(any(User.class));
verify(userRepository, never()).delete(any());
verify(userRepository, atLeastOnce()).findByCode(anyString());
```

### Pattern 4: Argument Captor
```java
@Captor
private ArgumentCaptor<User> userCaptor;

createUserService.createUser(request);

verify(userRepository).save(userCaptor.capture());
User captured = userCaptor.getValue();
assertThat(captured.getName()).isEqualTo("My User");
```

### Pattern 5: Throwing Exceptions
```java
when(userRepository.findByCode("ERROR"))
    .thenThrow(new ResourceNotFoundException("User not found"));
```

### Pattern 6: Void Methods
```java
doNothing().when(userRepository).delete(any(User.class));
doThrow(new RuntimeException()).when(userRepository).delete(any(User.class));
```

## MockMvc Patterns

### Pattern 1: GET Request
```java
mockMvc.perform(get("/api/v1/user/{code}", "ABC123"))
    .andExpect(status().isOk())
    .andExpect(jsonPath("$.code").value("ABC123"));
```

### Pattern 2: POST Request
```java
mockMvc.perform(post("/api/v1/user")
        .contentType(MediaType.APPLICATION_JSON)
        .content(objectMapper.writeValueAsString(request)))
    .andExpect(status().isCreated())
    .andExpect(header().exists("Location"));
```

### Pattern 3: PUT Request
```java
mockMvc.perform(put("/api/v1/user/{code}", "ABC123")
        .contentType(MediaType.APPLICATION_JSON)
        .content(objectMapper.writeValueAsString(updateRequest)))
    .andExpect(status().isOk());
```

### Pattern 4: DELETE Request
```java
mockMvc.perform(delete("/api/v1/user/{code}", "ABC123"))
    .andExpect(status().isNoContent());
```

### Pattern 5: With Headers
```java
mockMvc.perform(get("/api/v1/user")
        .header("Authorization", "Bearer token"))
    .andExpect(status().isOk());
```

### Pattern 6: JSON Path Assertions
```java
mockMvc.perform(get("/api/v1/user/{code}", "ABC"))
    .andExpect(jsonPath("$.id").value(1))
    .andExpect(jsonPath("$.name").value("My User"))
    .andExpect(jsonPath("$.orders").isArray())
    .andExpect(jsonPath("$.orders", hasSize(3)))
    .andExpect(jsonPath("$.orders[0].name").value("Living Order"));
```

## AssertJ Assertions

### Basic Assertions
```java
assertThat(result).isNotNull();
assertThat(result.getName()).isEqualTo("My User");
assertThat(result.getCode()).isEqualTo("ABC123");
assertThat(result.getOrderCount()).isGreaterThan(0);
```

### Collection Assertions
```java
assertThat(list).hasSize(3);
assertThat(list).isEmpty();
assertThat(list).isNotEmpty();
assertThat(list).contains(item);
assertThat(list).containsExactly(item1, item2);
assertThat(list).extracting("name").contains("User1", "User2");
```

### Exception Assertions
```java
assertThatThrownBy(() -> service.getUser("INVALID"))
    .isInstanceOf(ResourceNotFoundException.class)
    .hasMessageContaining("not found");

assertThatCode(() -> service.createUser(request))
    .doesNotThrowAnyException();
```

### Optional Assertions
```java
assertThat(optional).isPresent();
assertThat(optional).isEmpty();
assertThat(optional).contains(expectedValue);
```

## Test Naming Conventions

### Pattern: `shouldXxxWhenYyy`
```java
shouldCreateUserWhenValidRequest()
shouldThrowExceptionWhenUserNotFound()
shouldReturn404WhenResourceNotFound()
shouldUpdateUserWhenUserIsOwner()
shouldReturnEmptyListWhenNoUsersExist()
```

## Given-When-Then Structure

```java
@Test
void shouldCreateUserWhenValidRequest() {
    // Given - Setup test data and mocks
    CreateUserRequest request = new CreateUserRequest("My User", "password");
    User user = User.builder().name("My User").build();
    when(userMapper.toEntity(request)).thenReturn(user);

    // When - Execute the method under test
    UserDTO result = createUserService.createUser(request);

    // Then - Verify the results
    assertThat(result).isNotNull();
    verify(userRepository).save(any(User.class));
}
```

## Testing Security

### With @WithMockUser
```java
@Test
@WithMockUser(username = "testuser", roles = "USER")
void shouldAllowUserToCreateUser() throws Exception {
    mockMvc.perform(post("/api/v1/user")
            .contentType(MediaType.APPLICATION_JSON)
            .content(requestJson))
        .andExpect(status().isCreated());
}

@Test
@WithMockUser(roles = "ADMIN")
void shouldAllowAdminToDeleteUser() throws Exception {
    mockMvc.perform(delete("/api/v1/user/{code}", "ABC"))
        .andExpect(status().isNoContent());
}

@Test
@WithAnonymousUser
void shouldDenyAccessToUnauthenticatedUser() throws Exception {
    mockMvc.perform(get("/api/v1/user"))
        .andExpect(status().isUnauthorized());
}
```

## Test Configuration

### Test Profile
```yaml
# application-test.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

myapp:
  app:
    jwtSecret: test-secret
    jwtExpirationMs: 3600000
```

## Best Practices

1. ✅ Use `@ExtendWith(MockitoExtension.class)` for service tests
2. ✅ Use `@WebMvcTest` for controller tests (lightweight)
3. ✅ Use `@DataJpaTest` for repository tests
4. ✅ Use Given-When-Then structure
5. ✅ Name tests with `shouldXxxWhenYyy` pattern
6. ✅ Test both happy path and error cases
7. ✅ Use AssertJ for fluent assertions
8. ✅ Verify mock interactions
9. ✅ Use `@WithMockUser` for security tests
10. ✅ Keep tests focused and isolated
11. ❌ NO `@SpringBootTest` for unit tests (too heavy)
12. ❌ NO real database connections in unit tests
13. ❌ NO testing implementation details

## Related Documentation
- See [integration-tests.md](./integration-tests.md) for full integration tests
- See [test-conventions.md](./test-conventions.md) for naming and structure
- See [services.md](../backend/services.md) for service patterns
- See [controllers.md](../backend/controllers.md) for controller patterns
