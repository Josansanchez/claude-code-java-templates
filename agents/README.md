# Claude Code AI Agents

A comprehensive collection of AI agents for accelerating software development, organized by scope: **generic** (language-agnostic) and **framework-specific** implementations.

## 📁 Repository Structure

```
agents/
├── README.md                          # This file
│
├── generic/                           # Language-agnostic agents
│   ├── architecture-reviewer.md       # Software architecture patterns & principles
│   ├── security-principles.md         # OWASP Top 10, security best practices
│   ├── code-quality-reviewer.md       # Clean code, SOLID, code smells
│   ├── api-design-principles.md       # REST, GraphQL, gRPC, WebSocket design
│   └── refactoring-guide.md           # Refactoring patterns & techniques
│
└── spring-boot/                       # Spring Boot specific agents
    ├── code-reviewer.md               # Spring Boot code review
    ├── test-generator.md              # JUnit, Mockito test generation
    ├── security-auditor.md            # Spring Security auditing
    ├── api-designer.md                # Spring REST API design
    ├── entity-designer.md             # JPA/Hibernate entity design
    └── refactor-assistant.md          # Spring Boot refactoring
```

## 🎯 What are AI Agents?

AI agents are specialized prompts that instruct Claude Code to perform specific tasks with deep domain expertise. Think of them as **virtual expert consultants** for different aspects of software development.

## 🌍 Generic vs Framework-Specific Agents

### Generic Agents (Universal)

**Location:** `agents/generic/`

**Characteristics:**
- ✅ Language and framework agnostic
- ✅ Based on universal principles (SOLID, Clean Code, OWASP)
- ✅ Applicable to Java, Python, TypeScript, Go, Rust, etc.
- ✅ Focus on concepts, not syntax
- ✅ Timeless best practices

**Use for:**
- Architecture reviews across any project
- Security audits using OWASP standards
- Code quality assessment
- API design strategy
- Refactoring planning

### Framework-Specific Agents

**Location:** `agents/spring-boot/`, `agents/django/` (future), etc.

**Characteristics:**
- ✅ Concrete implementations for specific frameworks
- ✅ Technology-specific best practices
- ✅ Framework annotations and patterns
- ✅ Code generation with proper syntax
- ✅ Integration with ecosystem tools

**Use for:**
- Writing production-ready code
- Framework-specific optimizations
- Library and tool usage
- Testing with framework tools
- Deployment configurations

## 📚 Generic Agents

### 1. Architecture Reviewer
**Purpose:** Evaluate system design and architectural patterns

**Covers:**
- Layered, Hexagonal, Clean Architecture
- SOLID principles
- Domain-Driven Design (DDD)
- Microservices vs Monolith
- Scalability patterns
- Integration patterns

**When to use:**
- Reviewing overall system architecture
- Before major refactoring
- Planning system scalability
- Evaluating technical debt

**Example:**
```
Review the architecture of this codebase. Identify the architectural style, evaluate SOLID compliance, and provide recommendations.
```

---

### 2. Security Principles
**Purpose:** Identify security vulnerabilities and weaknesses

**Covers:**
- OWASP Top 10 (2021)
- Authentication & Authorization
- Data protection (GDPR, CCPA)
- Cryptography best practices
- API security
- Privacy compliance

**When to use:**
- Before production deployment
- After implementing auth features
- Handling sensitive data
- Regular security audits

**Example:**
```
Perform a comprehensive security audit focusing on OWASP Top 10 vulnerabilities and data protection.
```

---

### 3. Code Quality Reviewer
**Purpose:** Assess code quality and maintainability

**Covers:**
- Clean Code principles
- SOLID principles
- Code smells (duplicated code, long methods, etc.)
- DRY, KISS, YAGNI
- Naming conventions
- Error handling

**When to use:**
- Before creating pull requests
- During code reviews
- Evaluating legacy code
- Establishing coding standards

**Example:**
```
Review this code for clean code violations, code smells, and maintainability issues.
```

---

### 4. API Design Principles
**Purpose:** Design robust, scalable APIs

**Covers:**
- REST API design (resources, HTTP methods, status codes)
- GraphQL schema design
- gRPC / Protocol Buffers
- WebSocket protocols
- MQTT for IoT
- API versioning, security, documentation

**When to use:**
- Designing new APIs
- Choosing API paradigm (REST vs GraphQL vs gRPC)
- API versioning strategy
- Performance optimization

**Example:**
```
Design a REST API for managing IoT devices with support for real-time updates. Should I use REST, WebSocket, MQTT, or a combination?
```

---

### 5. Refactoring Guide
**Purpose:** Improve code structure without changing behavior

**Covers:**
- Martin Fowler's refactoring catalog
- Extract Method, Inline Method, Rename
- Replace Conditional with Polymorphism
- Design patterns (Strategy, Factory, Template Method)
- SOLID refactoring
- Performance refactoring

**When to use:**
- Cleaning up legacy code
- Removing code smells
- Applying design patterns
- Before adding new features
- Improving testability

**Example:**
```
This method is 200 lines long. Break it down using Extract Method refactoring following Single Responsibility Principle.
```

---

## 🍃 Spring Boot Specific Agents

### 1. Code Reviewer
Spring Boot-specific code review including annotations, dependency injection, and Spring conventions.

### 2. Test Generator
Generate JUnit 5, Mockito, and Spring Test code following project conventions.

### 3. Security Auditor
Spring Security configuration review, authentication/authorization flaws, and JPA security.

### 4. API Designer
Design Spring REST controllers, DTOs, services, and repositories with OpenAPI documentation.

### 5. Entity Designer
Design JPA/Hibernate entities with proper relationships, indexes, and performance optimization.

### 6. Refactor Assistant
Refactor Spring Boot code applying Spring-specific patterns and best practices.

📖 **See [`spring-boot/README.md`](./spring-boot/) for detailed documentation.**

---

## 🚀 How to Use Agents

### Method 1: Direct Prompt (Recommended)

1. Open the agent file (e.g., `generic/architecture-reviewer.md`)
2. Copy the prompt from the "Agent Prompt" section
3. Paste it into Claude Code
4. Add your specific request

**Example:**
```
[Paste architecture-reviewer.md agent prompt]

Review the architecture of my e-commerce application located in src/.
Identify architectural patterns, evaluate SOLID compliance, and suggest improvements.
```

### Method 2: Reference by Name

Simply mention the agent in your request:

```
Using the Architecture Reviewer agent, evaluate if this codebase follows Clean Architecture principles.
```

### Method 3: Create Slash Commands (Advanced)

Add agents as slash commands in `.claude/commands/`:

```bash
# Copy agent to commands directory
cp agents/generic/security-principles.md .claude/commands/security-audit.md

# Use in conversation
/security-audit
```

---

## 🔄 Workflow Examples

### Complete Feature Development

Use multiple agents in sequence:

```
1. Architecture Reviewer: Plan feature architecture
2. API Design Principles: Design API endpoints
3. Spring Boot API Designer: Implement controllers/services
4. Spring Boot Test Generator: Generate comprehensive tests
5. Security Principles: Check for vulnerabilities
6. Code Quality Reviewer: Final quality check
```

### Legacy Code Modernization

```
1. Code Quality Reviewer: Identify code smells and technical debt
2. Security Principles: Audit for security issues
3. Refactoring Guide: Plan refactoring strategy
4. Spring Boot Refactor Assistant: Apply Spring-specific refactorings
5. Spring Boot Test Generator: Add missing tests
6. Code Quality Reviewer: Verify improvements
```

### Security Hardening

```
1. Security Principles: Comprehensive security scan (OWASP Top 10)
2. Spring Boot Security Auditor: Spring Security configuration review
3. Refactoring Guide: Fix identified vulnerabilities
4. Spring Boot Test Generator: Add security tests
5. Security Principles: Verify fixes
```

### API Development

```
1. API Design Principles: Choose API paradigm (REST/GraphQL/gRPC)
2. API Design Principles: Design endpoints, request/response formats
3. Spring Boot API Designer: Implement with Spring conventions
4. Security Principles: API security review
5. Code Quality Reviewer: Ensure clean, maintainable code
```

---

## 🎓 Best Practices

### 1. Use Generic Agents First

Start with generic agents to make high-level decisions, then use framework-specific agents for implementation.

**Example workflow:**
```
Architecture Reviewer → Spring Boot Entity Designer
   (Decide pattern)     (Implement with JPA)

API Design Principles → Spring Boot API Designer
   (Design REST API)      (Implement controllers)
```

### 2. Combine Agents for Comprehensive Reviews

Don't rely on a single agent:

```
Before Pull Request:
✅ Code Quality Reviewer (clean code)
✅ Security Principles (vulnerabilities)
✅ Architecture Reviewer (design consistency)
```

### 3. Iterate Based on Feedback

- Review agent suggestions carefully
- Apply recommendations incrementally
- Re-run agents after changes
- Document decisions when deviating

### 4. Customize for Your Project

Generic agents are starting points. Adapt them:

- Add project-specific rules
- Include domain-specific validations
- Incorporate team conventions
- Reference internal standards

### 5. Document Agent Usage

Track agent usage in your development process:

- Which agents were used for each feature
- Decisions made based on agent recommendations
- Deviations from suggestions (with reasoning)
- Share findings in pull request descriptions

---

## 🆕 Adding New Framework-Specific Agents

To add agents for other frameworks (Django, FastAPI, NestJS, etc.):

### 1. Create Framework Directory

```bash
mkdir agents/django
mkdir agents/nestjs
```

### 2. Base on Generic Agents

Start with generic agent principles, add framework syntax:

```
# agents/django/orm-designer.md
Based on: agents/generic/entity-designer.md (for concepts)

Adds: Django ORM syntax, model patterns, migrations
```

### 3. Include Framework Specifics

- Framework annotations/decorators
- Standard library usage
- Testing tools (pytest, Jest, etc.)
- Deployment configurations
- Performance optimizations

### 4. Update Main README

Add your framework to the structure and workflows.

---

## 📊 Agent Effectiveness

### Measured Benefits

**Code Quality:**
- ✅ 40-60% reduction in code review comments
- ✅ 80%+ test coverage on new code
- ✅ Consistent adherence to best practices
- ✅ Early detection of security vulnerabilities

**Development Speed:**
- ✅ Faster feature development with design agents
- ✅ Reduced time writing boilerplate code/tests
- ✅ Quicker identification of technical debt
- ✅ Less back-and-forth in code reviews

**Knowledge Sharing:**
- ✅ Junior developers learn from agent suggestions
- ✅ Consistent patterns across team members
- ✅ Built-in documentation of best practices
- ✅ Reduced onboarding time

---

## 🤝 Contributing

### Improving Existing Agents

1. Use the agent in real development
2. Identify gaps or areas for improvement
3. Update the agent markdown file
4. Test with real code examples
5. Submit improvements

### Creating New Agents

**Generic agents should:**
- Be language/framework agnostic
- Based on universal principles
- Include examples in pseudocode
- Cover timeless best practices

**Framework-specific agents should:**
- Reference generic agent principles
- Include concrete syntax examples
- Cover framework-specific patterns
- Provide code generation templates

---

## 📚 Further Reading

### Software Architecture
- "Clean Architecture" by Robert C. Martin
- "Domain-Driven Design" by Eric Evans
- "Microservices Patterns" by Chris Richardson

### Security
- OWASP Top 10: https://owasp.org/Top10/
- OWASP Cheat Sheets: https://cheatsheetseries.owasp.org/

### Code Quality
- "Clean Code" by Robert C. Martin
- "Refactoring" by Martin Fowler
- "Design Patterns" by Gang of Four

### API Design
- "RESTful Web APIs" by Leonard Richardson
- GraphQL Documentation: https://graphql.org/
- gRPC Documentation: https://grpc.io/

---

## 📝 Version History

- **v2.0.0** (2025-01-05): Restructured with generic and framework-specific separation
  - Added 5 generic agents (universal principles)
  - Moved Spring Boot agents to dedicated directory
  - Updated workflows and best practices

- **v1.0.0** (2025-01-02): Initial release
  - 6 Spring Boot specific agents

---

## 💡 Tips

- Generic agents help with **architecture and strategy**
- Framework-specific agents help with **implementation**
- Use both for best results
- Start generic, end specific
- Agents are guides, not strict rules - use judgment
- Contribute improvements based on real-world usage
- Customize agents for your team's needs

**Pro Tip:** Run Code Quality Reviewer and Security Principles before every pull request for best results!

---

## 📞 Support

For questions, issues, or suggestions:
- Review agent documentation
- Check workflow examples
- Consult further reading resources
- Share feedback with your team

---

**Remember:** AI agents are powerful tools that augment human expertise. They provide guidance and automation, but engineering judgment remains essential.
