# Claude Code AI Agents for Spring Boot

A collection of specialized AI agents designed to accelerate Spring Boot development, maintain code quality, and enforce best practices.

## What are AI Agents?

AI agents are specialized prompts that instruct Claude Code to perform specific tasks with deep domain expertise. Each agent is crafted to handle a particular aspect of software development, acting as a virtual expert team member.

## Available Agents

### 🔍 [Code Reviewer](./code-reviewer.md)
Expert code reviewer that ensures quality, security, and adherence to best practices.

**Use when:**
- Before creating a pull request
- After implementing a new feature
- During code review process
- Validating legacy code quality

**Key capabilities:**
- Architecture and structure validation
- Clean code principles (SOLID, DRY, KISS)
- Security vulnerabilities (OWASP Top 10)
- Performance optimization checks
- Spring Boot best practices
- JPA/Hibernate patterns

**Example:**
```
Please review UserService.java for code quality, security, and performance issues.
```

---

### 🧪 [Test Generator](./test-generator.md)
Automatically generates comprehensive unit and integration tests following project conventions.

**Use when:**
- After implementing new services or controllers
- When test coverage is low
- Creating test templates for new features
- Retrofitting tests to legacy code

**Key capabilities:**
- Unit tests for services with Mockito
- Controller tests with MockMvc
- Repository tests with @DataJpaTest
- Integration tests with @SpringBootTest
- AAA pattern (Arrange, Act, Assert)
- 80%+ code coverage

**Example:**
```
Generate comprehensive unit tests for CreateUserService.java including happy path, error scenarios, and edge cases.
```

---

### 🔒 [Security Auditor](./security-auditor.md)
Comprehensive security expert identifying vulnerabilities based on OWASP Top 10 and industry best practices.

**Use when:**
- Before deploying to production
- After implementing authentication/authorization
- During security reviews or penetration testing prep
- Handling sensitive user data

**Key capabilities:**
- OWASP Top 10 vulnerability scanning
- SQL injection, XSS, CSRF detection
- Authentication/authorization flaws
- Sensitive data exposure
- Security misconfiguration
- Dependency vulnerability analysis

**Example:**
```
Perform a comprehensive security audit of AuthController.java and SecurityConfig.java focusing on authentication vulnerabilities.
```

---

### 🎨 [API Designer](./api-designer.md)
Expert RESTful API designer creating production-ready endpoints following HTTP semantics and REST principles.

**Use when:**
- Designing new API endpoints
- Creating CRUD operations for entities
- Refactoring APIs to RESTful standards
- Planning API versioning strategy

**Key capabilities:**
- RESTful URL design
- Proper HTTP methods and status codes
- Request/response DTOs with validation
- Complete implementation (Controller → Repository)
- OpenAPI/Swagger documentation
- Pagination, filtering, nested resources

**Example:**
```
Design a complete RESTful API for managing Product entities including CRUD operations, pagination, and search functionality.
```

---

### 🗄️ [Entity Designer](./entity-designer.md)
Expert JPA/Hibernate entity designer creating optimized database models with proper relationships.

**Use when:**
- Designing new database entities
- Creating entity relationships
- Optimizing existing entity models
- Planning database schemas

**Key capabilities:**
- Entity relationships (@OneToMany, @ManyToOne, @ManyToMany)
- Performance optimization (lazy/eager loading)
- Cascade types and orphan removal
- Inheritance strategies
- Indexing and constraints
- Embedded objects and composite keys

**Example:**
```
Design a Product entity with a many-to-one relationship to Category and a one-to-many relationship to ProductImage, including proper indexes.
```

---

### ♻️ [Refactor Assistant](./refactor-assistant.md)
Code refactoring specialist improving code quality, maintainability, and performance while preserving functionality.

**Use when:**
- Cleaning up legacy code
- Removing code smells and anti-patterns
- Applying SOLID principles
- Optimizing performance
- Breaking down large, monolithic classes

**Key capabilities:**
- Clean Code principles (SOLID, DRY, KISS)
- Code smell detection and fixes
- Design pattern application
- Performance optimization
- Extract method/class refactorings
- Reduce complexity and nesting

**Example:**
```
Refactor UserService.java to follow Single Responsibility Principle and remove code smells like long methods and duplicate code.
```

---

## How to Use Agents

### Method 1: Direct Prompt Copy-Paste
1. Open the agent file (e.g., `code-reviewer.md`)
2. Copy the prompt from the "Agent Prompt" section
3. Paste it into your Claude Code conversation
4. Add your specific request

### Method 2: Reference in Conversation
Simply reference the agent in your request:

```
Using the Code Reviewer agent, please review the changes in UserController.java
```

### Method 3: Create Custom Slash Commands
Add agents as slash commands in `.claude/commands/`:

```bash
# Create command file
echo "$(cat agents/code-reviewer.md)" > .claude/commands/review.md

# Use in conversation
/review UserService.java
```

## Agent Workflow Examples

### Complete Feature Development
```
1. Use Entity Designer: Design User entity with roles
2. Use API Designer: Create REST API for user management
3. Use Test Generator: Generate tests for all services
4. Use Security Auditor: Check for vulnerabilities
5. Use Code Reviewer: Final quality check
```

### Legacy Code Modernization
```
1. Use Code Reviewer: Identify issues in legacy code
2. Use Refactor Assistant: Clean up code smells
3. Use Test Generator: Add missing tests
4. Use Security Auditor: Fix security vulnerabilities
5. Use Code Reviewer: Verify improvements
```

### Security Hardening
```
1. Use Security Auditor: Comprehensive security scan
2. Use Refactor Assistant: Fix identified vulnerabilities
3. Use Test Generator: Add security tests
4. Use Code Reviewer: Verify security fixes
```

## Best Practices

### 1. Use Agents Proactively
Don't wait for problems - use agents throughout development:
- **Design phase**: Entity Designer, API Designer
- **Implementation**: Code Reviewer, Test Generator
- **Pre-PR**: Security Auditor, Code Reviewer
- **Maintenance**: Refactor Assistant

### 2. Combine Agents
Agents work best together:
- Design entities → Generate API → Create tests → Review security
- Each agent complements the others

### 3. Iterate Based on Feedback
- Review agent suggestions carefully
- Apply recommendations incrementally
- Re-run agents after changes to verify improvements

### 4. Customize for Your Project
- Adjust agent prompts for project-specific conventions
- Add domain-specific validation rules
- Include project-specific security requirements

### 5. Document Agent Usage
- Track which agents were used for each feature
- Document decisions when deviating from agent suggestions
- Share agent findings in PR descriptions

## Agent Effectiveness

### Code Quality Improvements
- ✅ 40-60% reduction in code review comments
- ✅ 80%+ test coverage on new code
- ✅ Consistent adherence to project conventions
- ✅ Early detection of security vulnerabilities

### Development Speed
- ✅ Faster feature development with API Designer
- ✅ Reduced time spent writing boilerplate tests
- ✅ Quicker identification of refactoring opportunities
- ✅ Less back-and-forth in code reviews

### Knowledge Sharing
- ✅ Junior developers learn from agent suggestions
- ✅ Consistent patterns across team members
- ✅ Built-in documentation of best practices
- ✅ Reduced onboarding time

## Feedback and Improvements

These agents are living documents. Improve them based on:

1. **Real-world usage**: What works? What doesn't?
2. **Team feedback**: What do developers find most helpful?
3. **Project evolution**: New patterns, technologies, requirements
4. **Security landscape**: New vulnerabilities, updated OWASP guidance

## Contributing

To improve an agent:

1. Use it in real development
2. Identify gaps or areas for improvement
3. Update the agent prompt
4. Test with real code examples
5. Document changes in commit message

## Version History

- **v1.0.0** (2025-01-02): Initial release
  - Code Reviewer
  - Test Generator
  - Security Auditor
  - API Designer
  - Entity Designer
  - Refactor Assistant

## Support

For questions or suggestions:
- Open an issue on GitHub
- Discuss with your team
- Update agents based on lessons learned

---

**Remember**: These agents are guides, not strict rules. Use professional judgment to adapt suggestions to your specific context.

**Pro Tip**: Run Code Reviewer and Security Auditor before every pull request for best results!
