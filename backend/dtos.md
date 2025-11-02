# DTOs and Validations

## Overview
DTOs (Data Transfer Objects) are used to transfer data between layers and to validate user input. They protect internal entity structure from external exposure.

## DTO Structure

### Basic DTO Template
```java
@Data  // Lombok generates getters, setters, toString, equals, hashCode
public class UserDTO {
    private Long id;
    private String name;
    private String code;
    private List<OrderDTO> orders;
}
```

## Lombok for DTOs

**Use `@Data` for DTOs** (without JPA relationships):
```java
@Data
public class UserDTO {
    // fields
}
```

`@Data` generates:
- Getters for all fields
- Setters for all fields
- `toString()`
- `equals()` and `hashCode()`
- Required args constructor

## Request Objects with Validations

**IMPORTANT**: Business validations belong in DTOs and Request objects, NOT in entities.

```java
@Data
public class CreateUserRequest {

    @NotBlank(message = Constants.MANDATORY.NAME)
    @Size(min = 3, max = 20, message = Constants.MANDATORY.SIZE)
    private String name;

    @NotBlank(message = Constants.MANDATORY.PASSWORD)
    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;

    @Email(message = Constants.MANDATORY.EMAILFORMAT)
    @NotBlank(message = Constants.MANDATORY.EMAIL)
    private String email;
}
```

## Validation Annotations

### Common Annotations
- `@NotNull`: Field cannot be null
- `@NotBlank`: String cannot be null or empty (trims whitespace)
- `@NotEmpty`: Collection/Array cannot be empty
- `@Size(min, max)`: String/Collection size constraints
- `@Min(value)`: Numeric minimum value
- `@Max(value)`: Numeric maximum value
- `@Email`: Valid email format
- `@Pattern(regexp)`: Matches regex pattern
- `@Positive`: Number must be positive
- `@PositiveOrZero`: Number must be positive or zero
- `@Past`: Date must be in the past
- `@Future`: Date must be in the future

### Example with Multiple Validations
```java
@Data
public class UpdateUserRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 3, max = 20, message = "Name must be between 3 and 20 characters")
    private String name;

    @NotBlank(message = "Description is required")
    @Size(max = 200, message = "Description cannot exceed 200 characters")
    private String description;

    @Min(value = 1, message = "Orders must be at least 1")
    @Max(value = 50, message = "Orders cannot exceed 50")
    private Integer orderCount;
}
```

## Using Constants for Messages

Define validation messages in constants for reusability:

```java
public final class Constants {
    public final class MANDATORY {
        public static final String NAME = "NameMandatory";
        public static final String EMAIL = "EmailMandatory";
        public static final String EMAILFORMAT = "InvalidEmailFormat";
        public static final String PASSWORD = "PasswordMandatory";
        public static final String SIZE = "InvalidSize";
    }
}
```

Usage:
```java
@Data
public class SignupRequest {
    @NotBlank(message = Constants.MANDATORY.NAME)
    private String name;

    @Email(message = Constants.MANDATORY.EMAILFORMAT)
    @NotBlank(message = Constants.MANDATORY.EMAIL)
    private String email;
}
```

## Directory Structure

DTOs are placed in the `dto/` directory (lowercase):
```
dto/
├── UserDTO.java
├── OrderDTO.java
├── ProductDTO.java
├── UserNavBarDTO.java  // Specific use case
├── CreateUserRequest.java
├── UpdateUserRequest.java
└── ErrorResponse.java
```

## Naming Conventions

### Response DTOs
- `<Resource>DTO.java` (e.g., `UserDTO.java`)
- For specific uses: `<Resource><Purpose>DTO.java` (e.g., `UserNavBarDTO.java`)

### Request Objects
- `Create<Resource>Request.java`
- `Update<Resource>Request.java`
- `<Action><Resource>Request.java`

## DTO Types

### 1. Response DTOs
Used for returning data to clients:
```java
@Data
public class UserDTO {
    private Long id;
    private String name;
    private String code;
    private LocalDateTime createdAt;
}
```

### 2. Request Objects
Used for receiving data from clients with validations:
```java
@Data
public class CreateUserRequest {
    @NotBlank(message = "Name is required")
    private String name;

    @NotBlank(message = "Password is required")
    private String password;
}
```

### 3. Specialized DTOs
For specific use cases:
```java
@Data
public class UserNavBarDTO {
    private String code;
    private String name;
    // Only fields needed for navigation bar
}
```

## Validation in Controllers

Use `@Valid` annotation to trigger validation:

```java
@PostMapping
public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
    // Spring automatically validates the request
    // If validation fails, MethodArgumentNotValidException is thrown
    // GlobalExceptionHandler catches and returns appropriate error response
    return ResponseEntity.created(location).body(createUserService.createUser(request));
}
```

## Error Response DTO

Create a standard error response DTO:

```java
@Data
@AllArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;
}
```

See [global-handler.md](../error-handling/global-handler.md) for usage.

## Separation of Responsibilities

### Entity vs DTO
- **Entity**: Database model, JPA annotations, relationships
- **DTO**: Data transfer, validation, API contract

### Why Use DTOs?
1. **Security**: Don't expose internal entity structure
2. **Flexibility**: API can differ from database model
3. **Validation**: Business rules validated at API boundary
4. **Versioning**: Easy to version API without changing database
5. **Performance**: Send only needed data (no lazy loading issues)

## Best Practices

1. ✅ Use `@Data` for DTOs without relationships
2. ✅ Validate in Request objects, not entities
3. ✅ Use constants for validation messages
4. ✅ Create specific DTOs for different use cases
5. ✅ Use `@Valid` in controllers
6. ✅ Never expose entities directly in APIs
7. ✅ Use meaningful validation messages
8. ❌ NO validations in entities
9. ❌ NO business logic in DTOs
10. ❌ NO JPA annotations in DTOs

## Related Documentation
- See [entities.md](./entities.md) for entity structure
- See [controllers.md](./controllers.md) for validation usage
- See [mappers.md](./mappers.md) for entity/DTO conversion
- See [global-handler.md](../error-handling/global-handler.md) for validation error handling
