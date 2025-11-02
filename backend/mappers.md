# MapStruct and ModelMapper

## Overview
Mappers convert between Entities and DTOs. **NEVER manually map in services** - always use a dedicated Mapper component.

## Choosing Between MapStruct and ModelMapper

### MapStruct (Recommended)
- **When**: Production code requiring performance
- **Pros**: Compile-time mapping, faster, type-safe
- **Cons**: More verbose configuration

### ModelMapper
- **When**: Simple projects or rapid prototyping
- **Pros**: Runtime mapping, less boilerplate
- **Cons**: Slower, errors found at runtime

## MapStruct Implementation

### 1. Add Dependencies
```xml
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct</artifactId>
    <version>1.5.5.Final</version>
</dependency>
<dependency>
    <groupId>org.mapstruct</groupId>
    <artifactId>mapstruct-processor</artifactId>
    <version>1.5.5.Final</version>
    <scope>provided</scope>
</dependency>
```

### 2. Create Mapper Interface
```java
@Mapper(componentModel = "spring")
public interface UserMapper {

    // Entity to DTO
    UserDTO toDTO(User user);

    // Request to Entity (for creation)
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    User toEntity(CreateUserRequest request);

    // Update existing entity from request
    @Mapping(target = "id", ignore = true)
    @Mapping(target = "code", ignore = true)
    @Mapping(target = "createdAt", ignore = true)
    void updateEntity(UpdateUserRequest request, @MappingTarget User user);

    // Lists
    List<UserDTO> toDTOList(List<User> users);

    // Pages
    default Page<UserDTO> toPageDTO(Page<User> page) {
        return page.map(this::toDTO);
    }
}
```

### 3. Use in Services
```java
@Service
@Slf4j
public class CreateUserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper;  // Injected by Spring

    public CreateUserService(UserRepository userRepository,
                              UserMapper userMapper) {
        this.userRepository = userRepository;
        this.userMapper = userMapper;
    }

    @Transactional
    public UserDTO createUser(CreateUserRequest request) {
        User user = userMapper.toEntity(request);  // ✅ Use mapper
        User saved = userRepository.save(user);
        return userMapper.toDTO(saved);  // ✅ Use mapper
    }
}
```

### MapStruct Advanced Features

#### Custom Mappings
```java
@Mapper(componentModel = "spring")
public interface UserMapper {

    @Mapping(source = "userCode", target = "code")
    @Mapping(source = "userName", target = "name")
    @Mapping(target = "createdAt", expression = "java(java.time.LocalDateTime.now())")
    User toEntity(CreateUserRequest request);
}
```

#### Nested Objects
```java
@Mapper(componentModel = "spring", uses = {OrderMapper.class})
public interface UserMapper {

    // Automatically uses OrderMapper for orders field
    UserDTO toDTO(User user);
}
```

#### Conditional Mapping
```java
@Mapper(componentModel = "spring")
public interface UserMapper {

    @Mapping(target = "orders", ignore = true)
    @Condition
    default boolean isNotEmpty(String value) {
        return value != null && !value.trim().isEmpty();
    }
}
```

## ModelMapper Implementation

### 1. Add Dependency
```xml
<dependency>
    <groupId>org.modelmapper</groupId>
    <artifactId>modelmapper</artifactId>
    <version>3.1.1</version>
</dependency>
```

### 2. Configuration
```java
@Configuration
public class ModelMapperConfig {

    @Bean
    public ModelMapper modelMapper() {
        ModelMapper mapper = new ModelMapper();
        mapper.getConfiguration()
            .setMatchingStrategy(MatchingStrategies.STRICT)
            .setSkipNullEnabled(true);
        return mapper;
    }
}
```

### 3. Create Mapper Component
```java
@Component
public class UserMapper {

    private final ModelMapper modelMapper;

    public UserMapper(ModelMapper modelMapper) {
        this.modelMapper = modelMapper;
    }

    public UserDTO toDTO(User user) {
        return modelMapper.map(user, UserDTO.class);
    }

    public User toEntity(CreateUserRequest request) {
        return modelMapper.map(request, User.class);
    }

    public void updateEntity(UpdateUserRequest request, User user) {
        modelMapper.map(request, user);
    }

    public List<UserDTO> toDTOList(List<User> users) {
        return users.stream()
            .map(this::toDTO)
            .collect(Collectors.toList());
    }

    public Page<UserDTO> toPageDTO(Page<User> page) {
        return page.map(this::toDTO);
    }
}
```

### 4. Use in Services
```java
@Service
@Slf4j
public class CreateUserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper;

    public CreateUserService(UserRepository userRepository,
                              UserMapper userMapper) {
        this.userRepository = userRepository;
        this.userMapper = userMapper;
    }

    @Transactional
    public UserDTO createUser(CreateUserRequest request) {
        User user = userMapper.toEntity(request);
        User saved = userRepository.save(user);
        return userMapper.toDTO(saved);
    }
}
```

## Common Mapping Patterns

### Pattern 1: Create
```java
public UserDTO createUser(CreateUserRequest request) {
    User user = userMapper.toEntity(request);
    user.setCode(generateCode());  // Set additional fields
    User saved = userRepository.save(user);
    return userMapper.toDTO(saved);
}
```

### Pattern 2: Read
```java
@Transactional(readOnly = true)
public UserDTO getUser(String code) {
    return userRepository.findByCode(code)
        .map(userMapper::toDTO)
        .orElseThrow(() -> new ResourceNotFoundException("User not found"));
}
```

### Pattern 3: Update
```java
@Transactional
public UserDTO updateUser(String code, UpdateUserRequest request) {
    User user = userRepository.findByCode(code)
        .orElseThrow(() -> new ResourceNotFoundException("User not found"));

    userMapper.updateEntity(request, user);  // Updates existing entity
    User updated = userRepository.save(user);
    return userMapper.toDTO(updated);
}
```

### Pattern 4: List
```java
@Transactional(readOnly = true)
public List<UserDTO> getAllUsers() {
    List<User> users = userRepository.findAll();
    return userMapper.toDTOList(users);
}
```

### Pattern 5: Page
```java
@Transactional(readOnly = true)
public Page<UserDTO> getAllUsers(Pageable pageable) {
    Page<User> page = userRepository.findAll(pageable);
    return userMapper.toPageDTO(page);
}
```

## Directory Structure

Mappers are placed in the `mappers/` directory:
```
mappers/
├── UserMapper.java
├── OrderMapper.java
├── ProductMapper.java
└── UserMapper.java
```

## Common Pitfalls

### ❌ INCORRECT - Manual Mapping in Service
```java
public UserDTO createUser(CreateUserRequest request) {
    User user = new User();
    user.setName(request.getName());
    user.setCode(generateCode());
    // ... more manual mapping
    User saved = userRepository.save(user);

    UserDTO dto = new UserDTO();
    dto.setId(saved.getId());
    dto.setName(saved.getName());
    // ... more manual mapping
    return dto;
}
```

### ✅ CORRECT - Using Mapper
```java
public UserDTO createUser(CreateUserRequest request) {
    User user = userMapper.toEntity(request);
    user.setCode(generateCode());
    User saved = userRepository.save(user);
    return userMapper.toDTO(saved);
}
```

## Best Practices

1. ✅ Use MapStruct for production code
2. ✅ Use ModelMapper for rapid prototyping
3. ✅ Create mapper interfaces/components for each entity
4. ✅ Use `@MappingTarget` for update operations
5. ✅ Create specialized methods for lists and pages
6. ✅ Ignore fields that shouldn't be mapped (`@Mapping(target = "id", ignore = true)`)
7. ✅ Inject mappers via constructor
8. ❌ NO manual mapping in services
9. ❌ NO private mapping methods in services
10. ❌ NO direct entity exposure in controllers

## Related Documentation
- See [entities.md](./entities.md) for entity structure
- See [dtos.md](./dtos.md) for DTO patterns
- See [services.md](./services.md) for mapper usage
- See [dependencies.md](../configuration/dependencies.md) for dependency setup
