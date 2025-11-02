# OpenAPI/Swagger Documentation

## Overview
SpringDoc OpenAPI provides automatic API documentation and Swagger UI for testing endpoints.

## Dependencies

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

## Configuration

### OpenApiConfig.java
```java
@Configuration
public class OpenApiConfig {

    @Bean
    public OpenAPI customOpenAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("Spring Boot Application API")
                .version("1.0")
                .description("API for Application system with external services integration")
                .contact(new Contact()
                    .name("Development Team")
                    .email("dev@myapp.com")
                    .url("https://github.com/myapp"))
                .license(new License()
                    .name("MIT")
                    .url("https://opensource.org/licenses/MIT")))
            .externalDocs(new ExternalDocumentation()
                .description("Full Documentation")
                .url("https://docs.myapp.com"))
            .addSecurityItem(new SecurityRequirement().addList("bearerAuth"))
            .components(new Components()
                .addSecuritySchemes("bearerAuth",
                    new SecurityScheme()
                        .type(SecurityScheme.Type.HTTP)
                        .scheme("bearer")
                        .bearerFormat("JWT")
                        .description("Enter JWT token")));
    }

    @Bean
    public GroupedOpenApi publicApi() {
        return GroupedOpenApi.builder()
            .group("public")
            .pathsToMatch("/api/v1/**")
            .build();
    }

    @Bean
    public GroupedOpenApi adminApi() {
        return GroupedOpenApi.builder()
            .group("admin")
            .pathsToMatch("/api/v1/admin/**")
            .build();
    }
}
```

### application.yml
```yaml
springdoc:
  api-docs:
    path: /v3/api-docs
    enabled: true

  swagger-ui:
    path: /swagger-ui.html
    enabled: true
    operationsSorter: method
    tagsSorter: alpha
    tryItOutEnabled: true
    filter: true

  show-actuator: false
```

### Disable in Production (Optional)
```yaml
# application-prod.yml
springdoc:
  api-docs:
    enabled: false
  swagger-ui:
    enabled: false
```

## Controller Documentation

### Basic Controller Annotations
```java
@RestController
@RequestMapping("/api/v1/user")
@Tag(name = "User", description = "User management operations")
public class CreateUserController {

    private final CreateUserService createUserService;

    public CreateUserController(CreateUserService createUserService) {
        this.createUserService = createUserService;
    }

    @PostMapping
    @PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
    @Operation(
        summary = "Create a new user",
        description = "Creates a new user for the authenticated user with a unique code"
    )
    @ApiResponses({
        @ApiResponse(
            responseCode = "201",
            description = "User created successfully",
            content = @Content(
                mediaType = "application/json",
                schema = @Schema(implementation = UserDTO.class)
            )
        ),
        @ApiResponse(
            responseCode = "400",
            description = "Invalid input data",
            content = @Content(
                mediaType = "application/json",
                schema = @Schema(implementation = ErrorResponse.class)
            )
        ),
        @ApiResponse(
            responseCode = "401",
            description = "Unauthorized - Invalid or missing JWT token"
        ),
        @ApiResponse(
            responseCode = "409",
            description = "User already exists"
        )
    })
    public ResponseEntity<UserDTO> createUser(
        @Valid @RequestBody @io.swagger.v3.oas.annotations.parameters.RequestBody(
            description = "User creation request",
            required = true
        ) CreateUserRequest request
    ) {
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

### GET Endpoint Documentation
```java
@GetMapping("/{code}")
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
@Operation(
    summary = "Get user by code",
    description = "Retrieves a user by its unique code"
)
@ApiResponses({
    @ApiResponse(
        responseCode = "200",
        description = "User found",
        content = @Content(schema = @Schema(implementation = UserDTO.class))
    ),
    @ApiResponse(
        responseCode = "404",
        description = "User not found",
        content = @Content(schema = @Schema(implementation = ErrorResponse.class))
    )
})
public ResponseEntity<UserDTO> getUser(
    @Parameter(description = "User unique code", example = "ABC123")
    @PathVariable String code
) {
    return ResponseEntity.ok(getUserService.getUser(code));
}
```

### Paginated Endpoint Documentation
```java
@GetMapping
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
@Operation(
    summary = "Get all users with pagination",
    description = "Retrieves all users for the authenticated user with pagination support"
)
@ApiResponse(
    responseCode = "200",
    description = "Users retrieved successfully",
    content = @Content(schema = @Schema(implementation = Page.class))
)
public ResponseEntity<Page<UserDTO>> getAllUsers(
    @Parameter(description = "Page number (0-based)")
    @RequestParam(defaultValue = "0") int page,

    @Parameter(description = "Page size")
    @RequestParam(defaultValue = "20") int size,

    @Parameter(description = "Sort by field")
    @RequestParam(defaultValue = "name") String sort
) {
    Pageable pageable = PageRequest.of(page, size, Sort.by(sort));
    return ResponseEntity.ok(getAllUsersService.getAllUsers(pageable));
}
```

## DTO Documentation

### Schema Annotations
```java
@Data
@Schema(description = "User data transfer object")
public class UserDTO {

    @Schema(description = "User unique identifier", example = "1")
    private Long id;

    @Schema(description = "User name", example = "My Home", required = true)
    private String name;

    @Schema(description = "User unique code", example = "ABC123", required = true)
    private String code;

    @Schema(description = "Number of orders", example = "5")
    private Integer orderCount;

    @Schema(description = "Creation timestamp", example = "2025-01-15T10:30:00")
    private LocalDateTime createdAt;

    @Schema(description = "List of orders in the user")
    private List<OrderDTO> orders;
}
```

### Request Object Documentation
```java
@Data
@Schema(description = "Request to create a new user")
public class CreateUserRequest {

    @NotBlank(message = "Name is required")
    @Size(min = 3, max = 20, message = "Name must be between 3 and 20 characters")
    @Schema(
        description = "User name",
        example = "My Home",
        required = true,
        minLength = 3,
        maxLength = 20
    )
    private String name;

    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Schema(
        description = "User password for access control",
        example = "securePassword123",
        required = true,
        minLength = 8,
        format = "password"
    )
    private String password;
}
```

### Error Response Documentation
```java
@Data
@AllArgsConstructor
@Schema(description = "Error response")
public class ErrorResponse {

    @Schema(description = "HTTP status code", example = "404")
    private int status;

    @Schema(description = "Error message", example = "User not found with code: ABC123")
    private String message;

    @Schema(description = "Timestamp of the error", example = "2025-01-15T10:30:00")
    private LocalDateTime timestamp;
}
```

## Security in Swagger UI

### Test with JWT
1. Click "Authorize" button
2. Enter JWT token in format: `your-jwt-token-here`
3. Click "Authorize" and "Close"
4. All requests will include the token

### Security Scheme Configuration
```java
.components(new Components()
    .addSecuritySchemes("bearerAuth",
        new SecurityScheme()
            .type(SecurityScheme.Type.HTTP)
            .scheme("bearer")
            .bearerFormat("JWT")
            .description("Enter JWT token obtained from /api/v1/auth/signin")))
```

## Grouping APIs

### By Tag
```java
@Tag(name = "User", description = "User management operations")
@Tag(name = "Authentication", description = "Authentication endpoints")
@Tag(name = "Order", description = "Order management operations")
```

### By Path
```java
@Bean
public GroupedOpenApi publicApi() {
    return GroupedOpenApi.builder()
        .group("public")
        .pathsToMatch("/api/v1/**")
        .pathsToExclude("/api/v1/admin/**")
        .build();
}

@Bean
public GroupedOpenApi adminApi() {
    return GroupedOpenApi.builder()
        .group("admin")
        .pathsToMatch("/api/v1/admin/**")
        .build();
}
```

## Examples

### Request Body Example
```java
@Operation(
    requestBody = @io.swagger.v3.oas.annotations.parameters.RequestBody(
        description = "User creation request",
        required = true,
        content = @Content(
            examples = @ExampleObject(
                name = "Create user example",
                value = "{\n" +
                       "  \"name\": \"My Home\",\n" +
                       "  \"password\": \"securePassword123\"\n" +
                       "}"
            )
        )
    )
)
public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
    // implementation
}
```

### Response Example
```java
@ApiResponse(
    responseCode = "200",
    description = "User found",
    content = @Content(
        examples = @ExampleObject(
            name = "User response example",
            value = "{\n" +
                   "  \"id\": 1,\n" +
                   "  \"name\": \"My Home\",\n" +
                   "  \"code\": \"ABC123\",\n" +
                   "  \"orderCount\": 5\n" +
                   "}"
        )
    )
)
```

## Accessing Swagger UI

### URLs
- Swagger UI: `http://localhost:8080/swagger-ui.html`
- OpenAPI JSON: `http://localhost:8080/v3/api-docs`
- OpenAPI YAML: `http://localhost:8080/v3/api-docs.yaml`

### Custom Path
```yaml
springdoc:
  swagger-ui:
    path: /docs  # Access at http://localhost:8080/docs
```

## Best Practices

1. ✅ Document all public endpoints
2. ✅ Use `@Operation` for endpoint description
3. ✅ Use `@ApiResponses` for all possible responses
4. ✅ Use `@Schema` for DTO documentation
5. ✅ Include examples in requests and responses
6. ✅ Group related endpoints with `@Tag`
7. ✅ Document path parameters with `@Parameter`
8. ✅ Include security requirements
9. ✅ Provide meaningful descriptions
10. ❌ NO exposing Swagger UI in production (optional)
11. ❌ NO missing response codes
12. ❌ NO undocumented parameters

## Related Documentation
- See [controllers.md](../backend/controllers.md) for controller patterns
- See [dtos.md](../backend/dtos.md) for DTO structure
