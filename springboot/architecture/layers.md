# Application Layers

## Layered Architecture

The project follows a standard Spring Boot layered architecture:

```
Controller → Service → Repository → Database
     ↓          ↓
    DTO      Entity
```

## Responsibilities by Layer

### Controllers (`controllers/`)
**Responsibility**: HTTP request handling and REST responses

- Receive HTTP requests
- Validate input with `@Valid`
- Apply security with `@PreAuthorize`
- Call the corresponding Service
- Return `ResponseEntity<T>` with appropriate HTTP code
- **NO business logic**
- **NO exception handling with try-catch**

### Services (`services/`)
**Responsibility**: Business logic

- Implement business rules
- Coordinate operations between multiple repositories
- Manage transactions with `@Transactional`
- Map Entities to DTOs
- Validate complex business rules
- Throw custom exceptions
- **NO direct access to HTTP requests**

### Repositories (`repositories/`)
**Responsibility**: Data access

- Interact with the database
- Extend `JpaRepository<Entity, ID>`
- Define custom queries using Specifications
- **NO business logic**

### Entities (`entities/`)
**Responsibility**: Domain model

- Represent database tables
- Define JPA relationships
- Only database constraints
- **NO business validations**

### DTOs (`dto/`)
**Responsibility**: Data transfer

- Expose data to the client
- Contain input validations
- **NO direct entity exposure**

### Mappers (`mappers/`)
**Responsibility**: Entity ↔ DTO conversion

- Convert entities to DTOs
- Convert requests to entities
- Update entities from requests
- **Reusable across multiple services**

## Data Flow

### Create Resource (POST)
```
1. Client → Controller (CreateUserController)
2. Controller validates with @Valid
3. Controller → Service (CreateUserService)
4. Service validates business rules
5. Service → Mapper (toEntity)
6. Service → Repository (save)
7. Repository → Database
8. Database → Repository (saved entity)
9. Repository → Service
10. Service → Mapper (toDTO)
11. Service → Controller
12. Controller → Client (201 Created + Location header)
```

### Get Resource (GET)
```
1. Client → Controller (GetUserController)
2. Controller → Service (GetUserService)
3. Service → Repository (findByCode)
4. Repository → Database
5. Database → Repository (Optional<Entity>)
6. Repository → Service
7. Service → Mapper (toDTO) or throws exception if empty
8. Service → Controller
9. Controller → Client (200 OK + DTO)
```

### Update Resource (PUT)
```
1. Client → Controller (UpdateUserController)
2. Controller validates with @Valid
3. Controller → Service (UpdateUserService)
4. Service → Repository (findById)
5. Service → Mapper (updateEntity)
6. Service → Repository (save)
7. Repository → Database
8. Service → Mapper (toDTO)
9. Service → Controller
10. Controller → Client (200 OK + updated DTO)
```

### Delete Resource (DELETE)
```
1. Client → Controller (DeleteUserController)
2. Controller → Service (DeleteUserService)
3. Service → Repository (delete)
4. Repository → Database
5. Service → Controller
6. Controller → Client (204 No Content)
```

## SOLID Principles

- **S**ingle Responsibility: Each layer has a unique responsibility
- **O**pen/Closed: Extensible without modifying existing code
- **L**iskov Substitution: Consistent interfaces
- **I**nterface Segregation: Specific interfaces per functionality
- **D**ependency Inversion: Depend on abstractions (interfaces)

## Anti-patterns to Avoid

❌ **Business logic in Controller**
```java
@PostMapping
public ResponseEntity<UserDTO> create(@RequestBody CreateUserRequest request) {
    // ❌ BAD - business logic in controller
    if (userRepository.existsByName(request.getName())) {
        return ResponseEntity.badRequest().build();
    }
    User user = new User();
    user.setName(request.getName());
    userRepository.save(user);
}
```

❌ **Direct Entity exposure**
```java
@GetMapping("/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return ResponseEntity.ok(userRepository.findById(id).get());  // ❌ BAD
}
```

❌ **Business logic in Repository**
```java
public interface UserRepository extends JpaRepository<User, Long> {
    // ❌ BAD - business logic in repository
    default User createUserWithCode(String name) {
        User user = new User();
        user.setName(name);
        user.setCode(generateCode());
        return save(user);
    }
}
```

✅ **Correct separation**
```java
// Controller
@PostMapping
public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
    UserDTO created = createUserService.createUser(request);
    return ResponseEntity.created(location).body(created);
}

// Service
@Transactional
public UserDTO createUser(CreateUserRequest request) {
    if (userRepository.existsByName(request.getName())) {
        throw new ResourceAlreadyExistsException("User already exists");
    }
    User user = userMapper.toEntity(request);
    user.setCode(generateUniqueCode());
    User saved = userRepository.save(user);
    return userMapper.toDTO(saved);
}
```
