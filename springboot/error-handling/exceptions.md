# Custom Exceptions

## Overview
Custom exceptions provide meaningful error responses and proper HTTP status codes. **NEVER use generic `RuntimeException`**.

## Exception Structure

### Basic Custom Exception
```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }

    public ResourceNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

## Standard Custom Exceptions

### 1. ResourceNotFoundException (404)
```java
@ResponseStatus(HttpStatus.NOT_FOUND)
public class ResourceNotFoundException extends RuntimeException {

    public ResourceNotFoundException(String message) {
        super(message);
    }

    public ResourceNotFoundException(String resourceName, String fieldName, Object fieldValue) {
        super(String.format("%s not found with %s: '%s'", resourceName, fieldName, fieldValue));
    }
}
```

**Usage**:
```java
public UserDTO getUser(String code) {
    return userRepository.findByCode(code)
        .map(userMapper::toDTO)
        .orElseThrow(() -> new ResourceNotFoundException("User", "code", code));
}
```

### 2. ResourceAlreadyExistsException (409)
```java
@ResponseStatus(HttpStatus.CONFLICT)
public class ResourceAlreadyExistsException extends RuntimeException {

    public ResourceAlreadyExistsException(String message) {
        super(message);
    }

    public ResourceAlreadyExistsException(String resourceName, String fieldName, Object fieldValue) {
        super(String.format("%s already exists with %s: '%s'", resourceName, fieldName, fieldValue));
    }
}
```

**Usage**:
```java
public void createUser(CreateUserRequest request) {
    if (userRepository.existsByCode(request.getCode())) {
        throw new ResourceAlreadyExistsException("User", "code", request.getCode());
    }
    // ... creation logic
}
```

### 3. InvalidOperationException (400)
```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class InvalidOperationException extends RuntimeException {

    public InvalidOperationException(String message) {
        super(message);
    }

    public InvalidOperationException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

**Usage**:
```java
public void transferUser(String code, String newOwner) {
    if (code.equals(newOwner)) {
        throw new InvalidOperationException("Cannot transfer user to the same user");
    }
    // ... transfer logic
}
```

### 4. UnauthorizedException (401)
```java
@ResponseStatus(HttpStatus.UNAUTHORIZED)
public class UnauthorizedException extends RuntimeException {

    public UnauthorizedException(String message) {
        super(message);
    }
}
```

**Usage**:
```java
public String getLoggedUsername() {
    Authentication auth = SecurityContextHolder.getContext().getAuthentication();
    if (auth == null || !auth.isAuthenticated()) {
        throw new UnauthorizedException("User not authenticated");
    }
    return auth.getName();
}
```

### 5. ForbiddenException (403)
```java
@ResponseStatus(HttpStatus.FORBIDDEN)
public class ForbiddenException extends RuntimeException {

    public ForbiddenException(String message) {
        super(message);
    }
}
```

**Usage**:
```java
public void deleteUser(String code) {
    String username = Utils.getLoggedUsername();
    User user = findUserByCode(code);

    if (!user.getCreatedBy().equals(username)) {
        throw new ForbiddenException("You don't have permission to delete this user");
    }
    // ... deletion logic
}
```

### 6. BadRequestException (400)
```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class BadRequestException extends RuntimeException {

    public BadRequestException(String message) {
        super(message);
    }

    public BadRequestException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

**Usage**:
```java
public void updateUser(UpdateUserRequest request) {
    if (request.getOrderCount() < 0) {
        throw new BadRequestException("Order count cannot be negative");
    }
    // ... update logic
}
```

## Directory Structure

```
exceptions/
├── ResourceNotFoundException.java
├── ResourceAlreadyExistsException.java
├── InvalidOperationException.java
├── UnauthorizedException.java
├── ForbiddenException.java
├── BadRequestException.java
└── GlobalExceptionHandler.java
```

## Exception Hierarchy

```
RuntimeException (Spring)
│
├── ResourceNotFoundException (404)
├── ResourceAlreadyExistsException (409)
├── InvalidOperationException (400)
├── BadRequestException (400)
├── UnauthorizedException (401)
└── ForbiddenException (403)
```

## Using Custom Exceptions in Services

### ✅ CORRECT Usage
```java
@Service
@Slf4j
public class GetUserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper;

    @Transactional(readOnly = true)
    public UserDTO getUser(String code) {
        return userRepository.findByCode(code)
            .map(userMapper::toDTO)
            .orElseThrow(() -> new ResourceNotFoundException("User", "code", code));
    }
}
```

### ❌ INCORRECT Usage
```java
@Service
public class GetUserService {

    @Transactional(readOnly = true)
    public UserDTO getUser(String code) {
        User user = userRepository.findByCode(code).orElse(null);
        if (user == null) {
            throw new RuntimeException("notFound");  // ❌ BAD - Generic exception
        }
        return userMapper.toDTO(user);
    }
}
```

## Using Optional with Exceptions

### Pattern 1: orElseThrow
```java
public UserDTO getUser(String code) {
    return userRepository.findByCode(code)
        .map(userMapper::toDTO)
        .orElseThrow(() -> new ResourceNotFoundException("User not found with code: " + code));
}
```

### Pattern 2: Conditional Throw
```java
public void validateUser(String code) {
    if (!userRepository.existsByCode(code)) {
        throw new ResourceNotFoundException("User", "code", code);
    }
}
```

### Pattern 3: Business Logic Validation
```java
public void createUser(CreateUserRequest request) {
    if (userRepository.existsByName(request.getName())) {
        throw new ResourceAlreadyExistsException("User", "name", request.getName());
    }

    if (request.getOrderCount() > 100) {
        throw new InvalidOperationException("User cannot have more than 100 orders");
    }

    // ... creation logic
}
```

## HTTP Status Code Mapping

| Exception | Status | Code | Use Case |
|-----------|--------|------|----------|
| ResourceNotFoundException | NOT_FOUND | 404 | Resource doesn't exist |
| ResourceAlreadyExistsException | CONFLICT | 409 | Duplicate resource |
| InvalidOperationException | BAD_REQUEST | 400 | Invalid business logic |
| BadRequestException | BAD_REQUEST | 400 | Invalid input |
| UnauthorizedException | UNAUTHORIZED | 401 | Not authenticated |
| ForbiddenException | FORBIDDEN | 403 | Not authorized |

## Exception Best Practices

1. ✅ Create specific exceptions for different error types
2. ✅ Use `@ResponseStatus` to set HTTP status code
3. ✅ Extend `RuntimeException` (unchecked exceptions)
4. ✅ Provide meaningful error messages
5. ✅ Use constructor overloading for flexibility
6. ✅ Throw exceptions early (fail fast)
7. ✅ Use `Optional.orElseThrow()` for cleaner code
8. ❌ NO generic `RuntimeException`
9. ❌ NO catching exceptions in controllers
10. ❌ NO business logic in exception constructors

## Advanced Exception Patterns

### Exception with Additional Data
```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class ValidationException extends RuntimeException {

    private final Map<String, String> errors;

    public ValidationException(String message, Map<String, String> errors) {
        super(message);
        this.errors = errors;
    }

    public Map<String, String> getErrors() {
        return errors;
    }
}
```

### Exception with Error Code
```java
@ResponseStatus(HttpStatus.BAD_REQUEST)
public class BusinessException extends RuntimeException {

    private final String errorCode;

    public BusinessException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }

    public String getErrorCode() {
        return errorCode;
    }
}
```

**Usage**:
```java
throw new BusinessException("HOUSE_001", "User name cannot contain special characters");
```

## Testing Exceptions

### Unit Test
```java
@ExtendWith(MockitoExtension.class)
class GetUserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private GetUserService getUserService;

    @Test
    void shouldThrowExceptionWhenUserNotFound() {
        // Given
        String code = "NONEXISTENT";
        when(userRepository.findByCode(code)).thenReturn(Optional.empty());

        // When & Then
        assertThatThrownBy(() -> getUserService.getUser(code))
            .isInstanceOf(ResourceNotFoundException.class)
            .hasMessageContaining("User not found");
    }
}
```

### Integration Test
```java
@SpringBootTest
class UserExceptionIntegrationTest {

    @Autowired
    private GetUserService getUserService;

    @Test
    void shouldThrowNotFoundExceptionForInvalidCode() {
        assertThatThrownBy(() -> getUserService.getUser("INVALID"))
            .isInstanceOf(ResourceNotFoundException.class);
    }
}
```

## Related Documentation
- See [global-handler.md](./global-handler.md) for exception handling
- See [services.md](../backend/services.md) for service patterns
- See [controllers.md](../backend/controllers.md) for controller error handling
