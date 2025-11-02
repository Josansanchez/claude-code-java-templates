# Database Configuration

## Overview
The system uses MySQL 8.0 with JPA/Hibernate for ORM and Flyway for migrations.

## JPA Configuration

### application.yml
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/myapp
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD:password}
    driver-class-name: com.mysql.cj.jdbc.Driver
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5
      connection-timeout: 30000
      idle-timeout: 600000
      max-lifetime: 1800000

  jpa:
    hibernate:
      ddl-auto: validate  # Use Flyway for schema management
      naming:
        implicit-strategy: org.hibernate.boot.model.naming.ImplicitNamingStrategyJpaCompliantImpl
        physical-strategy: org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
    show-sql: false
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
        format_sql: true
        use_sql_comments: true
        jdbc:
          batch_size: 20
        order_inserts: true
        order_updates: true
```

## Hibernate ddl-auto Settings

### Development
```yaml
jpa:
  hibernate:
    ddl-auto: update  # Auto-update schema (dev only)
```

### Testing
```yaml
jpa:
  hibernate:
    ddl-auto: create-drop  # Recreate schema for each test
```

### Production
```yaml
jpa:
  hibernate:
    ddl-auto: validate  # Only validate, use Flyway for changes
```

**IMPORTANT**: NEVER use `update` or `create-drop` in production.

## Naming Strategy

### Standard JPA Naming
```java
// Entity field names map directly to column names
@Entity
@Table(name = "user")
public class User {
    @Column(name = "id")  // Column name: id
    private Long id;

    @Column(name = "user_name")  // Column name: user_name
    private String userName;

    @Column(name = "created_at")  // Column name: created_at
    private LocalDateTime createdAt;
}
```

## Flyway Migration

### Setup
```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
    baseline-version: 0
    schemas: myapp
```

### Migration Files
```
src/main/resources/db/migration/
├── V1__initial_schema.sql
├── V2__add_user_roles.sql
├── V3__add_mqtt_products.sql
└── V4__add_user_orders.sql
```

### Migration File Format
```sql
-- V1__initial_schema.sql
CREATE TABLE IF NOT EXISTS users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    email VARCHAR(100) NOT NULL UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS roles (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(20) NOT NULL UNIQUE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

CREATE TABLE IF NOT EXISTS user_roles (
    user_id BIGINT NOT NULL,
    role_id INT NOT NULL,
    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- Insert default roles
INSERT INTO roles (name) VALUES ('ROLE_USER');
INSERT INTO roles (name) VALUES ('ROLE_MODERATOR');
INSERT INTO roles (name) VALUES ('ROLE_ADMIN');
```

### Migration Naming Convention
```
V<VERSION>__<DESCRIPTION>.sql

Examples:
V1__initial_schema.sql
V2__add_user_roles.sql
V3__add_mqtt_products.sql
V4__alter_user_table.sql
V5__add_order_index.sql
```

## HikariCP Connection Pool

### Configuration
```yaml
spring:
  datasource:
    hikari:
      # Maximum number of connections in pool
      maximum-pool-size: 10

      # Minimum number of idle connections
      minimum-idle: 5

      # Maximum time to wait for connection (ms)
      connection-timeout: 30000

      # Maximum time connection can be idle (ms)
      idle-timeout: 600000

      # Maximum lifetime of connection (ms)
      max-lifetime: 1800000

      # Connection test query
      connection-test-query: SELECT 1

      # Pool name
      pool-name: HomeServerHikariPool
```

### Production Settings
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20
      minimum-idle: 10
      connection-timeout: 30000
      idle-timeout: 300000
      max-lifetime: 1200000
      leak-detection-threshold: 60000
```

## MySQL Configuration

### Docker Compose
```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8.0
    container_name: mysql_myapp
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: myapp
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql
    networks:
      - myapp-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  mysql_data:

networks:
  myapp-network:
    driver: bridge
```

### MySQL my.cnf
```ini
[mysqld]
# Character set
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci

# Performance
max_connections=200
max_allowed_packet=256M
innodb_buffer_pool_size=1G

# Logging
slow_query_log=1
slow_query_log_file=/var/log/mysql/slow.log
long_query_time=2

# Binary logging for replication
server-id=1
log_bin=mysql-bin
binlog_format=ROW
expire_logs_days=7
```

## Database Initialization

### Data SQL (Dev/Test Only)
```sql
-- src/main/resources/data.sql (only for dev/test profiles)
-- This runs AFTER Flyway migrations

-- Insert test users
INSERT INTO users (username, password, email) VALUES
('admin', '$2a$10$...', 'admin@myapp.com'),
('testuser', '$2a$10$...', 'test@myapp.com');

-- Assign roles
INSERT INTO user_roles (user_id, role_id) VALUES
(1, 3),  -- admin -> ROLE_ADMIN
(2, 1);  -- testuser -> ROLE_USER
```

### Disable for Production
```yaml
spring:
  sql:
    init:
      mode: never  # Disable data.sql in production
```

## Transaction Management

### Isolation Levels
```java
@Transactional(isolation = Isolation.READ_COMMITTED)
public void updateUser(String code) {
    // Default isolation level for most operations
}

@Transactional(isolation = Isolation.SERIALIZABLE)
public void criticalOperation() {
    // Highest isolation for critical operations
}
```

### Propagation
```java
@Transactional(propagation = Propagation.REQUIRED)
public void normalOperation() {
    // Join existing transaction or create new one
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void independentOperation() {
    // Always create new transaction
}
```

## Query Optimization

### Batch Operations
```yaml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 20
        order_inserts: true
        order_updates: true
```

### N+1 Query Prevention
```java
// ❌ BAD - N+1 queries
@OneToMany(mappedBy = "user")
private List<Order> orders;

// ✅ GOOD - Fetch join
@Query("SELECT h FROM User h LEFT JOIN FETCH h.orders WHERE h.code = :code")
Optional<User> findByCodeWithOrders(@Param("code") String code);

// ✅ GOOD - Entity Graph
@EntityGraph(attributePaths = {"orders", "users"})
Optional<User> findByCode(String code);
```

### Pagination
```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

    Page<User> findByUserId(Long userId, Pageable pageable);
}

// Usage
Pageable pageable = PageRequest.of(0, 20, Sort.by("name").ascending());
Page<User> users = userRepository.findByUserId(userId, pageable);
```

## Database Best Practices

### 1. Use Flyway for Schema Management
```yaml
# ✅ Production
jpa:
  hibernate:
    ddl-auto: validate
flyway:
  enabled: true

# ❌ Production - NEVER
jpa:
  hibernate:
    ddl-auto: update
```

### 2. Optimize Connection Pool
```yaml
hikari:
  maximum-pool-size: 10  # Adjust based on load
  minimum-idle: 5
  connection-timeout: 30000
```

### 3. Use Indexes
```sql
-- Add indexes for frequently queried columns
CREATE INDEX idx_user_code ON user(code);
CREATE INDEX idx_user_username ON users(username);
CREATE INDEX idx_product_user_id ON product(user_id);
```

### 4. Foreign Key Constraints
```sql
-- Ensure data integrity
ALTER TABLE order
ADD CONSTRAINT fk_order_user
FOREIGN KEY (user_id) REFERENCES user(id)
ON DELETE CASCADE;
```

### 5. Use UTF8MB4
```sql
-- Support full Unicode including emojis
CREATE TABLE user (
    id BIGINT PRIMARY KEY,
    name VARCHAR(100)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

## Testing Database

### H2 for Unit Tests
```yaml
# application-test.yml
spring:
  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
  jpa:
    hibernate:
      ddl-auto: create-drop
```

### Testcontainers for Integration Tests
```yaml
# application-test.yml
spring:
  datasource:
    url: jdbc:tc:mysql:8.0:///testdb
    driver-class-name: org.testcontainers.jdbc.ContainerDatabaseDriver
```

## Monitoring

### Enable Statistics
```yaml
spring:
  jpa:
    properties:
      hibernate:
        generate_statistics: true
```

### Metrics
```java
@Configuration
public class DatabaseMetricsConfig {

    @Bean
    public MeterBinder hikariMetrics(DataSource dataSource) {
        return new HikariDataSourcePoolMetrics((HikariDataSource) dataSource,
            "hikaricp", Collections.emptyList());
    }
}
```

## Common Issues and Solutions

### Issue 1: Connection Pool Exhausted
```yaml
# Increase pool size or reduce timeout
hikari:
  maximum-pool-size: 20
  connection-timeout: 30000
```

### Issue 2: Slow Queries
```yaml
# Enable query logging
spring:
  jpa:
    properties:
      hibernate:
        show_sql: true
        format_sql: true
logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

### Issue 3: Schema Mismatch
```bash
# Reset Flyway baseline
mvn flyway:baseline
mvn flyway:migrate
```

## Related Documentation
- See [profiles.md](./profiles.md) for environment configuration
- See [entities.md](../backend/entities.md) for JPA entities
- See [repositories.md](../backend/repositories.md) for data access
- See [dependencies.md](./dependencies.md) for required dependencies
