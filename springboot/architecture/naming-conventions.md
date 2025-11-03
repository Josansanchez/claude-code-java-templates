# Naming Conventions

## General Format

**Files**: `<Action><Resource><Layer>.java` in CamelCase

**Directories**: Always in lowercase (user, product, order, auth, mqtt)

## Controllers

### Structure
- **Directory**: `controllers/<resource>/` (Example: `controllers/user/`, `controllers/auth/`)
- **File name**: `<Action><Resource>Controller.java`
- **Public method name**: `<action><Resource>` (same as file action)
- **Endpoints**: `/api/v1/<resource>` in lowercase

### Examples
```
controllers/user/CreateUserController.java → createUser()
controllers/user/GetUserController.java → getUser(code)
controllers/user/GetAllUsersController.java → getAllUsers(pageable)
controllers/user/UpdateUserController.java → updateUser(code, request)
controllers/user/DeleteUserController.java → deleteUser(code)
controllers/auth/LoginController.java → login(request)
controllers/auth/SignupController.java → signup(request)
```

## Services

### Structure
- **Directory**: `services/<resource>/` (Example: `services/user/`, `services/auth/`)
- **File name**: `<Action><Resource>Service.java`
- **Public method name**: `<action><Resource>`

### Examples
```
services/user/CreateUserService.java → createUser(request)
services/user/GetUserService.java → getUser(code)
services/user/GetAllUsersService.java → getAllUsers(pageable)
services/user/UpdateUserService.java → updateUser(code, request)
services/user/DeleteUserService.java → deleteUser(code)
services/auth/LoginService.java → login(request)
services/mqtt/PublishMqttService.java → publishMqtt(topic, message)
```

## Repositories

### Structure
- **Directory**: `repositories/<resource>/` (Example: `repositories/user/`)
- **Name**: `<Resource>Repository.java` (Example: `UserRepository.java`)
- **One repository per resource** (can have multiple methods)

### Examples
```
repositories/user/UserRepository.java
repositories/product/ProductRepository.java
repositories/order/OrderRepository.java
```

### Repository Methods
```java
// Spring Data JPA derived queries
Optional<User> findByCode(String code);
boolean existsByCode(String code);
List<User> findByNameContaining(String name);

// With Specifications (preferred)
// See UserSpecifications.java for complex queries
```

## DTOs

### Structure
- **Directory**: `dto/` (no subdirectories)
- **Name**: `<Resource>DTO.java` for standard DTOs
- **Name**: `<Action><Resource>Request.java` for requests
- **Name**: `<Action><Resource>Response.java` for specific responses

### Examples
```
dto/UserDTO.java
dto/ProductDTO.java
dto/CreateUserRequest.java
dto/UpdateUserRequest.java
dto/LoginRequest.java
dto/UserInfoResponse.java
dto/ErrorResponse.java
```

## Entities

### Structure
- **Directory**: `entities/` (no subdirectories)
- **Name**: Singular in English (Example: `User.java`, `Order.java`, `Product.java`)

### Examples
```
entities/User.java
entities/Order.java
entities/Product.java
entities/Category.java
entities/Role.java
entities/Action.java
```

## Mappers

### Structure
- **Directory**: `mappers/`
- **Name**: `<Resource>Mapper.java`

### Examples
```
mappers/UserMapper.java
mappers/ProductMapper.java
mappers/OrderMapper.java
```

### Mapper Methods
```java
UserDTO toDTO(User user);
User toEntity(CreateUserRequest request);
void updateEntity(UpdateUserRequest request, @MappingTarget User user);
List<UserDTO> toDTOList(List<User> users);
Page<UserDTO> toPageDTO(Page<User> page);
```

## Specifications

### Structure
- **Directory**: `repositories/<resource>/` (same as repository)
- **Name**: `<Resource>Specifications.java`

### Examples
```
repositories/user/UserRepository.java
repositories/user/UserSpecifications.java
repositories/product/ProductRepository.java
repositories/product/ProductSpecifications.java
```

### Specification Methods
```java
public class UserSpecifications {
    public static Specification<User> hasCode(String code) { ... }
    public static Specification<User> nameContains(String keyword) { ... }
    public static Specification<User> createdAfter(LocalDateTime date) { ... }
}
```

## Exceptions

### Structure
- **Directory**: `exceptions/`
- **Name**: `<Type>Exception.java`

### Examples
```
exceptions/ResourceNotFoundException.java
exceptions/ResourceAlreadyExistsException.java
exceptions/InvalidOperationException.java
exceptions/UnauthorizedException.java
exceptions/GlobalExceptionHandler.java
```

## Tests

### Structure
- **Directory**: `test/java/com/example/myapp/<resource>/<type>/`
- **Class name**: `<ClassName>Test.java`
- **Method name**: `shouldXxxWhenYyy()`

### Examples
```
test/user/controller/CreateUserControllerTest.java
test/user/service/CreateUserServiceTest.java
test/user/repository/UserRepositoryTest.java
test/integration/UserIntegrationTest.java
```

### Test Methods
```java
shouldCreateUserWhenValidRequest()
shouldThrowExceptionWhenUserNotFound()
shouldReturnBadRequestWhenInvalidInput()
shouldUpdateUserWhenOwner()
```

## Constants

### Structure
```
constants/Constants.java
constants/RolesConstants.java
```

### Usage
```java
public final class Constants {
    public final class MANDATORY {
        public static final String NAME = "NameMandatory";
        public static final String EMAIL = "EmailMandatory";
    }
}

public final class RolesConstants {
    public static final String ADMIN = "ROLE_ADMIN";
    public static final String USER = "ROLE_USER";
}
```

## Configuration

### Structure
```
config/OpenApiConfig.java
config/WebSecurityConfig.java
config/ModelMapperConfig.java
```

## Pattern Summary

| Type | Pattern | Example |
|------|---------|---------|
| Controller | `<Action><Resource>Controller` | `CreateUserController` |
| Service | `<Action><Resource>Service` | `CreateUserService` |
| Repository | `<Resource>Repository` | `UserRepository` |
| Specification | `<Resource>Specifications` | `UserSpecifications` |
| Entity | `<Resource>` | `User` |
| DTO | `<Resource>DTO` | `UserDTO` |
| Request | `<Action><Resource>Request` | `CreateUserRequest` |
| Mapper | `<Resource>Mapper` | `UserMapper` |
| Exception | `<Type>Exception` | `ResourceNotFoundException` |
| Test | `<ClassName>Test` | `CreateUserServiceTest` |
| Test Method | `should<Xxx>When<Yyy>` | `shouldCreateUserWhenValid` |
| Directory | lowercase | `user`, `auth`, `mqtt` |

## Golden Rule

**The file name must exactly reflect its single public method**:
- `CreateUserController.java` has the method `createUser()`
- `GetUserService.java` has the method `getUser()`
- `DeleteProductController.java` has the method `deleteProduct()`
