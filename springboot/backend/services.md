# Services

## Overview
Services contain business logic and orchestrate data operations. Each service should have **ONE public method** per file.

## Service Structure

### Basic Template
```java
@Service
@Slf4j
public class CreateUserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper;
    private final UserService userService;

    // Constructor injection (preferred over @Autowired)
    public CreateUserService(UserRepository userRepository,
                              UserMapper userMapper,
                              UserService userService) {
        this.userRepository = userRepository;
        this.userMapper = userMapper;
        this.userService = userService;
    }

    // ONE PUBLIC METHOD per service file
    @Transactional
    public UserDTO createUser(CreateUserRequest request) {
        log.info("Creating user with name: {}", request.getName());

        String username = Utils.getLoggedUsername();
        User user = userService.findByUsername(username);

        // Business logic validation
        validateUserCreation(request, user);

        // Map and save
        User user = userMapper.toEntity(request);
        user.setCode(generateUniqueCode());
        User saved = userRepository.save(user);

        log.info("User created successfully with ID: {}", saved.getId());
        return userMapper.toDTO(saved);
    }

    // Private helper methods are allowed
    private void validateUserCreation(CreateUserRequest request, User user) {
        if (userRepository.existsByCodeAndUserId(request.getCode(), user.getId())) {
            throw new ResourceAlreadyExistsException("User already exists");
        }
    }

    private String generateUniqueCode() {
        return RandomCodeGenerator.generate();
    }
}
```

## Directory Structure
Services are organized by resource in lowercase directories:
```
services/
├── user/
│   ├── CreateUserService.java
│   ├── GetUserService.java
│   ├── UpdateUserService.java
│   └── DeleteUserService.java
├── product/
│   ├── CreateProductService.java
│   └── GetProductService.java
└── mqtt/
    ├── ConnectMqttService.java
    └── PublishMqttService.java
```

## Naming Conventions
- **Directory**: `services/<resource>/` (lowercase)
- **File**: `<Action><Resource>Service.java` (e.g., `CreateUserService.java`)
- **Public Method**: `<action><Resource>` (e.g., `createUser()`)

## Transactions

### @Transactional Best Practices
**IMPORTANT**: Use `@Transactional` at the method level, NOT at the class level.

### For READ operations
```java
@Transactional(readOnly = true)
public UserDTO getUser(String code) {
    return userRepository.findByCode(code)
        .map(userMapper::toDTO)
        .orElseThrow(() -> new ResourceNotFoundException("User not found"));
}
```

**Benefits of `readOnly = true`**:
- Performance optimization
- Hibernate skips automatic flush
- Database can optimize the query
- Prevents accidental writes

### For WRITE operations
```java
@Transactional
public UserDTO createUser(CreateUserRequest request) {
    User user = userMapper.toEntity(request);
    User saved = userRepository.save(user);
    return userMapper.toDTO(saved);
}
```

### For operations with specific rollback
```java
@Transactional(rollbackFor = Exception.class)
public void updateUser(String code, UpdateUserRequest request) {
    User user = userRepository.findByCode(code)
        .orElseThrow(() -> new ResourceNotFoundException("User not found"));

    userMapper.updateEntity(request, user);
    userRepository.save(user);
}
```

## Mapper Usage

**IMPORTANT**: DO NOT manually map between entities and DTOs. Use a Mapper component.

### Correct Approach
```java
@Service
@Slf4j
public class CreateUserService {

    private final UserMapper userMapper;  // Injected mapper

    public UserDTO createUser(CreateUserRequest request) {
        User user = userMapper.toEntity(request);  // ✅ Use mapper
        User saved = userRepository.save(user);
        return userMapper.toDTO(saved);  // ✅ Use mapper
    }
}
```

See [mappers.md](./mappers.md) for mapper implementation details.

## Optional Handling

Use `Optional.orElseThrow()` for clean error handling:

```java
public UserDTO getUser(String code) {
    return userRepository.findByCode(code)
        .map(userMapper::toDTO)
        .orElseThrow(() -> new ResourceNotFoundException("User not found with code: " + code));
}
```

## Exception Handling

Services should throw custom exceptions with `@ResponseStatus`:

```java
public void createUser(CreateUserRequest request) {
    if (userRepository.existsByName(request.getName())) {
        throw new ResourceAlreadyExistsException("User already exists");
    }
    // ... creation logic
}
```

See [exceptions.md](../error-handling/exceptions.md) for custom exception details.

## Private Helper Methods

Private methods are allowed and recommended for:
- Business validations: `validateUser()`, `checkPermissions()`
- Auxiliary logic: `generateCode()`, `formatData()`
- Notifications: `sendEmail()`, `notifyUser()`

**DO NOT create private mapping methods** - use Mapper component instead.

## Dependency Injection

### Use Constructor Injection
```java
// ✅ CORRECT - Constructor injection
private final UserRepository userRepository;
private final UserMapper userMapper;

public CreateUserService(UserRepository userRepository,
                          UserMapper userMapper) {
    this.userRepository = userRepository;
    this.userMapper = userMapper;
}
```

```java
// ❌ INCORRECT - Field injection
@Autowired
private UserRepository userRepository;

@Autowired
private UserMapper userMapper;
```

### Use final for Dependencies
Always mark injected dependencies as `final` for immutability.

## Getting Current User

```java
String username = Utils.getLoggedUsername();
User user = userService.findByUsername(username);
```

## Logging

Use SLF4J with Lombok's `@Slf4j`:

```java
@Service
@Slf4j
public class CreateUserService {

    public UserDTO createUser(CreateUserRequest request) {
        log.info("Creating user with name: {}", request.getName());

        try {
            // logic
            log.info("User created successfully with ID: {}", saved.getId());
            return userMapper.toDTO(saved);
        } catch (Exception e) {
            log.error("Error creating user: {}", e.getMessage());
            throw e;
        }
    }
}
```

## Service Best Practices

1. ✅ One public method per service file
2. ✅ Use constructor injection with `final`
3. ✅ Use `@Transactional` at method level
4. ✅ Use `@Transactional(readOnly = true)` for reads
5. ✅ Use Mapper component for entity/DTO conversion
6. ✅ Throw custom exceptions for business errors
7. ✅ Use `Optional.orElseThrow()` correctly
8. ✅ Log at appropriate levels (INFO, ERROR, DEBUG)
9. ✅ Private methods for helper logic only
10. ❌ NO manual mapping in services
11. ❌ NO `@Transactional` at class level
12. ❌ NO field injection with `@Autowired`

## Related Documentation
- See [mappers.md](./mappers.md) for mapping patterns
- See [repositories.md](./repositories.md) for data access
- See [exceptions.md](../error-handling/exceptions.md) for error handling
- See [controllers.md](./controllers.md) for controller integration
