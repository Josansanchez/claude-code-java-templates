# Claude Code Templates for Development Projects

This directory contains reusable Claude Code rule templates for different types of projects.

## Available Templates

### Spring Boot Template (`springboot/`)

Comprehensive rules and best practices for Spring Boot projects with Java.

**Includes:**
- Architecture guidelines (layers, structure, naming)
- Backend best practices (entities, services, controllers, DTOs, repositories)
- Security setup (JWT, authorization, CORS)
- Error handling patterns
- Testing conventions (unit, integration)
- Configuration management
- API documentation (OpenAPI/Swagger)
- **Complete Git/GitHub workflow** (branching, commits, PRs, CI/CD)
- 47 best practices
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

# Copy desired template to your project
cp -r claude-code-java-templates/springboot /path/to/your/project/.claude

# Copy AI agents
cp -r claude-code-java-templates/agents /path/to/your/project/

# Or use degit for cleaner copy (without git history)
npx degit Josansanchez/claude-code-java-templates/springboot /path/to/your/project/.claude
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
- **Version**: 2.0.0
- **Last Updated**: 2025-01-02
- **Compatible with**: Spring Boot 3.x, Java 17+
- **New**: AI Agents for accelerated development

## Best Practices for Templates

1. **Keep it Generic**: Use generic examples (User, Order, Product) instead of domain-specific terms
2. **Complete Documentation**: Every feature should be well-documented
3. **Code Examples**: Include working code examples in each doc
4. **Consistency**: Maintain consistent style across all documentation
5. **English Only**: All documentation in English for wider adoption
6. **Version Control**: Tag template versions and document changes

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
