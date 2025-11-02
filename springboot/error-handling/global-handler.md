# Global Exception Handler

## Overview
Centralized exception handling using `@ControllerAdvice` provides consistent error responses across the application.

## GlobalExceptionHandler Structure

```java
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

    // Handle custom exceptions
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException e) {
        log.error("Resource not found: {}", e.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            e.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(ResourceAlreadyExistsException.class)
    public ResponseEntity<ErrorResponse> handleAlreadyExists(ResourceAlreadyExistsException e) {
        log.error("Resource already exists: {}", e.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.CONFLICT.value(),
            e.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
    }

    @ExceptionHandler(InvalidOperationException.class)
    public ResponseEntity<ErrorResponse> handleInvalidOperation(InvalidOperationException e) {
        log.error("Invalid operation: {}", e.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            e.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.badRequest().body(error);
    }

    @ExceptionHandler(UnauthorizedException.class)
    public ResponseEntity<ErrorResponse> handleUnauthorized(UnauthorizedException e) {
        log.error("Unauthorized: {}", e.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.UNAUTHORIZED.value(),
            e.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(error);
    }

    @ExceptionHandler(ForbiddenException.class)
    public ResponseEntity<ErrorResponse> handleForbidden(ForbiddenException e) {
        log.error("Forbidden: {}", e.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.FORBIDDEN.value(),
            e.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }

    // Handle validation errors
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidationErrors(MethodArgumentNotValidException e) {
        List<String> errors = e.getBindingResult()
            .getFieldErrors()
            .stream()
            .map(error -> error.getField() + ": " + error.getDefaultMessage())
            .toList();

        log.error("Validation failed: {}", errors);

        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Validation failed: " + String.join(", ", errors),
            LocalDateTime.now()
        );
        return ResponseEntity.badRequest().body(error);
    }

    // Handle Spring Security access denied
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ErrorResponse> handleAccessDenied(AccessDeniedException e) {
        log.error("Access denied: {}", e.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.FORBIDDEN.value(),
            "You don't have permission to access this resource",
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(error);
    }

    // Handle method argument type mismatch
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ErrorResponse> handleTypeMismatch(MethodArgumentTypeMismatchException e) {
        log.error("Type mismatch: {}", e.getMessage());
        String message = String.format("Invalid value '%s' for parameter '%s'",
            e.getValue(), e.getName());

        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            message,
            LocalDateTime.now()
        );
        return ResponseEntity.badRequest().body(error);
    }

    // Handle HTTP message not readable (malformed JSON)
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResponseEntity<ErrorResponse> handleHttpMessageNotReadable(HttpMessageNotReadableException e) {
        log.error("Malformed JSON request: {}", e.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            "Malformed JSON request",
            LocalDateTime.now()
        );
        return ResponseEntity.badRequest().body(error);
    }

    // Handle missing request parameters
    @ExceptionHandler(MissingServletRequestParameterException.class)
    public ResponseEntity<ErrorResponse> handleMissingParams(MissingServletRequestParameterException e) {
        log.error("Missing parameter: {}", e.getParameterName());
        String message = String.format("Missing required parameter: %s", e.getParameterName());

        ErrorResponse error = new ErrorResponse(
            HttpStatus.BAD_REQUEST.value(),
            message,
            LocalDateTime.now()
        );
        return ResponseEntity.badRequest().body(error);
    }

    // Handle database constraint violations
    @ExceptionHandler(DataIntegrityViolationException.class)
    public ResponseEntity<ErrorResponse> handleDataIntegrityViolation(DataIntegrityViolationException e) {
        log.error("Data integrity violation: {}", e.getMessage());

        String message = "Database constraint violation";
        if (e.getCause() instanceof ConstraintViolationException) {
            message = "Duplicate entry or constraint violation";
        }

        ErrorResponse error = new ErrorResponse(
            HttpStatus.CONFLICT.value(),
            message,
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
    }

    // Generic exception handler (catch-all)
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception e) {
        log.error("Unexpected error: ", e);
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

## ErrorResponse DTO

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private LocalDateTime timestamp;
}
```

### Extended ErrorResponse
```java
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
public class ErrorResponse {
    private int status;
    private String message;
    private String error;
    private String path;
    private LocalDateTime timestamp;
    private Map<String, String> validationErrors;
}
```

## Advanced Error Response Patterns

### Pattern 1: Validation Errors with Field Details
```java
@ExceptionHandler(MethodArgumentNotValidException.class)
public ResponseEntity<ErrorResponse> handleValidationErrors(
        MethodArgumentNotValidException e,
        WebRequest request) {

    Map<String, String> validationErrors = e.getBindingResult()
        .getFieldErrors()
        .stream()
        .collect(Collectors.toMap(
            FieldError::getField,
            error -> error.getDefaultMessage() != null ? error.getDefaultMessage() : "",
            (existing, replacement) -> existing
        ));

    ErrorResponse error = ErrorResponse.builder()
        .status(HttpStatus.BAD_REQUEST.value())
        .error("Validation Failed")
        .message("Invalid input parameters")
        .path(((ServletWebRequest) request).getRequest().getRequestURI())
        .timestamp(LocalDateTime.now())
        .validationErrors(validationErrors)
        .build();

    return ResponseEntity.badRequest().body(error);
}
```

### Pattern 2: Exception with Request Details
```java
@ExceptionHandler(ResourceNotFoundException.class)
public ResponseEntity<ErrorResponse> handleNotFound(
        ResourceNotFoundException e,
        WebRequest request) {

    ErrorResponse error = ErrorResponse.builder()
        .status(HttpStatus.NOT_FOUND.value())
        .error("Not Found")
        .message(e.getMessage())
        .path(((ServletWebRequest) request).getRequest().getRequestURI())
        .timestamp(LocalDateTime.now())
        .build();

    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
}
```

### Pattern 3: Detailed Database Errors (Dev Only)
```java
@ExceptionHandler(DataIntegrityViolationException.class)
public ResponseEntity<ErrorResponse> handleDataIntegrityViolation(
        DataIntegrityViolationException e) {

    String message = "Database constraint violation";

    // Include details only in development
    if (Arrays.asList(environment.getActiveProfiles()).contains("dev")) {
        Throwable rootCause = e.getRootCause();
        if (rootCause != null) {
            message = rootCause.getMessage();
        }
    }

    ErrorResponse error = new ErrorResponse(
        HttpStatus.CONFLICT.value(),
        message,
        LocalDateTime.now()
    );
    return ResponseEntity.status(HttpStatus.CONFLICT).body(error);
}
```

## Controller Integration

### ✅ CORRECT - No try-catch
```java
@DeleteMapping("/{code}")
@PreAuthorize("hasRole(T(com.example.myapp.constants.RolesConstants).USER)")
public ResponseEntity<Void> deleteUser(@PathVariable String code) {
    deleteUserService.deleteUser(code);  // Exceptions handled by @ControllerAdvice
    return ResponseEntity.noContent().build();
}
```

### ❌ INCORRECT - Manual error handling
```java
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

## Service Integration

Services simply throw exceptions:

```java
@Service
@Slf4j
public class DeleteUserService {

    @Transactional
    public void deleteUser(String code) {
        User user = userRepository.findByCode(code)
            .orElseThrow(() -> new ResourceNotFoundException("User", "code", code));

        String username = Utils.getLoggedUsername();
        if (!user.getCreatedBy().equals(username)) {
            throw new ForbiddenException("You don't have permission to delete this user");
        }

        userRepository.delete(user);
        log.info("User deleted: {}", code);
    }
}
```

## Exception Response Examples

### 404 Not Found
```json
{
  "status": 404,
  "message": "User not found with code: 'ABC123'",
  "timestamp": "2025-01-15T10:30:00"
}
```

### 400 Validation Error
```json
{
  "status": 400,
  "message": "Validation failed: name: Name is required, email: Invalid email format",
  "timestamp": "2025-01-15T10:30:00"
}
```

### 409 Conflict
```json
{
  "status": 409,
  "message": "User already exists with code: 'ABC123'",
  "timestamp": "2025-01-15T10:30:00"
}
```

### 403 Forbidden
```json
{
  "status": 403,
  "message": "You don't have permission to delete this user",
  "timestamp": "2025-01-15T10:30:00"
}
```

### Extended Response with Validation Errors
```json
{
  "status": 400,
  "error": "Validation Failed",
  "message": "Invalid input parameters",
  "path": "/api/v1/user",
  "timestamp": "2025-01-15T10:30:00",
  "validationErrors": {
    "name": "Name is required",
    "email": "Invalid email format",
    "password": "Password must be at least 8 characters"
  }
}
```

## Testing Exception Handling

### Unit Test
```java
@WebMvcTest(GetUserController.class)
class GetUserControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private GetUserService getUserService;

    @Test
    void shouldReturn404WhenUserNotFound() throws Exception {
        // Given
        String code = "NOTFOUND";
        when(getUserService.getUser(code))
            .thenThrow(new ResourceNotFoundException("User", "code", code));

        // When & Then
        mockMvc.perform(get("/api/v1/user/{code}", code))
            .andExpect(status().isNotFound())
            .andExpect(jsonPath("$.status").value(404))
            .andExpect(jsonPath("$.message").value(containsString("not found")));
    }

    @Test
    void shouldReturn400WhenValidationFails() throws Exception {
        // Given
        CreateUserRequest request = new CreateUserRequest("", "");

        // When & Then
        mockMvc.perform(post("/api/v1/user")
                .contentType(MediaType.APPLICATION_JSON)
                .content(objectMapper.writeValueAsString(request)))
            .andExpect(status().isBadRequest())
            .andExpect(jsonPath("$.status").value(400))
            .andExpect(jsonPath("$.message").value(containsString("Validation failed")));
    }
}
```

## Best Practices

1. ✅ Use `@ControllerAdvice` for centralized exception handling
2. ✅ Handle specific exceptions before generic ones
3. ✅ Log errors with appropriate levels
4. ✅ Return consistent error response structure
5. ✅ Include timestamp in error responses
6. ✅ Use meaningful error messages
7. ✅ Handle validation errors separately
8. ✅ Provide detailed errors in dev, generic in production
9. ✅ Include request path in error response (optional)
10. ❌ NO try-catch in controllers
11. ❌ NO exposing stack traces in production
12. ❌ NO sensitive information in error messages

## Related Documentation
- See [exceptions.md](./exceptions.md) for custom exception definitions
- See [controllers.md](../backend/controllers.md) for controller patterns
- See [services.md](../backend/services.md) for service error handling
- See [dtos.md](../backend/dtos.md) for validation
