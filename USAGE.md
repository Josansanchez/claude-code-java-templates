# How to Use This Spring Boot Template

## Quick Start

### 1. Copy Template to Your New Project

```bash
# Navigate to your new Spring Boot project
cd /path/to/your/new/project

# Copy the .claude directory from this template
cp -r /path/to/.claude-templates/springboot/ .claude/

# On Windows (PowerShell)
# Copy-Item -Recurse -Path "C:\path\to\.claude-templates\springboot" -Destination ".claude"
```

### 2. Customize for Your Project

Edit `.claude/README.md` and update:
- Project name and description
- Specific tech stack versions
- Add project-specific sections

Example:
```markdown
# Your Project Name - Development Guide

> **Your project description here**

## Tech Stack
- Framework: Spring Boot 3.2.1
- Language: Java 21
- Database: PostgreSQL 15
- ...
```

### 3. Configure Git Workflow (Recommended)

#### Option A: Use GitHub PR Template (Recommended)

Create `.github/PULL_REQUEST_TEMPLATE.md` in your repository:

```bash
mkdir -p .github
cp .claude/git-workflow/pr-template.md .github/PULL_REQUEST_TEMPLATE.md
```

This will automatically show the template when creating Pull Requests on GitHub.

#### Option B: Configure Local Git Hooks

Set up pre-push validation (ensures tests pass before pushing):

```bash
# Copy the pre-push hook example from git-workflow/github-workflow.md
# to .git/hooks/pre-push and make it executable

chmod +x .git/hooks/pre-push
```

### 4. Add Project-Specific Features

Create additional directories for your specific needs:

```bash
cd .claude

# Example: For a messaging system
mkdir messaging
echo "# Messaging Configuration\n\n..." > messaging/kafka-setup.md

# Example: For caching
mkdir cache
echo "# Redis Cache Setup\n\n..." > cache/redis-config.md

# Example: For IoT (like the original project)
mkdir iot
echo "# MQTT Configuration\n\n..." > iot/mqtt-setup.md
```

### 5. Configure Claude Code Permissions (Optional)

Edit `.claude/settings.local.json` to add project-specific commands:

```json
{
  "permissions": {
    "allow": [
      "Bash(./mvnw test)",
      "Bash(./mvnw clean install)",
      "Bash(npm run build)",
      "Bash(git status)"
    ],
    "deny": [],
    "ask": []
  }
}
```

## What's Included

### Core Documentation

- **Architecture** (3 files)
  - Project structure
  - Application layers (Controller → Service → Repository)
  - Naming conventions

- **Backend** (6 files)
  - Entities (JPA)
  - Services
  - Controllers
  - DTOs
  - Mappers
  - Repositories

- **Security** (3 files)
  - Authentication (JWT)
  - Authorization (roles)
  - CORS configuration

- **Error Handling** (2 files)
  - Custom exceptions
  - Global exception handler

- **Testing** (3 files)
  - Unit tests
  - Integration tests
  - Testing conventions

- **Configuration** (3 files)
  - Spring profiles
  - Database setup
  - Dependencies management

- **Documentation** (2 files)
  - OpenAPI/Swagger
  - Logging best practices

### Git Workflow

- **github-workflow.md** - Complete branching strategy, PR process, CI/CD
- **commit-conventions.md** - Conventional commits standard
- **pr-template.md** - Pull Request template for GitHub

### Quick References

- **best-practices.md** - Essential rules (47 best practices)
- **checklist.md** - Step-by-step checklist for new features

## Common Workflows

### Starting a New Feature

1. Read: `.claude/checklist.md`
2. Follow: `.claude/git-workflow/github-workflow.md`
3. Create branch: `feature/your-feature-name`
4. Develop following: `.claude/backend/` guidelines
5. Test following: `.claude/testing/` guidelines
6. Create PR using: `.claude/git-workflow/pr-template.md`

### Code Review Checklist

Use `.claude/git-workflow/pr-template.md` checklist sections:
- Code quality
- Testing
- Security
- Documentation
- Database
- Performance

### Implementing CRUD Operations

Follow `.claude/checklist.md` - "Example: Complete CRUD" section

## Customization Examples

### Example 1: E-commerce Project

```bash
cd .claude

# Add e-commerce specific docs
mkdir payment
echo "# Payment Gateway Integration\n\n..." > payment/stripe-setup.md

mkdir inventory
echo "# Inventory Management\n\n..." > inventory/stock-control.md

# Update README.md with e-commerce context
# Replace generic examples (User, Order, Product) with actual domain models
```

### Example 2: Healthcare Project

```bash
cd .claude

# Add healthcare specific docs
mkdir hipaa
echo "# HIPAA Compliance Guidelines\n\n..." > hipaa/compliance.md

mkdir ehr
echo "# EHR Integration\n\n..." > ehr/fhir-api.md

# Update security docs with healthcare-specific requirements
```

### Example 3: Microservices Project

```bash
cd .claude

# Add microservices specific docs
mkdir microservices
echo "# Service Discovery\n\n..." > microservices/eureka-config.md
echo "# API Gateway\n\n..." > microservices/gateway-setup.md
echo "# Inter-service Communication\n\n..." > microservices/feign-clients.md
```

## Updating the Template

If you improve the rules and want to update the base template:

```bash
# From your project
cd .claude

# Copy improved files back to template (excluding project-specific ones)
cp architecture/*.md /path/to/.claude-templates/springboot/architecture/
cp backend/*.md /path/to/.claude-templates/springboot/backend/
# ... etc

# Don't copy project-specific folders
# (e.g., iot/, payment/, inventory/)
```

## Best Practices for Using This Template

1. **Keep it Updated**: When you improve a rule, update the template for future projects

2. **Don't Modify Core Files**: Instead of modifying core files, create additional files for project-specific rules

3. **Share Improvements**: If you create useful generic documentation, contribute it back to the template

4. **Consistency Across Projects**: Use the same template for all your Spring Boot projects to maintain consistency

5. **Onboarding Tool**: Use this documentation to onboard new team members quickly

## Troubleshooting

### Issue: Files Not Showing Up in Claude Code

**Solution**: Make sure the `.claude` directory is in the root of your project:
```bash
ls -la .claude
# Should show all the files
```

### Issue: Git Workflow Not Working

**Solution**: Ensure you've configured branch protection rules in GitHub Settings → Branches

### Issue: Tests Not Running in Pre-Push Hook

**Solution**: Make the hook executable:
```bash
chmod +x .git/hooks/pre-push
```

## Resources

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [GitHub Flow](https://guides.github.com/introduction/flow/)
- [MapStruct Documentation](https://mapstruct.org/)
- [JUnit 5 Documentation](https://junit.org/junit5/docs/current/user-guide/)

## Support

For issues or improvements to this template, please:
1. Check existing documentation in `.claude/`
2. Review examples in each file
3. Consult with your team lead
4. Create an issue in the template repository (if applicable)

---

**Template Version**: 1.0.0
**Last Updated**: 2025-01-02
