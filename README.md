# Claude Code Templates for Development Projects

This directory contains reusable Claude Code rule templates for different types of projects.

## Available Templates

### Angular Template (`angular/`)

**NEW**: Comprehensive rules and best practices for Angular 17+ projects with TypeScript and standalone components.

**Includes:**
- Architecture guidelines (layers, structure, naming) - **All in English**
- Frontend best practices (components, services, forms, routing)
- **Signals** for reactive state management (Angular 17+)
- **Standalone Components** architecture (no NgModules)
- Security (authentication, JWT, guards)
- State management (signals, computed, effects)
- Testing conventions (Jasmine/Karma, component testing)
- Docker deployment (nginx, multi-stage builds)
- **70+ best practices** (comprehensive coverage)

**Compatible with:**
- Angular 17+
- TypeScript 5.2+
- Standalone Components
- Signals API

**How to use:**
```bash
# Copy to your Angular project
cp -r angular/ /path/to/your/project/.claude/

# See angular/README.md for detailed instructions
cat angular/README.md
```

### Spring Boot Template (`springboot/`)

Comprehensive rules and best practices for Spring Boot projects with Java.

**Includes:**
- Architecture guidelines (layers, structure, naming) - **All in English**
- Backend best practices (entities, services, controllers, DTOs, repositories)
- **JPA Specifications** for type-safe queries (no @Query or native SQL)
- Security setup (JWT, authorization, CORS, **OWASP Top 10, rate limiting**)
- Error handling patterns
- Testing conventions (unit, integration)
- Configuration management
- API documentation (OpenAPI/Swagger)
- **Complete Git/GitHub workflow** (branching, commits, PRs, CI/CD)
- **Database Migrations** guide (Flyway/Liquibase with examples)
- **Caching Strategies** (Redis, Spring Cache, patterns)
- **Docker Deployment** (Dockerfile, docker-compose, production)
- **Monitoring & Observability** (Actuator, Prometheus, Grafana)
- **Async Processing** (@Async, events, messaging)
- **54 best practices** (comprehensive coverage)
- Feature implementation checklist

**Compatible with:**
- Spring Boot 3.x
- Java 17+
- MySQL/PostgreSQL
- Maven

**How to use:**
```bash
# Copy to your new Spring Boot project
cp -r springboot/ /path/to/your/project/.claude/

# See USAGE.md for detailed instructions
cat springboot/USAGE.md
```

## AI Agents for Spring Boot

**NEW**: Specialized AI agents to accelerate development! (`agents/`)

Six specialized agents designed for Spring Boot projects:

| Agent | Purpose |
|-------|---------|
| Code Reviewer | Quality, security, and best practices validation |
| Test Generator | Unit and integration test generation |
| Security Auditor | OWASP Top 10 vulnerability scanning |
| API Designer | RESTful API design and implementation |
| Entity Designer | JPA entity modeling with relationships |
| Refactor Assistant | Code quality improvements |

**See**: [AI Agents Guide](agents/README.md) for detailed usage.

## Purpose

These templates provide:

1. **Consistency**: Same conventions across all your projects
2. **Quick Start**: New projects start with battle-tested rules
3. **Onboarding**: Easy for new team members to understand project structure
4. **Quality**: Built-in best practices from day one
5. **AI-Powered**: Specialized agents to accelerate development
6. **CI/CD Ready**: Includes Git workflow and GitHub Actions examples

## Template Structure

Each template contains:

```
template-name/
├── README.md              # Overview and quick start
├── USAGE.md               # Detailed usage instructions
├── best-practices.md      # Core rules and conventions
├── checklist.md           # Implementation checklist
├── settings.local.json    # Claude Code settings
├── .gitignore             # Template-specific gitignore
├── architecture/          # Architecture docs
├── backend/               # Backend-specific docs
├── security/              # Security guidelines
├── testing/               # Testing conventions
├── configuration/         # Configuration management
├── documentation/         # Documentation standards
└── git-workflow/          # Git and GitHub workflow
    ├── github-workflow.md
    ├── commit-conventions.md
    └── pr-template.md
```

## AI Agents Structure

```
agents/
├── README.md                  # Agents guide and usage
├── code-reviewer.md           # Quality and security reviewer
├── test-generator.md          # Test generation specialist
├── security-auditor.md        # Security vulnerability scanner
├── api-designer.md            # RESTful API designer
├── entity-designer.md         # JPA entity designer
└── refactor-assistant.md      # Code refactoring expert
```

## Adding a New Template

1. Create a new directory with the template name
2. Follow the structure above
3. Write comprehensive documentation in English
4. Include USAGE.md with clear instructions
5. Add examples and code snippets
6. Test in a real project before committing

## Using Templates from GitHub

```bash
# Clone the repository
git clone https://github.com/Josansanchez/claude-code-java-templates.git

# For Spring Boot projects
cp -r claude-code-java-templates/springboot /path/to/your/project/.claude
cp -r claude-code-java-templates/agents /path/to/your/project/

# For Angular projects
cp -r claude-code-java-templates/angular /path/to/your/project/.claude

# Or use degit for cleaner copy (without git history)
npx degit Josansanchez/claude-code-java-templates/springboot /path/to/your/project/.claude
npx degit Josansanchez/claude-code-java-templates/angular /path/to/your/project/.claude
```

## Contributing

Improvements to templates are welcome:

1. Use the template in a real project
2. Identify improvements or missing documentation
3. Update the template
4. Test thoroughly
5. Submit a pull request

## Template Versions

### Spring Boot Template
- **Version**: 2.1.0
- **Last Updated**: 2025-11-03
- **Compatible with**: Spring Boot 3.x, Java 17+
- **New in 2.1.0**:
  - ✅ All documentation in English (architecture files translated)
  - ✅ JPA Specifications for type-safe queries (855-line comprehensive guide)
  - ✅ **Database Migrations** (Flyway/Liquibase, 700+ lines)
  - ✅ **Caching Strategies** (Redis, Spring Cache, 550+ lines)
  - ✅ **Docker Deployment** (Dockerfile, docker-compose, 650+ lines)
  - ✅ **Monitoring & Observability** (Actuator, Prometheus, Grafana, 450+ lines)
  - ✅ **Advanced Security** (OWASP Top 10, rate limiting, XSS, 550+ lines)
  - ✅ **Async Processing** (@Async, events, RabbitMQ, Kafka, 500+ lines)
  - ✅ **54 best practices** (updated from 47)
  - ✅ Reorganized structure (ready for multi-language templates)
  - ✅ 3,850+ lines of new production-ready documentation

## Best Practices for Templates

1. **Keep it Generic**: Use generic examples (User, Order, Product) instead of domain-specific terms
2. **Complete Documentation**: Every feature should be well-documented
3. **Code Examples**: Include working code examples in each doc
4. **Consistency**: Maintain consistent style across all documentation
5. **English Only**: All documentation in English for wider adoption
6. **Version Control**: Tag template versions and document changes

## Template Versions

### Angular Template
- **Version**: 1.0.0
- **Last Updated**: 2025-11-03
- **Compatible with**: Angular 17+, TypeScript 5.2+
- **Features**:
  - ✅ Standalone components architecture
  - ✅ Angular signals integration
  - ✅ Modern TypeScript patterns
  - ✅ Comprehensive testing strategies
  - ✅ Docker deployment with nginx
  - ✅ JWT authentication patterns
  - ✅ 70+ best practices

## Future Templates (Ideas)

- `react/` - React with TypeScript, Redux, Testing Library
- `python-fastapi/` - FastAPI with SQLAlchemy, Pydantic
- `nodejs-express/` - Express with TypeScript, Prisma
- `flutter/` - Flutter mobile app development
- `golang-gin/` - Go with Gin framework
- `dotnet-core/` - .NET Core Web API

## License

MIT License - See [LICENSE](LICENSE) file for details.

---

**Maintained by**: Josansanchez
**Repository**: https://github.com/Josansanchez/claude-code-java-templates
