# Test Conventions

## Test Naming Conventions

### Test Class Names
```java
// Pattern: <ClassName>Test
CreateUserServiceTest
GetUserControllerTest
UserRepositoryTest
UserIntegrationTest
```

### Test Method Names

#### Pattern: `shouldXxxWhenYyy`
```java
// Service tests
shouldCreateUserWhenValidRequest()
shouldThrowExceptionWhenUserNotFound()
shouldUpdateUserWhenUserIsOwner()
shouldDeleteUserWhenUserExists()

// Controller tests
shouldReturn201WhenUserCreated()
shouldReturn404WhenUserNotFound()
shouldReturn400WhenInvalidInput()
shouldReturn401WhenNotAuthenticated()

// Repository tests
shouldFindUserByCode()
shouldReturnEmptyWhenCodeNotFound()
shouldSaveUserSuccessfully()
```

## Given-When-Then Structure

### Standard Structure
```java
@Test
void shouldCreateUserWhenValidRequest() {
    // Given - Setup test data and mocks
    CreateUserRequest request = new CreateUserRequest("My User", "password");
    User user = User.builder().name("My User").build();
    when(userMapper.toEntity(request)).thenReturn(user);
    when(userRepository.save(any(User.class))).thenReturn(user);

    // When - Execute the method under test
    UserDTO result = createUserService.createUser(request);

    // Then - Verify the results
    assertThat(result).isNotNull();
    assertThat(result.getName()).isEqualTo("My User");
    verify(userRepository).save(any(User.class));
}
```

### When & Then Combined (for assertions)
```java
@Test
void shouldThrowExceptionWhenUserNotFound() {
    // Given
    String code = "NOTFOUND";
    when(userRepository.findByCode(code)).thenReturn(Optional.empty());

    // When & Then
    assertThatThrownBy(() -> getUserService.getUser(code))
        .isInstanceOf(ResourceNotFoundException.class)
        .hasMessageContaining("User not found");
}
```

## Test Organization

### Directory Structure
```
src/test/java/com/example/myapp/
├── models/              # Entity tests
│   ├── UserTest.java
│   ├── ProductTest.java
│   └── UserTest.java
├── user/
│   ├── service/         # Service tests
│   │   ├── CreateUserServiceTest.java
│   │   └── GetUserServiceTest.java
│   └── controller/      # Controller tests
│       ├── CreateUserControllerTest.java
│       └── GetUserControllerTest.java
├── integration/         # Integration tests
│   └── UserIntegrationTest.java
└── util/                # Test utilities
    └── TestDataBuilder.java
```

**Note**: All test directories should be in lowercase.

## Test Data Builders

### Builder Pattern for Test Data
```java
public class UserTestDataBuilder {

    public static User.UserBuilder defaultUser() {
        return User.builder()
            .id(1L)
            .name("Test User")
            .code("TEST1")
            .createdAt(LocalDateTime.now());
    }

    public static User createTestUser() {
        return defaultUser().build();
    }

    public static User createTestUserWithCode(String code) {
        return defaultUser()
            .code(code)
            .build();
    }

    public static CreateUserRequest defaultCreateRequest() {
        return new CreateUserRequest("Test User", "password123");
    }

    public static UserDTO defaultDTO() {
        return new UserDTO(1L, "Test User", "TEST1");
    }
}
```

**Usage**:
```java
@Test
void shouldCreateUserWhenValidRequest() {
    // Given
    User user = UserTestDataBuilder.createTestUser();
    CreateUserRequest request = UserTestDataBuilder.defaultCreateRequest();

    when(userMapper.toEntity(request)).thenReturn(user);
    when(userRepository.save(any())).thenReturn(user);

    // When
    UserDTO result = createUserService.createUser(request);

    // Then
    assertThat(result).isNotNull();
}
```

## Test Categories

### 1. Unit Tests
```java
@ExtendWith(MockitoExtension.class)
class CreateUserServiceTest {
    // Tests for individual service methods
}
```

### 2. Controller Tests
```java
@WebMvcTest(CreateUserController.class)
@ActiveProfiles("test")
class CreateUserControllerTest {
    // Tests for HTTP endpoints
}
```

### 3. Repository Tests
```java
@DataJpaTest
@ActiveProfiles("test")
class UserRepositoryTest {
    // Tests for database queries
}
```

### 4. Integration Tests
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@ActiveProfiles("test")
class UserIntegrationTest {
    // Tests for complete workflows
}
```

## Assertion Patterns

### AssertJ (Preferred)
```java
// Basic assertions
assertThat(result).isNotNull();
assertThat(result.getName()).isEqualTo("Expected");
assertThat(result.getCount()).isGreaterThan(0);

// Collections
assertThat(list).hasSize(3);
assertThat(list).contains(item);
assertThat(list).extracting("name").contains("User1");

// Exceptions
assertThatThrownBy(() -> service.method())
    .isInstanceOf(Exception.class)
    .hasMessageContaining("error");

// Optional
assertThat(optional).isPresent();
assertThat(optional).contains(expected);
```

### JUnit Assertions (Alternative)
```java
assertEquals("Expected", result.getName());
assertTrue(result.isActive());
assertNotNull(result);
assertThrows(Exception.class, () -> service.method());
```

## Mock Verification Patterns

### Verify Method Calls
```java
// Verify called once
verify(repository).save(any(User.class));
verify(repository, times(1)).save(any());

// Verify never called
verify(repository, never()).delete(any());

// Verify called at least once
verify(repository, atLeastOnce()).findByCode(anyString());

// Verify call order
InOrder inOrder = inOrder(repository, mapper);
inOrder.verify(repository).findByCode("ABC");
inOrder.verify(mapper).toDTO(any());
```

### Argument Captor
```java
@Captor
private ArgumentCaptor<User> userCaptor;

@Test
void shouldSaveUserWithCorrectData() {
    // When
    createUserService.createUser(request);

    // Then
    verify(userRepository).save(userCaptor.capture());
    User captured = userCaptor.getValue();
    assertThat(captured.getName()).isEqualTo("My User");
    assertThat(captured.getCode()).isNotNull();
}
```

## Test Coverage Guidelines

### Minimum Coverage
- Overall: 80%
- Services: 90%
- Controllers: 85%
- Repositories: 70%

### What to Test
- ✅ Happy path scenarios
- ✅ Error scenarios
- ✅ Edge cases
- ✅ Validation rules
- ✅ Business logic
- ✅ Exception handling

### What NOT to Test
- ❌ Getters/Setters
- ❌ Lombok-generated code
- ❌ Spring framework code
- ❌ Configuration classes (unless complex logic)

## Test Isolation

### Each Test Should Be Independent
```java
@BeforeEach
void setUp() {
    // Reset mocks
    reset(userRepository, userMapper);

    // Clear test data
    testData = new ArrayList<>();
}

@AfterEach
void tearDown() {
    // Clean up if needed
}
```

### No Shared State
```java
// ❌ BAD - Shared mutable state
private static List<User> sharedList = new ArrayList<>();

// ✅ GOOD - Each test creates its own data
@Test
void test1() {
    List<User> users = new ArrayList<>();
    // use users
}

@Test
void test2() {
    List<User> users = new ArrayList<>();
    // use users
}
```

## Parameterized Tests

### Multiple Test Cases
```java
@ParameterizedTest
@CsvSource({
    "ABC123, true",
    "XYZ789, true",
    "INVALID, false"
})
void shouldValidateUserCode(String code, boolean expected) {
    boolean result = validationService.isValidCode(code);
    assertThat(result).isEqualTo(expected);
}
```

### Method Source
```java
@ParameterizedTest
@MethodSource("provideUserCodes")
void shouldFindUserByCode(String code) {
    Optional<User> result = userRepository.findByCode(code);
    assertThat(result).isPresent();
}

static Stream<String> provideUserCodes() {
    return Stream.of("ABC123", "XYZ789", "TEST1");
}
```

## Test Documentation

### Document Complex Tests
```java
/**
 * Tests the user creation workflow including:
 * 1. User authentication
 * 2. Permission validation
 * 3. Code generation
 * 4. Database persistence
 */
@Test
void shouldCreateUserWithCompleteWorkflow() {
    // Test implementation
}
```

### Use Descriptive Display Names
```java
@Test
@DisplayName("Should create user when user has USER role and valid input")
void shouldCreateUserWhenValidRequest() {
    // Test implementation
}
```

## Test Performance

### Fast Tests
- Unit tests should run in milliseconds
- Integration tests should run in seconds
- Full test suite should run in minutes

### Optimize Slow Tests
```java
// ❌ SLOW - Multiple database calls
@Test
void slowTest() {
    for (int i = 0; i < 100; i++) {
        repository.save(createUser(i));
    }
}

// ✅ FAST - Batch operation
@Test
void fastTest() {
    List<User> users = IntStream.range(0, 100)
        .mapToObj(this::createUser)
        .collect(Collectors.toList());
    repository.saveAll(users);
}
```

## Common Patterns

### Pattern 1: Test Exceptions
```java
@Test
void shouldThrowExceptionWhenUserNotFound() {
    when(repository.findByCode("INVALID")).thenReturn(Optional.empty());

    assertThatThrownBy(() -> service.getUser("INVALID"))
        .isInstanceOf(ResourceNotFoundException.class)
        .hasMessageContaining("User not found");
}
```

### Pattern 2: Test Validation
```java
@Test
void shouldRejectInvalidInput() {
    CreateUserRequest request = new CreateUserRequest("", "");

    mockMvc.perform(post("/api/v1/user")
            .contentType(MediaType.APPLICATION_JSON)
            .content(objectMapper.writeValueAsString(request)))
        .andExpect(status().isBadRequest());
}
```

### Pattern 3: Test Authentication
```java
@Test
@WithMockUser(roles = "USER")
void shouldAllowAuthenticatedUser() {
    mockMvc.perform(get("/api/v1/user"))
        .andExpect(status().isOk());
}

@Test
@WithAnonymousUser
void shouldDenyAnonymousUser() {
    mockMvc.perform(get("/api/v1/user"))
        .andExpect(status().isUnauthorized());
}
```

## Best Practices Summary

1. ✅ Use `shouldXxxWhenYyy` naming pattern
2. ✅ Follow Given-When-Then structure
3. ✅ Keep tests focused and isolated
4. ✅ Use builders for test data
5. ✅ Test both happy and error paths
6. ✅ Verify mock interactions
7. ✅ Use AssertJ for assertions
8. ✅ Maintain 80%+ code coverage
9. ✅ Write fast, independent tests
10. ❌ NO shared mutable state
11. ❌ NO testing framework code
12. ❌ NO ignoring failing tests

## Related Documentation
- See [unit-tests.md](./unit-tests.md) for unit testing details
- See [integration-tests.md](./integration-tests.md) for integration testing
