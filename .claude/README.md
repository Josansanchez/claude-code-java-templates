# Spring Boot Development Guide with Claude Code

> **Comprehensive development guide for Spring Boot 3.x projects with Java 17+**

This documentation defines conventions, best practices, and AI agents for Spring Boot development using Claude Code.

## 📚 Documentation Index

### 🏗️ Architecture
- [Project Structure](architecture/project-structure.md) - Directory organization and layers
- [Application Layers](architecture/layers.md) - Controller → Service → Repository
- [Naming Conventions](architecture/naming-conventions.md) - File and method naming standards

### 💻 Backend
- [Entities (JPA)](backend/entities.md) - Database models, Lombok, relationships
- [Repositories](backend/repositories.md) - Data access with Spring Data JPA
- [Services](backend/services.md) - Business logic, transactions, mappers
- [Controllers](backend/controllers.md) - REST endpoints, HTTP codes, documentation
- [DTOs](backend/dtos.md) - Data Transfer Objects and validation
- [Mappers](backend/mappers.md) - MapStruct and ModelMapper

### 🔒 Security
- [Authentication](security/authentication.md) - JWT, cookies, configuration
- [Authorization](security/authorization.md) - Roles, @PreAuthorize
- [CORS](security/cors.md) - Global CORS configuration

### ⚠️ Error Handling
- [Custom Exceptions](error-handling/exceptions.md) - Domain-specific exceptions
- [Global Exception Handler](error-handling/global-handler.md) - Centralized @ControllerAdvice

### 🧪 Testing
- [Unit Tests](testing/unit-tests.md) - Mockito, @WebMvcTest, @DataJpaTest
- [Integration Tests](testing/integration-tests.md) - @SpringBootTest, Testcontainers
- [Test Conventions](testing/test-conventions.md) - Naming and structure

### ⚙️ Configuration
- [Spring Profiles](configuration/profiles.md) - YAML per environment (dev, test, prod)
- [Database](configuration/database.md) - JPA, Flyway, MySQL
- [Dependencies](configuration/dependencies.md) - pom.xml and libraries

### 📖 Documentation
- [OpenAPI/Swagger](documentation/openapi.md) - API documentation
- [Logging](documentation/logging.md) - SLF4J and log levels

### 🏠 IoT (Optional)
- [MQTT](iot/mqtt.md) - Configuration and services for IoT devices

### ✅ Quick Guides
- [Best Practices](best-practices.md) - 47 essential rules
- [Feature Checklist](checklist.md) - Steps to implement new features

---

## 🤖 AI Agents

**NEW**: Specialized AI agents to accelerate development and maintain code quality!

### Available Agents

| Agent | Purpose | Use When |
|-------|---------|----------|
| [🔍 Code Reviewer](agents/code-reviewer.md) | Ensures quality, security, and best practices | Before PR, after feature |
| [🧪 Test Generator](agents/test-generator.md) | Generates unit and integration tests | After implementation, low coverage |
| [🔒 Security Auditor](agents/security-auditor.md) | Identifies OWASP Top 10 vulnerabilities | Before production, security reviews |
| [🎨 API Designer](agents/api-designer.md) | Designs RESTful APIs end-to-end | New endpoints, API refactoring |
| [🗄️ Entity Designer](agents/entity-designer.md) | Creates JPA entities with relationships | New entities, schema design |
| [♻️ Refactor Assistant](agents/refactor-assistant.md) | Improves code quality and maintainability | Legacy code, code smells |

**Learn more**: See [AI Agents Guide](agents/README.md) for detailed usage instructions.

### Quick Agent Usage

```
# Review code before PR
Using the Code Reviewer agent, review UserService.java

# Generate comprehensive tests
Using the Test Generator agent, create tests for CreateUserService.java

# Security audit
Using the Security Auditor agent, check AuthController.java for vulnerabilities

# Design complete API
Using the API Designer agent, create CRUD API for Product entity

# Design entity with relationships
Using the Entity Designer agent, create Order entity with OrderItems

# Refactor legacy code
Using the Refactor Assistant agent, refactor UserManager.java to follow SOLID principles
```

---

## 🚀 Quick Start

### Core Principle
**One public method per file** in Controllers and Services.

### Basic Structure
```
controllers/user/
├── CreateUserController.java → createUser()
├── GetUserController.java → getUser()
└── DeleteUserController.java → deleteUser()

services/user/
├── CreateUserService.java → createUser()
├── GetUserService.java → getUser()
└── DeleteUserService.java → deleteUser()

repositories/
└── UserRepository.java

mappers/
└── UserMapper.java
```

### Key Conventions
- Directories in **lowercase** (user, product, auth)
- Files in **CamelCase** (CreateUserController.java)
- API versioning (`/api/v1/users`)
- **No @CrossOrigin** in controllers (use global config)
- **No try-catch** in controllers (use @ControllerAdvice)
- **MapStruct** for mapping (not manual)
- **@Transactional(readOnly=true)** for read operations
- **Custom exceptions** with @ResponseStatus

## 🛠️ Tech Stack

- **Framework**: Spring Boot 3.4.2
- **Language**: Java 17+
- **Database**: MySQL 8.0
- **ORM**: JPA/Hibernate
- **Security**: Spring Security + JWT
- **IoT**: Eclipse Paho MQTT (optional)
- **Mapping**: MapStruct
- **Testing**: JUnit 5, Mockito, Testcontainers
- **Documentation**: SpringDoc OpenAPI

## 💡 Development Workflow

### 1. Plan Your Feature
- Review [Feature Checklist](checklist.md)
- Use [Entity Designer](agents/entity-designer.md) for data models
- Use [API Designer](agents/api-designer.md) for endpoints

### 2. Implement
- Follow [Architecture Guidelines](architecture/layers.md)
- Apply [Best Practices](best-practices.md)
- Use [Code Reviewer](agents/code-reviewer.md) during development

### 3. Test
- Use [Test Generator](agents/test-generator.md) for comprehensive tests
- Follow [Test Conventions](testing/test-conventions.md)
- Aim for 80%+ coverage on business logic

### 4. Review Security
- Use [Security Auditor](agents/security-auditor.md)
- Check [Security Guidelines](security/authentication.md)
- Verify OWASP Top 10 compliance

### 5. Code Review & Deploy
- Use [Code Reviewer](agents/code-reviewer.md) before PR
- Address all agent feedback
- Create pull request

## 📞 Support

For questions about these conventions:
- Check the relevant documentation section
- Consult with the development team
- Review AI agent suggestions
- Open an issue on the project repository

---

**Last Updated**: 2025-01-02
**Version**: 2.0.0 (with AI Agents)
