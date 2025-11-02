# Claude Code Templates for Development Projects

This directory contains reusable Claude Code rule templates for different types of projects.

## Available Templates

### Spring Boot Template (`springboot/`)

Comprehensive rules and best practices for Spring Boot projects with Java.

**Includes:**
- ✅ Architecture guidelines (layers, structure, naming)
- ✅ Backend best practices (entities, services, controllers, DTOs, repositories)
- ✅ Security setup (JWT, authorization, CORS)
- ✅ Error handling patterns
- ✅ Testing conventions (unit, integration)
- ✅ Configuration management
- ✅ API documentation (OpenAPI/Swagger)
- ✅ **Complete Git/GitHub workflow** (branching, commits, PRs, CI/CD)
- ✅ 47 best practices
- ✅ Feature implementation checklist

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

## Purpose

These templates provide:

1. **Consistency**: Same conventions across all your projects
2. **Quick Start**: New projects start with battle-tested rules
3. **Onboarding**: Easy for new team members to understand project structure
4. **Quality**: Built-in best practices from day one
5. **CI/CD Ready**: Includes Git workflow and GitHub Actions examples

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

## Adding a New Template

1. Create a new directory with the template name
2. Follow the structure above
3. Write comprehensive documentation in English
4. Include USAGE.md with clear instructions
5. Add examples and code snippets
6. Test in a real project before committing

## Creating a GitHub Repository

To share these templates publicly or within your organization:

```bash
# Initialize git repository
cd .claude-templates
git init

# Add files
git add .
git commit -m "feat: initial commit of Spring Boot template

- Complete Spring Boot 3.x template
- Architecture and backend guidelines
- Security and testing best practices
- Git workflow documentation with commit conventions and PR templates
- 47 best practices and feature checklist"

# Add remote and push
git remote add origin https://github.com/your-username/claude-code-templates.git
git branch -M main
git push -u origin main
```

### Repository Structure on GitHub

```
claude-code-templates/
├── README.md              # This file
├── LICENSE                # MIT or Apache 2.0
├── springboot/            # Spring Boot template
│   ├── README.md
│   ├── USAGE.md
│   └── ...
├── react/                 # Future: React template
├── python-fastapi/        # Future: FastAPI template
└── nodejs-express/        # Future: Express template
```

## Using Templates from GitHub

Once published to GitHub:

```bash
# Clone the repository
git clone https://github.com/your-username/claude-code-templates.git

# Copy desired template to your project
cp -r claude-code-templates/springboot /path/to/your/project/.claude

# Or use degit for cleaner copy (without git history)
npx degit your-username/claude-code-templates/springboot /path/to/your/project/.claude
```

## Contributing

Improvements to templates are welcome:

1. Use the template in a real project
2. Identify improvements or missing documentation
3. Update the template
4. Test thoroughly
5. Submit a pull request (if using GitHub)

## Template Versions

### Spring Boot Template
- **Version**: 1.0.0
- **Last Updated**: 2025-01-02
- **Compatible with**: Spring Boot 3.x, Java 17+

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

These templates are provided as-is for use in your projects. Feel free to modify and adapt them to your needs.

---

**Maintained by**: [Your Name/Organization]
**Repository**: [GitHub URL]
**Issues**: [GitHub Issues URL]
