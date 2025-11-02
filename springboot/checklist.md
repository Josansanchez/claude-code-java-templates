# Checklist for New Features

## Complete checklist for implementing new features in the Spring Boot Application project.

## Directory Structure

- [ ] Create directory for resource in `controllers/<resource>/` (lowercase)
- [ ] Create directory for resource in `services/<resource>/` (lowercase)
- [ ] Create directory for resource in `repositories/<resource>/` (lowercase)

## Entities

- [ ] Entity with Lombok `@Getter`, `@Setter`, `@Builder`, `@NoArgsConstructor`, `@AllArgsConstructor` in `entities/`
- [ ] `@ToString(exclude = {"relationships"})` to avoid lazy loading
- [ ] `@EqualsAndHashCode(of = "id")` only by ID
- [ ] **NO** implement `Serializable` (unless necessary)
- [ ] Only database constraints in entity, NO business validations

## DTOs and Request Objects

- [ ] DTOs with Lombok `@Data` in `dto/`
- [ ] Request objects with validations `@NotBlank`, `@Size`, `@Email`, etc.
- [ ] Use constants for validation messages

## Mappers

- [ ] Create Mapper interface with MapStruct: `@Mapper(componentModel = "spring")`
- [ ] Methods: `toDTO()`, `toEntity()`, `updateEntity()`, `toPageDTO()`
- [ ] **NO** create private mapping methods in services

## Repositories

- [ ] Repository in `repositories/<resource>/` extending `JpaRepository`
- [ ] Derived methods from Spring Data JPA when possible
- [ ] `@Query` for complex queries

## Services

- [ ] Service(s) in `services/<resource>/` with **one public method each**
- [ ] Annotate with `@Service` and `@Slf4j`
- [ ] `@Transactional` only on methods that modify data
- [ ] `@Transactional(readOnly = true)` for read operations
- [ ] Constructor injection with `final` for dependencies
- [ ] Use Mapper component for conversions
- [ ] Throw custom exceptions (`ResourceNotFoundException`, etc.)
- [ ] Use `Optional.orElseThrow()` correctly
- [ ] Logging with `log.info()`, `log.error()`

## Controllers

- [ ] Controller(s) in `controllers/<resource>/` with **one public method each**
- [ ] File name = Public method name + Layer (Example: `CreateUserController.java` → `createUser()`)
- [ ] Routes with version: `/api/v1/<resource>`
- [ ] **NO** use `@CrossOrigin` (configure globally)
- [ ] `@PreAuthorize` on protected endpoints
- [ ] `@Valid` for validating request bodies
- [ ] **NO** use try-catch (exceptions handled by `@ControllerAdvice`)
- [ ] Return `ResponseEntity<T>` with appropriate HTTP code:
  - `201 Created` for POST (with `Location` header)
  - `200 OK` for GET, PUT
  - `204 No Content` for DELETE
- [ ] Document with `@Operation` and `@ApiResponses`
- [ ] Pagination with `Pageable` for GET ALL

## Exceptions

- [ ] Create custom exceptions with `@ResponseStatus`
- [ ] Handler in `GlobalExceptionHandler` with `@ControllerAdvice`
- [ ] `ErrorResponse` DTO for consistent error responses

## Testing

- [ ] Unit tests for service with Mockito (`@ExtendWith(MockitoExtension.class)`)
- [ ] Unit tests for controller with `@WebMvcTest`
- [ ] Tests for repository with `@DataJpaTest`
- [ ] Integration tests with `@SpringBootTest` (only when necessary)
- [ ] Naming: `shouldXxxWhenYyy()`
- [ ] Given-When-Then structure
- [ ] AssertJ for assertions
- [ ] Minimum 80% coverage

## Configuration

- [ ] Environment variables for secrets in production
- [ ] Configuration per profile in YAML (dev, test, prod)
- [ ] Document required environment variables

## Documentation

- [ ] Swagger UI accessible and working
- [ ] All endpoints documented with OpenAPI

## General

- [ ] Constructor injection, NO `@Autowired` on fields
- [ ] Injected fields as `final`
- [ ] Private methods only for auxiliary logic (NO mapping)
- [ ] Appropriate logs at key points
- [ ] Code in English, comments in Spanish

---

## Example: Complete CRUD for User

### Directory Structure
```
controllers/user/
├── CreateUserController.java    → createUser()
├── GetUserController.java       → getUser(code)
├── GetAllUsersController.java   → getAllUsers()
├── UpdateUserController.java    → updateUser(dto)
└── DeleteUserController.java    → deleteUser(code)

services/user/
├── CreateUserService.java       → createUser()
├── GetUserService.java          → getUser(code)
├── GetAllUsersService.java      → getAllUsers()
├── UpdateUserService.java       → updateUser(dto)
└── DeleteUserService.java       → deleteUser(code)

repositories/user/
└── UserRepository.java          → (multiple methods allowed)

entities/
└── User.java

dto/
├── UserDTO.java
├── CreateUserRequest.java
└── UpdateUserRequest.java

mappers/
└── UserMapper.java
```

**Note**: All directories in lowercase (user, product, auth, etc.)

---

## Quick Validation Before PR

### Code Quality
- [ ] No hardcoded secrets
- [ ] No `@CrossOrigin` on controllers
- [ ] No try-catch in controllers
- [ ] No manual mapping in services
- [ ] No `@Transactional` at class level
- [ ] No `@Data` on entities with relationships

### Testing
- [ ] All tests pass
- [ ] Coverage above 80%
- [ ] No skipped/ignored tests

### Documentation
- [ ] All endpoints documented in Swagger
- [ ] README updated if necessary
- [ ] CHANGELOG updated

### Security
- [ ] `@PreAuthorize` on protected endpoints
- [ ] Secrets in environment variables
- [ ] No sensitive data in logs

### Database
- [ ] Flyway migration created (if schema changes)
- [ ] No `ddl-auto=update` in production
- [ ] Indexes added for queried columns

---

## Related Documentation
- See [best-practices.md](./best-practices.md) for complete best practices list
- See [setup.md](./setup.md) for detailed development rules
- See individual topic files in `backend/`, `security/`, `testing/`, etc.
