# Commit Conventions

## Conventions for Commit Messages

We follow the **Conventional Commits** standard to maintain a clean and semantic commit history.

## Commit Format

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Complete Example

```
feat(auth): add user registration endpoint

- Implement UserRegistrationRequest DTO with validation rules
- Create RegisterUserService with email uniqueness check
- Add RegisterUserController with proper HTTP responses
- Include unit tests for service and controller
- Add integration tests for the complete flow

Closes #123
```

## Types

### Main Types

- **feat**: New feature
  ```
  feat(user): add user profile endpoint
  feat(auth): implement JWT refresh token
  ```

- **fix**: Bug fix
  ```
  fix(login): resolve null pointer exception on invalid credentials
  fix(validation): correct email regex pattern
  ```

- **refactor**: Code refactoring without changing functionality
  ```
  refactor(service): extract validation logic to separate class
  refactor(controller): simplify response mapping
  ```

- **test**: Add or modify tests
  ```
  test(user): add unit tests for user service
  test(integration): add integration tests for auth flow
  ```

- **docs**: Documentation changes
  ```
  docs(readme): update installation instructions
  docs(api): add OpenAPI documentation for new endpoints
  ```

- **style**: Formatting changes (don't affect logic)
  ```
  style(controller): apply code formatting
  style(all): organize imports
  ```

- **perf**: Performance improvements
  ```
  perf(query): optimize user search with database indexes
  perf(cache): implement Redis caching for frequent queries
  ```

- **chore**: Maintenance tasks
  ```
  chore(deps): update Spring Boot to 3.2.1
  chore(build): configure Maven profiles
  ```

- **build**: Build system changes
  ```
  build(maven): add new dependency for JWT
  build(docker): update Dockerfile with multi-stage build
  ```

- **ci**: CI/CD changes
  ```
  ci(github): add automated testing workflow
  ci(deploy): configure production deployment pipeline
  ```

- **revert**: Revert a previous commit
  ```
  revert: revert "feat(auth): add OAuth integration"

  This reverts commit 1234567890abcdef
  ```

## Scope

The scope indicates which part of the code was modified:

### Examples by Module

- **auth**: Authentication and authorization
  ```
  feat(auth): implement two-factor authentication
  ```

- **user**: User management
  ```
  fix(user): resolve duplicate email issue
  ```

- **api**: General API
  ```
  refactor(api): standardize error responses
  ```

- **db**: Database
  ```
  feat(db): add Flyway migration for new tables
  ```

- **security**: Security
  ```
  fix(security): patch XSS vulnerability
  ```

- **config**: Configuration
  ```
  chore(config): update production YAML settings
  ```

- **test**: Testing
  ```
  test(integration): add end-to-end test suite
  ```

## Subject

The subject is a brief description of the change:

### Rules

1. **Maximum 50 characters**
2. **Use imperative mood** ("add" not "added" or "adds")
3. **Do not end with a period**
4. **Lowercase** after the colon
5. **Be specific** but concise

### Good Examples ✅

```
feat(auth): add JWT token validation
fix(user): resolve null pointer in user update
refactor(service): extract email validation logic
test(controller): add unit tests for CRUD operations
docs(readme): update environment variables section
```

### Bad Examples ❌

```
feat: stuff                          // Too vague
fix(user): Fixed the bug             // Past tense instead of imperative
feat(auth): Added new feature.       // Past tense and with period
refactor: Refactoring the code       // Redundant and not specific
Update                               // No type or scope
```

## Body

The body explains **what and why**, not how (that's in the code).

### Rules

1. **Optional** but recommended for complex changes
2. **Separate from subject** with a blank line
3. **Maximum 72 characters** per line
4. **Use bullet points** for multiple changes
5. **Explain the context** and motivation

### Complete Example

```
feat(auth): implement OAuth2 authentication

- Add OAuth2 configuration for Google and GitHub providers
- Create OAuth2UserService to handle user registration
- Implement custom success and failure handlers
- Add redirect URLs configuration in application.yml
- Include security filters in SecurityFilterChain

This allows users to sign in using their existing accounts,
reducing registration friction and improving user experience.
```

## Footer

The footer includes:
- Issue references
- Breaking changes
- Additional metadata

### Issue References

```
Closes #123
Fixes #456
Resolves #789
Related to #101, #102
```

### Breaking Changes

```
BREAKING CHANGE: Authentication endpoint changed from /login to /api/v1/auth/login

Update all client applications to use the new endpoint.
Migration guide: docs/migration/v2-auth.md
```

### Complete Example with Footer

```
feat(api)!: change authentication endpoint structure

- Rename /login to /api/v1/auth/login
- Rename /logout to /api/v1/auth/logout
- Add versioning to all auth endpoints
- Update documentation and Swagger

BREAKING CHANGE: Authentication endpoints have been moved to /api/v1/auth/*

Update your client applications to use:
- POST /api/v1/auth/login
- POST /api/v1/auth/logout

Migration guide: docs/migration/auth-endpoints.md

Closes #234
```

## Atomic Commits

Each commit should be **atomic**: represents one complete logical change.

### ✅ Good Atomic Commits

```
feat(user): add user profile endpoint
feat(user): add user profile validation
test(user): add tests for user profile endpoint
docs(api): document user profile endpoint
```

### ❌ Non-Atomic Commit

```
feat: implement complete user management system

- Add user CRUD endpoints
- Implement authentication
- Add role management
- Fix bug in login
- Update documentation
- Refactor services
```

This commit does too many things and should be split up.

## Golden Rules

1. **Frequent commits**: Better many small commits than one large one
2. **Compilable**: Each commit must leave the code in a compilable state
3. **Tests passing**: Tests must pass after each commit
4. **Descriptive message**: Someone should understand what changed without seeing the code
5. **Consistent language**: Always use English for commits

## Useful Tools

### Commitizen (Optional)

```bash
# Install globally
npm install -g commitizen cz-conventional-changelog

# Use
git cz

# Will follow a wizard to create conventional commits
```

### Git Commit Template

Create `.gitmessage` in your home directory:

```
# <type>(<scope>): <subject>
# |<----  Using a Maximum Of 50 Characters  ---->|

# <body>
# |<----   Try To Limit Each Line to a Maximum Of 72 Characters   ---->|

# <footer>
# Example:
# feat(auth): add JWT token validation
#
# - Implement JWT token decoder
# - Add token expiration check
# - Create custom authentication filter
#
# Closes #123

# Types: feat, fix, refactor, test, docs, style, perf, chore, build, ci, revert
```

Configure:
```bash
git config --global commit.template ~/.gitmessage
```

## Complete Examples

### Feature with Tests

```
feat(order): add order cancellation feature

- Create CancelOrderService with validation rules
- Add CancelOrderController with proper authorization
- Implement order status transition logic
- Add unit tests for service layer
- Add integration tests for complete flow
- Update OrderStatus enum with CANCELLED state

Users can now cancel orders within 24 hours of placement.
The feature includes proper authorization checks to ensure
users can only cancel their own orders.

Closes #456
```

### Bug Fix

```
fix(payment): resolve race condition in payment processing

The payment processing service had a race condition when
multiple requests were made simultaneously for the same order.
This caused duplicate charges.

Solution:
- Add optimistic locking to Payment entity
- Implement idempotency key validation
- Add distributed lock using Redis
- Include retry logic for failed operations

Fixes #789
```

### Refactoring

```
refactor(service): extract validation logic to separate validators

- Create UserValidator class with specific validation rules
- Create OrderValidator class for order business rules
- Remove validation logic from service methods
- Add unit tests for new validator classes

This improves code organization and makes validators reusable
across different parts of the application. Also makes testing
easier by isolating validation logic.

Related to #321
```

### Breaking Change

```
feat(api)!: migrate to new error response format

- Create standardized ErrorResponse DTO
- Update GlobalExceptionHandler to use new format
- Remove old MessageResponse class
- Update all controllers to return new format
- Update API documentation

BREAKING CHANGE: Error response format has changed

Old format:
{
  "message": "Error description"
}

New format:
{
  "timestamp": "2024-01-15T10:30:00Z",
  "status": 400,
  "error": "Bad Request",
  "message": "Error description",
  "path": "/api/v1/users"
}

Update your client applications to handle the new error format.

Closes #567
```

## Quick Reference

```
Types:
  feat      → New feature
  fix       → Bug fix
  refactor  → Code refactoring
  test      → Add/update tests
  docs      → Documentation
  style     → Code formatting
  perf      → Performance improvement
  chore     → Maintenance
  build     → Build system
  ci        → CI/CD changes
  revert    → Revert previous commit

Format:
  <type>(<scope>): <subject>

  <body>

  <footer>

Rules:
  • Subject: imperative, lowercase, max 50 chars, no period
  • Body: wrap at 72 chars, explain what and why
  • Footer: reference issues, breaking changes
  • Atomic commits: one logical change per commit
```

## Related Documentation
- [GitHub Workflow](./github-workflow.md) - Complete Git workflow
- [PR Template](./pr-template.md) - Pull request template
