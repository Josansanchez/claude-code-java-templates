# Convenciones de Nombres

## Formato General

**Archivos**: `<Acción><Recurso><Capa>.java` en CamelCase

**Directorios**: Siempre en minúsculas (user, product, order, auth, mqtt)

## Controllers

### Estructura
- **Directorio**: `controllers/<recurso>/` (Ejemplo: `controllers/user/`, `controllers/auth/`)
- **Nombre archivo**: `<Acción><Recurso>Controller.java`
- **Nombre método público**: `<accion><Recurso>` (mismo que la acción del archivo)
- **Endpoints**: `/api/v1/<recurso>` en minúsculas

### Ejemplos
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

### Estructura
- **Directorio**: `services/<recurso>/` (Ejemplo: `services/user/`, `services/auth/`)
- **Nombre archivo**: `<Acción><Recurso>Service.java`
- **Nombre método público**: `<accion><Recurso>`

### Ejemplos
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

### Estructura
- **Directorio**: `repositories/<recurso>/` (Ejemplo: `repositories/user/`)
- **Nombre**: `<Recurso>Repository.java` (Ejemplo: `UserRepository.java`)
- **Un repository por recurso** (puede tener múltiples métodos)

### Ejemplos
```
repositories/user/UserRepository.java
repositories/product/ProductRepository.java
repositories/user/UserRepository.java
```

### Métodos de Repository
```java
// Spring Data JPA derived queries
Optional<User> findByCode(String code);
boolean existsByCode(String code);
List<User> findByNameContaining(String name);

// Con @Query
@Query("SELECT h FROM User h WHERE h.code = :code AND h.active = true")
Optional<User> findActiveByCode(@Param("code") String code);
```

## DTOs

### Estructura
- **Directorio**: `dto/` (sin subdirectorios)
- **Nombre**: `<Recurso>DTO.java` para DTOs estándar
- **Nombre**: `<Acción><Recurso>Request.java` para requests
- **Nombre**: `<Acción><Recurso>Response.java` para responses específicas

### Ejemplos
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

### Estructura
- **Directorio**: `entities/` (sin subdirectorios)
- **Nombre**: Singular en inglés (Ejemplo: `User.java`, `Order.java`, `Product.java`)

### Ejemplos
```
entities/User.java
entities/Order.java
entities/Product.java
entities/User.java
entities/Role.java
entities/Action.java
```

## Mappers

### Estructura
- **Directorio**: `mappers/`
- **Nombre**: `<Recurso>Mapper.java`

### Ejemplos
```
mappers/UserMapper.java
mappers/ProductMapper.java
mappers/UserMapper.java
```

### Métodos de Mapper
```java
UserDTO toDTO(User user);
User toEntity(CreateUserRequest request);
void updateEntity(UpdateUserRequest request, @MappingTarget User user);
List<UserDTO> toDTOList(List<User> users);
Page<UserDTO> toPageDTO(Page<User> page);
```

## Excepciones

### Estructura
- **Directorio**: `exceptions/`
- **Nombre**: `<Tipo>Exception.java`

### Ejemplos
```
exceptions/ResourceNotFoundException.java
exceptions/ResourceAlreadyExistsException.java
exceptions/InvalidOperationException.java
exceptions/UnauthorizedException.java
exceptions/GlobalExceptionHandler.java
```

## Tests

### Estructura
- **Directorio**: `test/java/com/example/myapp/<recurso>/<tipo>/`
- **Nombre clase**: `<NombreClase>Test.java`
- **Nombre método**: `shouldXxxWhenYyy()`

### Ejemplos
```
test/user/controller/CreateUserControllerTest.java
test/user/service/CreateUserServiceTest.java
test/user/repository/UserRepositoryTest.java
test/integration/UserIntegrationTest.java
```

### Métodos de Test
```java
shouldCreateUserWhenValidRequest()
shouldThrowExceptionWhenUserNotFound()
shouldReturnBadRequestWhenInvalidInput()
shouldUpdateUserWhenOwner()
```

## Constantes

### Estructura
```
constants/Constants.java
constants/RolesConstants.java
```

### Uso
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

## Configuración

### Estructura
```
config/OpenApiConfig.java
config/WebSecurityConfig.java
config/ModelMapperConfig.java
```

## Resumen de Patrones

| Tipo | Patrón | Ejemplo |
|------|--------|---------|
| Controller | `<Acción><Recurso>Controller` | `CreateUserController` |
| Service | `<Acción><Recurso>Service` | `CreateUserService` |
| Repository | `<Recurso>Repository` | `UserRepository` |
| Entity | `<Recurso>` | `User` |
| DTO | `<Recurso>DTO` | `UserDTO` |
| Request | `<Acción><Recurso>Request` | `CreateUserRequest` |
| Mapper | `<Recurso>Mapper` | `UserMapper` |
| Exception | `<Tipo>Exception` | `ResourceNotFoundException` |
| Test | `<NombreClase>Test` | `CreateUserServiceTest` |
| Método Test | `should<Xxx>When<Yyy>` | `shouldCreateUserWhenValid` |
| Directorio | lowercase | `user`, `auth`, `mqtt` |

## Regla de Oro

**El nombre del archivo debe reflejar exactamente su único método público**:
- `CreateUserController.java` tiene el método `createUser()`
- `GetUserService.java` tiene el método `getUser()`
- `DeleteProductController.java` tiene el método `deleteProduct()`
