# Database Migrations

## Overview

Database migrations are version-controlled, incremental changes to your database schema. They are **essential** for production applications to manage schema evolution safely and consistently across environments.

## Why Use Migrations?

### ❌ Problems with `ddl-auto`

```yaml
# application.yml - NEVER in production
spring:
  jpa:
    hibernate:
      ddl-auto: update  # ❌ DANGEROUS
```

**Issues**:
- ❌ No version control of schema changes
- ❌ Cannot rollback changes
- ❌ Can cause data loss
- ❌ Different behavior in dev vs prod
- ❌ No audit trail of who changed what
- ❌ Cannot handle data migrations

### ✅ Benefits of Migration Tools

- ✅ **Version controlled** - Track all schema changes in Git
- ✅ **Reproducible** - Same schema in all environments
- ✅ **Rollback support** - Undo changes if needed
- ✅ **Audit trail** - Know who changed what and when
- ✅ **Data migrations** - Safely migrate data during schema changes
- ✅ **Team collaboration** - Multiple developers can work safely
- ✅ **CI/CD integration** - Automated deployments

## Flyway vs Liquibase

| Feature | Flyway | Liquibase |
|---------|--------|-----------|
| **Complexity** | Simple, SQL-based | More complex, multiple formats |
| **Learning Curve** | Low | Medium |
| **Format** | SQL files | SQL, XML, YAML, JSON |
| **Rollback** | Paid feature | Free |
| **Performance** | Fast | Slower (XML parsing) |
| **Community** | Large | Large |
| **Use Case** | Most projects | Complex migrations, need rollbacks |
| **Recommendation** | ✅ **Start here** | Use if you need advanced features |

## Flyway Setup (Recommended)

### 1. Add Dependency

**Maven (`pom.xml`)**:
```xml
<dependencies>
    <!-- Spring Boot Starter Data JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- Flyway Migration -->
    <dependency>
        <groupId>org.flywaydb</groupId>
        <artifactId>flyway-core</artifactId>
    </dependency>

    <!-- MySQL Driver (example) -->
    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

**Gradle (`build.gradle`)**:
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.flywaydb:flyway-core'
    runtimeOnly 'com.mysql:mysql-connector-j'
}
```

### 2. Configure Application Properties

**`application.yml`**:
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/myapp_db
    username: ${DB_USERNAME:root}
    password: ${DB_PASSWORD:password}
    driver-class-name: com.mysql.cj.jdbc.Driver

  jpa:
    hibernate:
      ddl-auto: validate  # ✅ IMPORTANT: validate only, never update/create
    show-sql: false
    properties:
      hibernate:
        format_sql: true

  flyway:
    enabled: true
    baseline-on-migrate: true  # For existing databases
    locations: classpath:db/migration
    schemas: myapp_db
    validate-on-migrate: true
```

**`application-dev.yml`** (Development):
```yaml
spring:
  jpa:
    show-sql: true  # Show SQL in development

  flyway:
    clean-disabled: false  # Allow clean in dev (NEVER in prod)
```

**`application-prod.yml`** (Production):
```yaml
spring:
  jpa:
    show-sql: false

  flyway:
    clean-disabled: true  # ✅ CRITICAL: Prevent accidental data loss
    validate-on-migrate: true
```

### 3. Directory Structure

```
src/main/resources/
└── db/
    └── migration/
        ├── V1__initial_schema.sql
        ├── V2__add_users_table.sql
        ├── V3__add_products_table.sql
        ├── V4__add_orders_table.sql
        ├── V5__add_user_email_index.sql
        └── V6__migrate_user_codes.sql
```

## Flyway Naming Conventions

### Pattern

```
V{VERSION}__{DESCRIPTION}.sql
```

- **V**: Version prefix (required)
- **{VERSION}**: Version number (e.g., 1, 2, 3, or 1.0, 1.1, 2.0)
- **__**: Double underscore separator (required)
- **{DESCRIPTION}**: Snake_case description
- **.sql**: File extension

### Examples

```
✅ CORRECT:
V1__initial_schema.sql
V2__add_users_table.sql
V3__add_products_table.sql
V4__add_email_to_users.sql
V5__create_index_on_user_email.sql
V2.1__hotfix_user_table.sql

❌ INCORRECT:
1_initial_schema.sql              (missing V)
V1_initial_schema.sql             (single underscore)
V1__Initial-Schema.sql            (camelCase/kebab-case)
V1__add users table.sql           (spaces)
initial_schema.sql                (no version)
```

### Versioning Strategy

**Simple Sequential** (Recommended for small teams):
```
V1__initial_schema.sql
V2__add_users_table.sql
V3__add_products_table.sql
```

**Date-based** (Recommended for large teams):
```
V20250103_1000__initial_schema.sql
V20250103_1430__add_users_table.sql
V20250104_0900__add_products_table.sql
```

**Feature-based**:
```
V1.0__initial_schema.sql
V1.1__add_users_module.sql
V2.0__add_products_module.sql
V2.1__hotfix_products.sql
```

## Migration Examples

### Initial Schema

**`V1__initial_schema.sql`**:
```sql
-- Create initial database schema
-- Version: 1.0
-- Author: Your Name
-- Date: 2025-01-03

-- Enable foreign key checks
SET FOREIGN_KEY_CHECKS = 1;

-- Users table
CREATE TABLE users (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_users_username (username),
    INDEX idx_users_email (email),
    INDEX idx_users_active (active)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Roles table
CREATE TABLE roles (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE,
    description VARCHAR(255),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- User roles join table
CREATE TABLE user_roles (
    user_id BIGINT NOT NULL,
    role_id BIGINT NOT NULL,
    assigned_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,

    PRIMARY KEY (user_id, role_id),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE,
    FOREIGN KEY (role_id) REFERENCES roles(id) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Insert default roles
INSERT INTO roles (name, description) VALUES
    ('ROLE_USER', 'Standard user role'),
    ('ROLE_ADMIN', 'Administrator role');
```

### Add New Table

**`V2__add_products_table.sql`**:
```sql
-- Add products table
-- Version: 2.0
-- Author: Your Name
-- Date: 2025-01-04

CREATE TABLE products (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    price DECIMAL(10, 2) NOT NULL,
    stock INT NOT NULL DEFAULT 0,
    active BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    created_by BIGINT,

    INDEX idx_products_code (code),
    INDEX idx_products_active (active),
    INDEX idx_products_price (price),
    FOREIGN KEY (created_by) REFERENCES users(id) ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### Add Column

**`V3__add_phone_to_users.sql`**:
```sql
-- Add phone number to users table
-- Version: 3.0
-- Author: Your Name
-- Date: 2025-01-05

ALTER TABLE users
ADD COLUMN phone_number VARCHAR(20) AFTER email;

-- Add index for phone lookups
CREATE INDEX idx_users_phone ON users(phone_number);
```

### Add Index

**`V4__add_index_user_created_at.sql`**:
```sql
-- Add index on users.created_at for performance
-- Version: 4.0
-- Author: Your Name
-- Date: 2025-01-06

CREATE INDEX idx_users_created_at ON users(created_at DESC);

-- Analyze table to update statistics
ANALYZE TABLE users;
```

### Modify Column

**`V5__modify_user_email_length.sql`**:
```sql
-- Increase email column length
-- Version: 5.0
-- Author: Your Name
-- Date: 2025-01-07

ALTER TABLE users
MODIFY COLUMN email VARCHAR(255) NOT NULL;
```

### Data Migration

**`V6__migrate_inactive_users.sql`**:
```sql
-- Migrate inactive users to archived status
-- Version: 6.0
-- Author: Your Name
-- Date: 2025-01-08

-- Add archived column
ALTER TABLE users
ADD COLUMN archived BOOLEAN NOT NULL DEFAULT FALSE AFTER active;

-- Migrate data: Mark inactive users as archived
UPDATE users
SET archived = TRUE
WHERE active = FALSE
  AND updated_at < DATE_SUB(NOW(), INTERVAL 1 YEAR);

-- Add index
CREATE INDEX idx_users_archived ON users(archived);
```

### Complex Migration with Rollback Comments

**`V7__refactor_user_roles.sql`**:
```sql
-- Refactor user_roles to support role hierarchy
-- Version: 7.0
-- Author: Your Name
-- Date: 2025-01-09
-- Rollback: Run V7__rollback_user_roles.sql

-- Add priority column to roles
ALTER TABLE roles
ADD COLUMN priority INT NOT NULL DEFAULT 0 AFTER description;

-- Add granted_by tracking to user_roles
ALTER TABLE user_roles
ADD COLUMN granted_by BIGINT AFTER role_id,
ADD FOREIGN KEY (granted_by) REFERENCES users(id) ON DELETE SET NULL;

-- Update existing data
UPDATE roles SET priority = 10 WHERE name = 'ROLE_USER';
UPDATE roles SET priority = 100 WHERE name = 'ROLE_ADMIN';

-- Add indexes
CREATE INDEX idx_roles_priority ON roles(priority DESC);
```

### Drop Column (Careful!)

**`V8__remove_deprecated_user_fields.sql`**:
```sql
-- Remove deprecated fields from users table
-- Version: 8.0
-- Author: Your Name
-- Date: 2025-01-10
-- WARNING: This will permanently delete data
-- Rollback: Restore from backup

-- Backup data before dropping (commented for reference)
-- CREATE TABLE users_backup_v8 AS SELECT * FROM users;

-- Drop deprecated columns
ALTER TABLE users
DROP COLUMN phone_number;  -- Moved to separate contacts table
```

## Repeatable Migrations

For migrations that should run on every deployment (views, procedures):

**`R__create_user_summary_view.sql`**:
```sql
-- Repeatable migration: User summary view
-- Runs whenever the file changes

DROP VIEW IF EXISTS user_summary;

CREATE VIEW user_summary AS
SELECT
    u.id,
    u.username,
    u.email,
    u.active,
    COUNT(DISTINCT ur.role_id) as role_count,
    u.created_at
FROM users u
LEFT JOIN user_roles ur ON u.id = ur.user_id
GROUP BY u.id, u.username, u.email, u.active, u.created_at;
```

## Best Practices

### 1. Never Modify Existing Migrations

❌ **WRONG**:
```sql
-- V1__initial_schema.sql (modified after deployment)
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(255) NOT NULL  -- ❌ Changed from VARCHAR(100)
);
```

✅ **CORRECT**:
```sql
-- V1__initial_schema.sql (never change this)
CREATE TABLE users (
    id BIGINT PRIMARY KEY,
    email VARCHAR(100) NOT NULL
);

-- V2__increase_user_email_length.sql (new migration)
ALTER TABLE users MODIFY COLUMN email VARCHAR(255) NOT NULL;
```

### 2. Test Migrations Before Deployment

```bash
# Test migration on local database
./mvnw flyway:migrate -Dflyway.url=jdbc:mysql://localhost:3306/test_db

# Verify migration succeeded
./mvnw flyway:info

# Test rollback (if using Flyway Teams)
./mvnw flyway:undo
```

### 3. Backup Before Migrations

```bash
# Production deployment script
#!/bin/bash

# Backup database
mysqldump -u root -p myapp_db > backup_$(date +%Y%m%d_%H%M%S).sql

# Run migrations
./mvnw flyway:migrate -Dflyway.url=jdbc:mysql://localhost:3306/myapp_db

# Verify
./mvnw flyway:info
```

### 4. Use Transactions

**MySQL**:
```sql
-- V5__transactional_migration.sql
START TRANSACTION;

ALTER TABLE users ADD COLUMN new_field VARCHAR(50);
UPDATE users SET new_field = 'default_value';
ALTER TABLE users MODIFY COLUMN new_field VARCHAR(50) NOT NULL;

COMMIT;
```

**PostgreSQL** (supports transactional DDL):
```sql
-- PostgreSQL automatically wraps in transaction
-- No need for explicit BEGIN/COMMIT
```

### 5. Handle Large Data Migrations

**For large tables, batch updates**:
```sql
-- V6__migrate_large_table.sql

-- Batch update in chunks
SET @batch_size = 1000;
SET @processed = 0;

REPEAT
    UPDATE users
    SET migrated = TRUE
    WHERE migrated IS NULL
    LIMIT @batch_size;

    SET @processed = ROW_COUNT();

    -- Pause to avoid locking
    DO SLEEP(0.1);
UNTIL @processed < @batch_size END REPEAT;
```

### 6. Document Complex Migrations

```sql
-- V10__complex_schema_refactoring.sql
--
-- DESCRIPTION:
--   Refactor user authentication to support multiple providers
--
-- CHANGES:
--   1. Create auth_providers table
--   2. Create user_auth_providers join table
--   3. Migrate existing password authentication
--   4. Add indexes for performance
--
-- ROLLBACK PLAN:
--   Restore from backup_20250110.sql
--
-- ESTIMATED TIME: 5 minutes for 100k users
--
-- AUTHOR: John Doe
-- DATE: 2025-01-10
-- TICKET: JIRA-1234

-- Implementation starts here
CREATE TABLE auth_providers (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(50) NOT NULL UNIQUE,
    enabled BOOLEAN NOT NULL DEFAULT TRUE
);

-- ... rest of migration
```

## Flyway Commands

### Maven

```bash
# Apply migrations
./mvnw flyway:migrate

# Show migration status
./mvnw flyway:info

# Validate migrations
./mvnw flyway:validate

# Clean database (DEV ONLY!)
./mvnw flyway:clean

# Repair metadata table
./mvnw flyway:repair

# Baseline existing database
./mvnw flyway:baseline
```

### Gradle

```bash
./gradlew flywayMigrate
./gradlew flywayInfo
./gradlew flywayValidate
./gradlew flywayClean  # DEV ONLY
```

### Programmatic

```java
@Configuration
public class FlywayConfig {

    @Bean
    public Flyway flyway(DataSource dataSource) {
        return Flyway.configure()
            .dataSource(dataSource)
            .locations("classpath:db/migration")
            .baselineOnMigrate(true)
            .validateOnMigrate(true)
            .load();
    }
}
```

## Liquibase Setup (Alternative)

### 1. Add Dependency

```xml
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>
```

### 2. Configure

**`application.yml`**:
```yaml
spring:
  liquibase:
    enabled: true
    change-log: classpath:db/changelog/db.changelog-master.yaml
```

### 3. Directory Structure

```
src/main/resources/
└── db/
    └── changelog/
        ├── db.changelog-master.yaml
        ├── changes/
        │   ├── v1.0-initial-schema.yaml
        │   ├── v1.1-add-users-table.sql
        │   └── v2.0-add-products-table.yaml
        └── data/
            └── default-data.sql
```

### 4. Master Changelog

**`db.changelog-master.yaml`**:
```yaml
databaseChangeLog:
  - include:
      file: db/changelog/changes/v1.0-initial-schema.yaml
  - include:
      file: db/changelog/changes/v1.1-add-users-table.sql
  - include:
      file: db/changelog/changes/v2.0-add-products-table.yaml
```

### 5. Change Set Example

**`v1.0-initial-schema.yaml`**:
```yaml
databaseChangeLog:
  - changeSet:
      id: 1
      author: john.doe
      changes:
        - createTable:
            tableName: users
            columns:
              - column:
                  name: id
                  type: BIGINT
                  autoIncrement: true
                  constraints:
                    primaryKey: true
              - column:
                  name: username
                  type: VARCHAR(50)
                  constraints:
                    nullable: false
                    unique: true
              - column:
                  name: email
                  type: VARCHAR(100)
                  constraints:
                    nullable: false
                    unique: true
      rollback:
        - dropTable:
            tableName: users
```

## CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/deploy.yml
name: Deploy with Migrations

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'

      - name: Backup Database
        run: |
          mysqldump -h ${{ secrets.DB_HOST }} \
                    -u ${{ secrets.DB_USER }} \
                    -p${{ secrets.DB_PASSWORD }} \
                    myapp_db > backup_$(date +%Y%m%d_%H%M%S).sql

      - name: Run Flyway Migrations
        run: ./mvnw flyway:migrate
        env:
          SPRING_DATASOURCE_URL: ${{ secrets.DATABASE_URL }}
          SPRING_DATASOURCE_USERNAME: ${{ secrets.DB_USER }}
          SPRING_DATASOURCE_PASSWORD: ${{ secrets.DB_PASSWORD }}

      - name: Verify Migrations
        run: ./mvnw flyway:info
```

## Troubleshooting

### Migration Failed

```bash
# Check what went wrong
./mvnw flyway:info

# Fix the database manually if needed
mysql -u root -p myapp_db

# Repair Flyway metadata
./mvnw flyway:repair

# Try again
./mvnw flyway:migrate
```

### Checksum Mismatch

```
ERROR: Migration checksum mismatch for migration version 3
```

**Cause**: Migration file was modified after being applied

**Solution**:
```bash
# Option 1: Repair (if you're sure the change is safe)
./mvnw flyway:repair

# Option 2: Create new migration instead
# Don't modify V3, create V4 with the fix
```

### Out of Order Migrations

```yaml
spring:
  flyway:
    out-of-order: true  # Allow applying missed migrations
```

### Baseline Existing Database

```bash
# For databases that already exist
./mvnw flyway:baseline -Dflyway.baselineVersion=1.0
```

## Summary: Flyway Checklist

✅ **Setup**:
- [ ] Add Flyway dependency
- [ ] Set `ddl-auto: validate`
- [ ] Configure `application.yml`
- [ ] Create `src/main/resources/db/migration/` directory

✅ **Naming**:
- [ ] Use `V{VERSION}__{DESCRIPTION}.sql` format
- [ ] Use sequential or date-based versioning
- [ ] Use snake_case for descriptions

✅ **Best Practices**:
- [ ] Never modify applied migrations
- [ ] Test migrations locally first
- [ ] Backup before production migrations
- [ ] Use transactions where possible
- [ ] Document complex migrations
- [ ] Review migration before committing

✅ **Production**:
- [ ] Set `clean-disabled: true`
- [ ] Enable `validate-on-migrate`
- [ ] Automate backups
- [ ] Monitor migration execution time
- [ ] Have rollback plan ready

## Related Documentation

- See [database.md](./database.md) for JPA configuration
- See [profiles.md](./profiles.md) for environment-specific settings
- See [../testing/integration-tests.md](../testing/integration-tests.md) for testing with migrations
