# Logging

## Overview
Use SLF4J with Lombok's `@Slf4j` annotation for logging throughout the application.

## Lombok @Slf4j

### Enable Logging in Classes
```java
@Service
@Slf4j
public class CreateUserService {

    public UserDTO createUser(CreateUserRequest request) {
        log.info("Creating user with name: {}", request.getName());

        try {
            // Business logic
            User saved = userRepository.save(user);
            log.info("User created successfully with ID: {}", saved.getId());
            return userMapper.toDTO(saved);
        } catch (Exception e) {
            log.error("Error creating user: {}", e.getMessage(), e);
            throw e;
        }
    }
}
```

## Log Levels

### INFO - General Information
```java
log.info("Creating user with name: {}", request.getName());
log.info("User created successfully with ID: {}", saved.getId());
log.info("User {} logged in successfully", username);
```

### DEBUG - Debugging Information
```java
log.debug("Validating user creation request: {}", request);
log.debug("Retrieved {} users from database", users.size());
log.debug("Mapping entity to DTO: {}", entity);
```

### ERROR - Errors and Exceptions
```java
log.error("Error creating user: {}", e.getMessage());
log.error("Failed to connect to messaging broker: {}", e.getMessage(), e);
log.error("Database constraint violation: {}", e.getMessage());
```

### WARN - Warnings
```java
log.warn("User code already exists: {}", code);
log.warn("Connection pool near capacity: {}/{}", active, max);
log.warn("Slow query detected: {} ms", duration);
```

### TRACE - Very Detailed Information
```java
log.trace("Entering method createUser with request: {}", request);
log.trace("Query parameters: {}", params);
```

## Configuration

### application.yml (Development)
```yaml
logging:
  level:
    root: INFO
    com.example.myapp: DEBUG
    org.springframework.web: DEBUG
    org.springframework.security: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE

  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
```

### application-prod.yml (Production)
```yaml
logging:
  level:
    root: WARN
    com.example.myapp: INFO
    org.springframework: WARN
    org.springframework.security: INFO

  file:
    name: /var/log/myapp/application.log
    max-size: 10MB
    max-history: 30
    total-size-cap: 1GB

  pattern:
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"

  logback:
    rollingpolicy:
      max-file-size: 10MB
      max-history: 30
```

## Logback Configuration

### logback-spring.xml (Advanced)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- Console Appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- File Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>/var/log/myapp/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>/var/log/myapp/application-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Error File Appender -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>/var/log/myapp/error.log</file>
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>ERROR</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>/var/log/myapp/error-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>90</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- Loggers -->
    <logger name="com.example.myapp" level="INFO"/>
    <logger name="org.springframework.web" level="INFO"/>
    <logger name="org.hibernate.SQL" level="DEBUG"/>

    <springProfile name="dev">
        <logger name="com.example.myapp" level="DEBUG"/>
        <logger name="org.springframework.security" level="DEBUG"/>
    </springProfile>

    <springProfile name="prod">
        <logger name="com.example.myapp" level="INFO"/>
        <logger name="org.springframework" level="WARN"/>
    </springProfile>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="ERROR_FILE"/>
    </root>
</configuration>
```

## Logging Patterns

### Service Logging
```java
@Service
@Slf4j
public class CreateUserService {

    @Transactional
    public UserDTO createUser(CreateUserRequest request) {
        log.info("Creating user with name: {}", request.getName());

        String username = Utils.getLoggedUsername();
        log.debug("User {} is creating a user", username);

        // Validation
        if (userRepository.existsByName(request.getName())) {
            log.warn("User name already exists: {}", request.getName());
            throw new ResourceAlreadyExistsException("User already exists");
        }

        // Save
        User user = userMapper.toEntity(request);
        user.setCode(generateCode());
        User saved = userRepository.save(user);

        log.info("User created successfully with ID: {} and code: {}",
            saved.getId(), saved.getCode());

        return userMapper.toDTO(saved);
    }
}
```

### Controller Logging
```java
@RestController
@RequestMapping("/api/v1/user")
@Slf4j
public class CreateUserController {

    @PostMapping
    public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
        log.info("Received request to create user: {}", request.getName());

        try {
            UserDTO created = createUserService.createUser(request);
            log.info("User created successfully with code: {}", created.getCode());
            return ResponseEntity.created(location).body(created);
        } catch (Exception e) {
            log.error("Error creating user: {}", e.getMessage());
            throw e;
        }
    }
}
```

### Exception Handler Logging
```java
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler {

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

    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGenericException(Exception e) {
        log.error("Unexpected error occurred", e);  // Include stack trace
        ErrorResponse error = new ErrorResponse(
            HttpStatus.INTERNAL_SERVER_ERROR.value(),
            "An unexpected error occurred",
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
    }
}
```

## Structured Logging

### With MDC (Mapped Diagnostic Context)
```java
@Component
@Slf4j
public class LoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ServletException, IOException {
        try {
            MDC.put("requestId", UUID.randomUUID().toString());
            MDC.put("userId", getCurrentUserId());
            MDC.put("ipAddress", request.getRemoteAddr());

            filterChain.doFilter(request, response);
        } finally {
            MDC.clear();
        }
    }
}
```

### Log Pattern with MDC
```yaml
logging:
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} [%thread] [%X{requestId}] [%X{userId}] %-5level %logger{36} - %msg%n"
```

## Performance Logging

### Execution Time
```java
@Service
@Slf4j
public class UserService {

    public List<UserDTO> getAllUsers() {
        long startTime = System.currentTimeMillis();

        List<UserDTO> users = userRepository.findAll()
            .stream()
            .map(userMapper::toDTO)
            .collect(Collectors.toList());

        long duration = System.currentTimeMillis() - startTime;
        log.info("Retrieved {} users in {} ms", users.size(), duration);

        if (duration > 1000) {
            log.warn("Slow query detected: {} ms", duration);
        }

        return users;
    }
}
```

## Best Practices

### 1. Use Parameterized Logging
```java
// ✅ GOOD - Efficient, no concatenation if log level disabled
log.info("Creating user with name: {}", request.getName());
log.debug("User {} created user {}", username, userCode);

// ❌ BAD - String concatenation always happens
log.info("Creating user with name: " + request.getName());
```

### 2. Appropriate Log Levels
```java
// INFO - Important business events
log.info("User created successfully");

// DEBUG - Detailed debugging information
log.debug("Validating user request: {}", request);

// ERROR - Errors and exceptions
log.error("Failed to create user: {}", e.getMessage(), e);

// WARN - Warnings
log.warn("User name already exists: {}", name);
```

### 3. Include Exceptions in ERROR Logs
```java
// ✅ GOOD - Includes stack trace
log.error("Error creating user: {}", e.getMessage(), e);

// ❌ BAD - Missing stack trace
log.error("Error creating user: {}", e.getMessage());
```

### 4. Avoid Logging Sensitive Data
```java
// ❌ BAD - Logging password
log.info("User login: {} with password: {}", username, password);

// ✅ GOOD - No sensitive data
log.info("User login: {}", username);
```

### 5. Use Guards for Expensive Operations
```java
if (log.isDebugEnabled()) {
    log.debug("Complex object: {}", expensiveToStringOperation());
}
```

## Summary

1. ✅ Use `@Slf4j` from Lombok
2. ✅ Use parameterized logging
3. ✅ Use appropriate log levels
4. ✅ Include stack traces for errors
5. ✅ Log important business events at INFO
6. ✅ Use DEBUG for detailed information
7. ✅ Configure different levels per environment
8. ✅ Use rolling file appenders in production
9. ❌ NO logging sensitive data (passwords, tokens)
10. ❌ NO expensive operations in log statements
11. ❌ NO logging in tight loops
12. ❌ NO String concatenation in log statements

## Related Documentation
- See [services.md](../backend/services.md) for service logging patterns
- See [global-handler.md](../error-handling/global-handler.md) for error logging
- See [profiles.md](../configuration/profiles.md) for environment-specific configuration
