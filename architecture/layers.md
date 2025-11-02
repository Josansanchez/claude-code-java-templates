# Capas de la Aplicación

## Arquitectura en Capas

El proyecto sigue una arquitectura en capas estándar de Spring Boot:

```
Controller → Service → Repository → Database
     ↓          ↓
    DTO      Entity
```

## Responsabilidades por Capa

### Controllers (`controllers/`)
**Responsabilidad**: Manejo de peticiones HTTP y respuestas REST

- Recibir peticiones HTTP
- Validar entrada con `@Valid`
- Aplicar seguridad con `@PreAuthorize`
- Llamar al Service correspondiente
- Devolver `ResponseEntity<T>` con código HTTP apropiado
- **NO contener lógica de negocio**
- **NO manejar excepciones con try-catch**

### Services (`services/`)
**Responsabilidad**: Lógica de negocio

- Implementar reglas de negocio
- Coordinar operaciones entre múltiples repositorios
- Gestionar transacciones con `@Transactional`
- Mapear Entities a DTOs
- Validar reglas de negocio complejas
- Lanzar excepciones personalizadas
- **NO acceder directamente a peticiones HTTP**

### Repositories (`repositories/`)
**Responsabilidad**: Acceso a datos

- Interactuar con la base de datos
- Extender `JpaRepository<Entity, ID>`
- Definir consultas personalizadas
- **NO contener lógica de negocio**

### Entities (`entities/`)
**Responsabilidad**: Modelo de dominio

- Representar tablas de base de datos
- Definir relaciones JPA
- Solo constraints de base de datos
- **NO contener validaciones de negocio**

### DTOs (`dto/`)
**Responsabilidad**: Transferencia de datos

- Exponer datos al cliente
- Contener validaciones de entrada
- **NO exponer entities directamente**

### Mappers (`mappers/`)
**Responsabilidad**: Conversión Entity ↔ DTO

- Convertir entities a DTOs
- Convertir requests a entities
- Actualizar entities desde requests
- **Reutilizables en múltiples services**

## Flujo de Datos

### Crear Recurso (POST)
```
1. Client → Controller (CreateUserController)
2. Controller valida con @Valid
3. Controller → Service (CreateUserService)
4. Service valida reglas de negocio
5. Service → Mapper (toEntity)
6. Service → Repository (save)
7. Repository → Database
8. Database → Repository (entity guardada)
9. Repository → Service
10. Service → Mapper (toDTO)
11. Service → Controller
12. Controller → Client (201 Created + Location header)
```

### Obtener Recurso (GET)
```
1. Client → Controller (GetUserController)
2. Controller → Service (GetUserService)
3. Service → Repository (findByCode)
4. Repository → Database
5. Database → Repository (Optional<Entity>)
6. Repository → Service
7. Service → Mapper (toDTO) o lanza excepción si vacío
8. Service → Controller
9. Controller → Client (200 OK + DTO)
```

### Actualizar Recurso (PUT)
```
1. Client → Controller (UpdateUserController)
2. Controller valida con @Valid
3. Controller → Service (UpdateUserService)
4. Service → Repository (findById)
5. Service → Mapper (updateEntity)
6. Service → Repository (save)
7. Repository → Database
8. Service → Mapper (toDTO)
9. Service → Controller
10. Controller → Client (200 OK + DTO actualizado)
```

### Eliminar Recurso (DELETE)
```
1. Client → Controller (DeleteUserController)
2. Controller → Service (DeleteUserService)
3. Service → Repository (delete)
4. Repository → Database
5. Service → Controller
6. Controller → Client (204 No Content)
```

## Principios SOLID

- **S**ingle Responsibility: Cada capa tiene una responsabilidad única
- **O**pen/Closed: Extensible sin modificar código existente
- **L**iskov Substitution: Interfaces consistentes
- **I**nterface Segregation: Interfaces específicas por funcionalidad
- **D**ependency Inversion: Depender de abstracciones (interfaces)

## Anti-patrones a Evitar

❌ **Lógica de negocio en Controller**
```java
@PostMapping
public ResponseEntity<UserDTO> create(@RequestBody CreateUserRequest request) {
    // ❌ MAL - lógica de negocio en controller
    if (userRepository.existsByName(request.getName())) {
        return ResponseEntity.badRequest().build();
    }
    User user = new User();
    user.setName(request.getName());
    userRepository.save(user);
}
```

❌ **Exponer Entity directamente**
```java
@GetMapping("/{id}")
public ResponseEntity<User> getUser(@PathVariable Long id) {
    return ResponseEntity.ok(userRepository.findById(id).get());  // ❌ MAL
}
```

❌ **Lógica de negocio en Repository**
```java
public interface UserRepository extends JpaRepository<User, Long> {
    // ❌ MAL - lógica de negocio en repository
    default User createUserWithCode(String name) {
        User user = new User();
        user.setName(name);
        user.setCode(generateCode());
        return save(user);
    }
}
```

✅ **Separación correcta**
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
