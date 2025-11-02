# Spring Profiles Configuration

## Overview
Use YAML files for Spring configuration. Organize by environment: dev, test, prod.

## File Structure
```
src/main/resources/
├── application.yml          # Common configuration
├── application-dev.yml      # Development
├── application-test.yml     # Testing
└── application-prod.yml     # Production
```

## application.yml (Common)

```yaml
spring:
  application:
    name: home-server

  jpa:
    hibernate:
      ddl-auto: validate  # Use Flyway for migrations
      naming:
        implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyJpaCompliantImpl
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
    show-sql: false  # Override per environment
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
        format_sql: true

server:
  port: ${SERVER_PORT:8080}

springdoc:
  api-docs:
    path: /v3/api-docs
  swagger-ui:
    path: /swagger-ui.html
    enabled: true
```

## application-dev.yml (Development)

```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/myapp
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD:password}
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 5
      minimum-idle: 2

  jpa:
    show-sql: true
    hibernate:
      ddl-auto: update  # OK for development only

myapp:
  app:
    jwtSecret: ${JWT_SECRET:dev-secret-key-change-in-production}
    jwtExpirationMs: 86400000  # 24 hours
    jwtCookieName: myapp

  mqtt:
    broker: tcp://localhost:1883
    clientId: myapp-dev
    username: ${messaging_USERNAME:user}
    password: ${messaging_PASSWORD:password}
    topic: "#"

logging:
  level:
    com.example.myapp: DEBUG
    org.springframework.security: DEBUG
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

## application-test.yml (Testing)

```yaml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:

  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

  h2:
    console:
      enabled: false

myapp:
  app:
    jwtSecret: test-secret-key-for-testing-only
    jwtExpirationMs: 3600000  # 1 hour for tests
    jwtCookieName: myapp-test

  mqtt:
    broker: tcp://localhost:1883
    clientId: myapp-test
    username: test
    password: test
    topic: "#"

logging:
  level:
    com.example.myapp: INFO
    org.springframework: WARN
```

## application-test.yml (with Testcontainers)

```yaml
spring:
  datasource:
    url: jdbc:tc:mysql:8.0:///testdb
    driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver

  jpa:
    hibernate:
      ddl-auto: create-drop
    show-sql: true

myapp:
  app:
    jwtSecret: test-secret-key
    jwtExpirationMs: 3600000
    jwtCookieName: myapp-test
```

## application-prod.yml (Production)

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000

  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate  # NEVER update or create-drop in production

  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true

myapp:
  app:
    jwtSecret: ${JWT_SECRET}  # MUST be from environment variable
    jwtExpirationMs: ${JWT_EXPIRATION:86400000}
    jwtCookieName: myapp

  mqtt:
    broker: ${messaging_BROKER}
    clientId: ${messaging_CLIENT_ID:myapp}
    username: ${messaging_USERNAME}
    password: ${messaging_PASSWORD}
    topic: ${messaging_TOPIC:#}

logging:
  level:
    com.example.myapp: INFO
    org.springframework: WARN
    org.springframework.security: INFO
  file:
    name: /var/log/myapp/application.log
    max-size: 10MB
    max-history: 30
```

## Activating Profiles

### In IDE
```
VM Options: -Dspring.profiles.active=dev
```

### In application.properties
```properties
spring.profiles.active=dev
```

### Command Line
```bash
# Maven
mvn spring-boot:run -Dspring-boot.run.profiles=dev

# Java JAR
java -jar -Dspring.profiles.active=prod application.jar

# Environment Variable
export SPRING_PROFILES_ACTIVE=prod
java -jar application.jar
```

### Docker
```dockerfile
ENV SPRING_PROFILES_ACTIVE=prod
```

## Environment Variables

### Development (.env file)
```bash
# Database
DB_USERNAME=root
DB_PASSWORD=password

# JWT
JWT_SECRET=your-dev-secret-key

# messaging
messaging_USERNAME=mqtt_user
messaging_PASSWORD=mqtt_password
```

### Production (System Environment)
```bash
# Required variables
export DB_URL=jdbc:mysql://production-db:3306/myapp
export DB_USERNAME=prod_user
export DB_PASSWORD=secure_password
export JWT_SECRET=very-long-random-secret-key-production

# Optional with defaults
export SERVER_PORT=8080
export JWT_EXPIRATION=86400000
export messaging_BROKER=tcp://mqtt-broker:1883
```

## Configuration Best Practices

### 1. Use Environment Variables for Secrets
```yaml
# ✅ GOOD - From environment
jwtSecret: ${JWT_SECRET}

# ❌ BAD - Hardcoded
jwtSecret: my-secret-key
```

### 2. Provide Defaults for Development
```yaml
# ✅ GOOD - Default for dev, required in prod
username: ${DB_USERNAME:root}

# ✅ GOOD - No default for sensitive data
jwtSecret: ${JWT_SECRET}
```

### 3. Different Settings per Environment
```yaml
# dev
jpa:
  show-sql: true
  hibernate:
    ddl-auto: update

# prod
jpa:
  show-sql: false
  hibernate:
    ddl-auto: validate
```

### 4. Use Flyway in Production
```yaml
# prod only
flyway:
  enabled: true
  locations: classpath:db/migration
```

## Profile-Specific Beans

```java
@Configuration
@Profile("dev")
public class DevConfig {

    @Bean
    public DataSourceInitializer dataSourceInitializer(DataSource dataSource) {
        // Initialize with test data in dev
        return new DataSourceInitializer(dataSource);
    }
}

@Configuration
@Profile("prod")
public class ProdConfig {

    @Bean
    public DataSource dataSource() {
        // Production datasource configuration
        HikariConfig config = new HikariConfig();
        config.setMaximumPoolSize(20);
        // ... production settings
        return new HikariDataSource(config);
    }
}
```

## Conditional Configuration

```java
@Configuration
public class ApplicationConfig {

    @Bean
    @Profile("!prod")
    public SwaggerConfig swaggerConfig() {
        // Enable Swagger only in non-production
        return new SwaggerConfig();
    }

    @Bean
    @Profile("dev | test")
    public DebugLogger debugLogger() {
        // Debug logging only in dev and test
        return new DebugLogger();
    }
}
```

## YAML vs Properties

### Why YAML is Preferred
```yaml
# ✅ YAML - More readable, hierarchical
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/db
    username: user
    password: pass
```

```properties
# ❌ Properties - Repetitive
spring.datasource.url=jdbc:mysql://localhost:3306/db
spring.datasource.username=user
spring.datasource.password=pass
```

## Configuration Validation

### Required Properties
```java
@Configuration
@ConfigurationProperties(prefix = "myapp.app")
@Validated
public class AppProperties {

    @NotBlank
    private String jwtSecret;

    @Positive
    private int jwtExpirationMs;

    @NotBlank
    private String jwtCookieName;

    // Getters and setters
}
```

### Usage
```java
@Service
public class JwtService {

    private final AppProperties appProperties;

    public JwtService(AppProperties appProperties) {
        this.appProperties = appProperties;
    }

    public String generateToken(String username) {
        return Jwts.builder()
            .setSubject(username)
            .setExpiration(new Date(System.currentTimeMillis() +
                appProperties.getJwtExpirationMs()))
            .signWith(SignatureAlgorithm.HS512, appProperties.getJwtSecret())
            .compact();
    }
}
```

## Documentation of Environment Variables

### Create env.example
```bash
# Database Configuration
DB_URL=jdbc:mysql://localhost:3306/myapp
DB_USERNAME=root
DB_PASSWORD=your_password

# JWT Configuration (REQUIRED in production)
JWT_SECRET=your-very-long-secret-key-minimum-256-bits

# Server Configuration (Optional)
SERVER_PORT=8080

# messaging Configuration
messaging_BROKER=tcp://192.168.1.120:1883
messaging_CLIENT_ID=myapp
messaging_USERNAME=mqtt_user
messaging_PASSWORD=mqtt_password
messaging_TOPIC=#
```

## Best Practices Summary

1. ✅ Use YAML over properties
2. ✅ Organize by environment (dev, test, prod)
3. ✅ Use environment variables for secrets
4. ✅ Provide defaults for development only
5. ✅ Never hardcode secrets
6. ✅ Use Flyway in production
7. ✅ Different `ddl-auto` per environment
8. ✅ Document required environment variables
9. ✅ Validate configuration properties
10. ❌ NO secrets in version control
11. ❌ NO `ddl-auto=update` in production
12. ❌ NO default values for secrets in production

## Related Documentation
- See [database.md](./database.md) for JPA configuration
- See [authentication.md](../security/authentication.md) for JWT configuration
- See [dependencies.md](./dependencies.md) for required dependencies
