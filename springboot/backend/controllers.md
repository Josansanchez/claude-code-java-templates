# Controllers

## Overview
Controllers handle HTTP requests and responses. Each controller should have **ONE public method** per file.

## Controller Structure

### Basic Template
```java
@RestController
@RequestMapping("/api/v1/user")  // Include API version
@Tag(name = "User", description = "User management API")
public class CreateUserController {

    private final CreateUserService createUserService;

    // Constructor injection (preferred)
    public CreateUserController(CreateUserService createUserService) {
        this.createUserService = createUserService;
    }

    // ONE PUBLIC METHOD per controller
    @PostMapping
    @PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
    @Operation(summary = "Create a new user", description = "Creates a new user for the authenticated user")
    @ApiResponses({
        @ApiResponse(responseCode = "201", description = "User created successfully"),
        @ApiResponse(responseCode = "400", description = "Invalid input data"),
        @ApiResponse(responseCode = "401", description = "Unauthorized")
    })
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
        UserDTO created = createUserService.createUser(request);
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{code}")
            .buildAndExpand(created.getCode())
            .toUri();
        return ResponseEntity.created(location).body(created);
    }
}
```

## Directory Structure
Controllers are organized by resource in lowercase directories:
```
controllers/
├── user/
│   ├── CreateUserController.java
│   ├── GetUserController.java
│   ├── UpdateUserController.java
│   └── DeleteUserController.java
├── product/
│   ├── CreateProductController.java
│   └── GetProductController.java
└── auth/
    ├── LoginController.java
    └── SignupController.java
```

## Naming Conventions
- **Directory**: `controllers/<resource>/` (lowercase)
- **File**: `<Action><Resource>Controller.java`
- **Public Method**: `<action><Resource>`
- **Endpoint**: `/api/v1/<resource>` (lowercase with version)

## HTTP Methods and Status Codes

### CREATE (POST)
```java
@PostMapping
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
@Operation(summary = "Create a new user")
@ApiResponse(responseCode = "201", description = "User created successfully")
public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
    UserDTO created = createUserService.createUser(request);
    URI location = ServletUriComponentsBuilder
        .fromCurrentRequest()
        .path("/{code}")
        .buildAndExpand(created.getCode())
        .toUri();
    return ResponseEntity.created(location).body(created);  // 201 + Location header
}
```

### READ (GET ONE)
```java
@GetMapping("/{code}")
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
@Operation(summary = "Get user by code")
@ApiResponse(responseCode = "200", description = "User found")
@ApiResponse(responseCode = "404", description = "User not found")
public ResponseEntity<UserDTO> getUser(@PathVariable String code) {
    return ResponseEntity.ok(getUserService.getUser(code));
}
```

### READ (GET ALL with Pagination)
```java
@GetMapping
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
@Operation(summary = "Get all users with pagination")
@ApiResponse(responseCode = "200", description = "Users retrieved successfully")
public ResponseEntity<Page<UserDTO>> getAllUsers(
    @PageableDefault(size = 20, sort = "name") Pageable pageable) {
    return ResponseEntity.ok(getAllUsersService.getAllUsers(pageable));
}
```

### UPDATE (PUT)
```java
@PutMapping("/{code}")
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
@Operation(summary = "Update user")
@ApiResponse(responseCode = "200", description = "User updated successfully")
public ResponseEntity<UserDTO> updateUser(
    @PathVariable String code,
    @Valid @RequestBody UpdateUserRequest request) {
    UserDTO updated = updateUserService.updateUser(code, request);
    return ResponseEntity.ok(updated);
}
```

### DELETE
```java
@DeleteMapping("/{code}")
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
@Operation(summary = "Delete user")
@ApiResponse(responseCode = "204", description = "User deleted successfully")
public ResponseEntity<Void> deleteUser(@PathVariable String code) {
    deleteUserService.deleteUser(code);  // Throws exception if not found
    return ResponseEntity.noContent().build();  // 204 No Content
}
```

## HTTP Status Codes
- `200 OK`: GET, PUT successful
- `201 Created`: POST successful (with Location header)
- `204 No Content`: DELETE successful (no body)
- `400 Bad Request`: Validation errors
- `401 Unauthorized`: Not authenticated
- `403 Forbidden`: Not authorized
- `404 Not Found`: Resource not found

## CORS Configuration

**IMPORTANT**: DO NOT use `@CrossOrigin` on controllers. Configure CORS globally in `WebSecurityConfig`.

See [cors.md](../security/cors.md) for details.

## Authorization

Use `@PreAuthorize` for role-based access control:

```java
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).ADMIN)")
```

See [authorization.md](../security/authorization.md) for details.

## Validation

Always use `@Valid` to validate request bodies:

```java
public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
    // Spring automatically validates the request
    // Validation errors are handled by GlobalExceptionHandler
}
```

See [dtos.md](./dtos.md) for validation details.

## Error Handling

**IMPORTANT**: DO NOT use try-catch in controllers. Exceptions are handled by `@ControllerAdvice`.

```java
// ✅ CORRECT - No try-catch, exceptions handled globally
@DeleteMapping("/{code}")
public ResponseEntity<Void> deleteUser(@PathVariable String code) {
    deleteUserService.deleteUser(code);  // Throws exception if fails
    return ResponseEntity.noContent().build();
}
```

```java
// ❌ INCORRECT - Don't do this
@DeleteMapping("/{code}")
public ResponseEntity<MessageResponse> deleteUser(@PathVariable String code) {
    try {
        deleteUserService.deleteUser(code);
        return ResponseEntity.ok().body(new MessageResponse("Deleted"));
    } catch (Exception e) {
        return ResponseEntity.badRequest().body(new MessageResponse(e.getMessage()));
    }
}
```

See [global-handler.md](../error-handling/global-handler.md) for exception handling.

## OpenAPI Documentation

Document all endpoints with OpenAPI annotations:

```java
@Tag(name = "User", description = "User management API")
public class CreateUserController {

    @Operation(summary = "Create a new user", description = "Creates a new user for the authenticated user")
    @ApiResponses({
        @ApiResponse(responseCode = "201", description = "User created successfully"),
        @ApiResponse(responseCode = "400", description = "Invalid input data"),
        @ApiResponse(responseCode = "401", description = "Unauthorized")
    })
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
        // implementation
    }
}
```

See [openapi.md](../documentation/openapi.md) for Swagger configuration.

## API Versioning

Always include version in API paths:

```java
@RequestMapping("/api/v1/user")  // ✅ With version
@RequestMapping("/api/user")      // ❌ Without version
```

## Controller Best Practices

1. ✅ One public method per controller file
2. ✅ Use `ResponseEntity<T>` for all responses
3. ✅ Return appropriate HTTP status codes
4. ✅ Include API version in paths (`/api/v1/...`)
5. ✅ Use `@Valid` for request validation
6. ✅ Use `@PreAuthorize` for authorization
7. ✅ Document with `@Operation` and `@ApiResponses`
8. ✅ Use `Pageable` for GET ALL operations
9. ✅ Constructor injection for services
10. ❌ NO `@CrossOrigin` on controllers
11. ❌ NO try-catch for error handling
12. ❌ NO business logic in controllers

## Related Documentation
- See [services.md](./services.md) for business logic
- See [authorization.md](../security/authorization.md) for access control
- See [cors.md](../security/cors.md) for CORS setup
- See [global-handler.md](../error-handling/global-handler.md) for error handling
- See [openapi.md](../documentation/openapi.md) for API documentation
