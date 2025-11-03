# Spring Boot Project Template - Claude Code Rules

> **Generic rules and best practices template for Spring Boot projects with Java**

This template defines conventions, best practices, and development standards for Spring Boot projects.

## 🚀 How to Use This Template

### 1. Copy the Template to Your Project
```bash
# From this repository
cp -r .claude-templates/springboot/.claude /path/to/your/project/

# Or clone and copy
git clone <repo-url>
cp -r <repo>/.claude-templates/springboot/.claude /path/to/your/project/
```

### 2. Customize for Your Project
- Edit `README.md` to include project-specific information
- Add additional folders for specific features (e.g., `iot/`, `messaging/`, etc.)
- Update the tech stack in README.md according to your project
- Adjust rules based on your domain's particular needs

### 3. Configure Git Workflow (Optional)
- Review `git-workflow/github-workflow.md` to configure your workflow
- Adjust branching rules according to your team
- Configure hooks if needed in `settings.local.json`

## 📚 Documentation Index

### 🏗️ Architecture
- [Project Structure](architecture/project-structure.md) - Directory organization and layers
- [Application Layers](architecture/layers.md) - Controller → Service → Repository
- [Naming Conventions](architecture/naming-conventions.md) - File and method naming

### 💻 Backend
- [Entities (JPA)](backend/entities.md) - Database models, Lombok, relationships
- [Repositories](backend/repositories.md) - Data access with Spring Data JPA
- [**Specifications**](backend/specifications.md) - **Type-safe queries, composable patterns**
- [Services](backend/services.md) - Business logic, transactions, mappers
- [Controllers](backend/controllers.md) - REST endpoints, HTTP codes, documentation
- [DTOs](backend/dtos.md) - Data Transfer Objects and validations
- [Mappers](backend/mappers.md) - MapStruct and ModelMapper

### 🔒 Security
- [Authentication](security/authentication.md) - JWT, cookies, configuration
- [Authorization](security/authorization.md) - Roles, @PreAuthorize
- [CORS](security/cors.md) - Global CORS configuration
- [**Advanced Security**](security/advanced-security.md) - **OWASP Top 10, rate limiting, input sanitization**

### ⚠️ Error Handling
- [Custom Exceptions](error-handling/exceptions.md) - Creating specific exceptions
- [Global Exception Handler](error-handling/global-handler.md) - Centralized @ControllerAdvice

### 🧪 Testing
- [Unit Tests](testing/unit-tests.md) - Mockito, @WebMvcTest, @DataJpaTest
- [Integration Tests](testing/integration-tests.md) - @SpringBootTest, Testcontainers
- [Testing Conventions](testing/test-conventions.md) - Naming and structure

### ⚙️ Configuration
- [Spring Profiles](configuration/profiles.md) - YAML per environment (dev, test, prod)
- [Database](configuration/database.md) - JPA configuration, connection pooling
- [**Database Migrations**](configuration/database-migrations.md) - **Flyway/Liquibase, versioning, best practices**
- [**Caching**](configuration/caching.md) - **Redis, Spring Cache, strategies, patterns**
- [Dependencies](configuration/dependencies.md) - pom.xml and libraries

### 📖 Documentation
- [OpenAPI/Swagger](documentation/openapi.md) - API documentation
- [Logging](documentation/logging.md) - SLF4J and log levels

### 🔄 Git Workflow
- [GitHub Workflow](git-workflow/github-workflow.md) - Branching, testing, PR
- [Commit Conventions](git-workflow/commit-conventions.md) - Commit messages
- [PR Template](git-workflow/pr-template.md) - Pull Request template

### 🚀 Deployment
- [**Docker**](deployment/docker.md) - **Dockerfile, docker-compose, production deployment**
- [**Monitoring**](deployment/monitoring.md) - **Spring Actuator, Prometheus, Grafana dashboards**

### 🎯 Advanced Topics
- [**Async Processing**](advanced/async-processing.md) - **@Async, events, RabbitMQ, Kafka**

### ✅ Quick Guides
- [Best Practices](best-practices.md) - Essential rules
- [Checklist for New Features](checklist.md) - Steps to implement features

## 🚀 Quick Start

### Fundamental Rule
**One public method per file** in Controllers and Services.

### Basic Structure
```
controllers/<resource>/
├── Create<Resource>Controller.java → create<Resource>()
├── Get<Resource>Controller.java → get<Resource>()
└── Delete<Resource>Controller.java → delete<Resource>()

services/<resource>/
├── Create<Resource>Service.java → create<Resource>()
├── Get<Resource>Service.java → get<Resource>()
└── Delete<Resource>Service.java → delete<Resource>()

repositories/<resource>/
└── <Resource>Repository.java

mappers/
└── <Resource>Mapper.java
```

### Key Conventions
- Directories in **lowercase** (user, order, product)
- Files in **CamelCase** (CreateUserController.java)
- API with **version** (`/api/v1/user`)
- **No @CrossOrigin** in controllers (configure globally)
- **No try-catch** in controllers (use @ControllerAdvice)
- **MapStruct** for mapping (not manual)
- **@Transactional(readOnly=true)** for reads
- **Custom exceptions** with @ResponseStatus

## 🛠️ Base Tech Stack

- **Framework**: Spring Boot 3.x
- **Language**: Java 17+
- **Database**: MySQL/PostgreSQL
- **ORM**: JPA/Hibernate
- **Security**: Spring Security + JWT
- **Mapping**: MapStruct
- **Testing**: JUnit 5, Mockito, Testcontainers
- **Documentation**: SpringDoc OpenAPI

## 🎯 Template Philosophy

This template is designed for:

1. **Consistency**: All Spring Boot projects follow the same conventions
2. **Scalability**: Structure that scales from small microservices to large applications
3. **Maintainability**: Easy to understand and modify code
4. **Testability**: Architecture that facilitates unit and integration testing
5. **Security**: Security best practices incorporated from the start

## 📝 Customization for Your Project

1. **Add Specific Features**: Create additional folders in `.claude/` for specific technologies (e.g., `messaging/`, `cache/`, `iot/`)
2. **Adapt Examples**: Replace generic examples with examples from your domain
3. **Configure Hooks**: Define validation hooks in `settings.local.json`
4. **Document Specifics**: Add documentation about unique architectural decisions in your project

## 🤝 Contributions

If you improve these rules, consider updating the base template to benefit other projects.

---

**Template Version**: 1.0.0
**Compatible with**: Spring Boot 3.x, Java 17+
**Last updated**: 2025-01-02
