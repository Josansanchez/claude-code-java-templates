# Pull Request Template Guide

## GitHub Pull Request Template

### Basic Template Format

Copy this template to `.github/pull_request_template.md` in your repository:

```markdown
## Summary

<!-- Brief description of what this PR does (2-3 sentences) -->

## Type of Change

- [ ] New feature (feat)
- [ ] Bug fix (fix)
- [ ] Refactoring (refactor)
- [ ] Performance improvement (perf)
- [ ] Documentation (docs)
- [ ] Tests (test)
- [ ] Build/CI changes (build/ci)
- [ ] Breaking change (BREAKING CHANGE)

## Changes

<!-- List the main changes in bullet points -->

-
-
-

## Test Plan

<!-- Describe how you tested these changes -->

- [ ] Unit tests added/updated
- [ ] Integration tests added/updated
- [ ] Manual testing performed
- [ ] All existing tests pass
- [ ] Code coverage maintained/improved

## Code Quality Checklist

- [ ] Code follows project style guidelines
- [ ] Self-review of code performed
- [ ] Comments added for complex logic
- [ ] No unnecessary console logs or debug code
- [ ] No hardcoded secrets or credentials
- [ ] Documentation updated (if needed)

## Database Changes

- [ ] No database changes
- [ ] Migration scripts added (Flyway/Liquibase)
- [ ] Migration tested locally
- [ ] Backward compatible changes

## Breaking Changes

<!-- If this introduces breaking changes, describe them here -->

- [ ] No breaking changes
- [ ] Breaking changes documented below

## Related Issues

<!-- Link related issues using keywords -->

Closes #
Fixes #
Related to #

## Screenshots/Logs (if applicable)

<!-- Add screenshots or logs to help explain your changes -->

## Deployment Notes

<!-- Any special deployment considerations? -->

## Reviewer Notes

<!-- Anything specific you want reviewers to focus on? -->
```

## Comprehensive PR Checklist

### Before Creating PR

#### Code Quality
- [ ] All tests pass locally (`./mvnw test`)
- [ ] Build succeeds (`./mvnw clean install`)
- [ ] No compiler warnings
- [ ] Code style checked (`./mvnw checkstyle:check`)
- [ ] No commented-out code blocks
- [ ] No debug statements (`System.out.println`, etc.)
- [ ] No pending TODO comments (or justified)

#### Testing
- [ ] Unit tests added for new functionality
- [ ] Integration tests if necessary
- [ ] Edge cases tested
- [ ] Error scenarios tested
- [ ] Test coverage >= 80%
- [ ] Tests properly named (shouldDoXxxWhenYyy)
- [ ] Mock usage is appropriate

#### Security
- [ ] No hardcoded credentials
- [ ] No `.env` files committed
- [ ] Input validation implemented
- [ ] SQL injection prevention
- [ ] XSS prevention measures
- [ ] CSRF protection (if applicable)
- [ ] Authorization checks on protected endpoints
- [ ] Sensitive data properly encrypted

#### Documentation
- [ ] OpenAPI/Swagger documentation updated
- [ ] README updated (if configuration changes)
- [ ] Complex logic commented
- [ ] API changes documented
- [ ] Migration guide (for breaking changes)

#### Database
- [ ] Flyway/Liquibase migrations created
- [ ] Migrations tested locally
- [ ] Migrations are reversible (if possible)
- [ ] Indexes added for performance
- [ ] No use of `ddl-auto=update` in production

#### Performance
- [ ] No N+1 query problems
- [ ] Appropriate use of database indexes
- [ ] Caching implemented where needed
- [ ] Lazy loading configured properly
- [ ] No memory leaks

## Examples of Good Pull Requests

### Example 1: Feature Addition

```markdown
## Summary

Add user registration endpoint with email validation and duplicate detection.
Users can now register with email, password, and basic profile information.
The system validates email format and prevents duplicate registrations.

## Type of Change

- [x] New feature (feat)

## Changes

- Created `UserRegistrationRequest` DTO with Bean Validation annotations
- Implemented `RegisterUserService` with email uniqueness check
- Added `RegisterUserController` with POST /api/v1/auth/register endpoint
- Created `UserRegistrationResponse` DTO for successful registration
- Added custom `EmailAlreadyExistsException` for duplicate emails
- Updated `GlobalExceptionHandler` to handle registration exceptions
- Added 12 unit tests for service layer
- Added 5 integration tests for the complete flow

## Test Plan

- [x] Unit tests added/updated (12 new tests)
- [x] Integration tests added/updated (5 new tests)
- [x] Manual testing performed with Postman
- [x] All existing tests pass (127/127)
- [x] Code coverage: 94% (increased from 89%)

**Test scenarios covered:**
- Valid registration with all required fields
- Invalid email format rejection
- Duplicate email detection
- Password strength validation
- Missing required fields handling
- SQL injection attempt prevention

## Code Quality Checklist

- [x] Code follows project style guidelines
- [x] Self-review of code performed
- [x] Comments added for email validation regex
- [x] No unnecessary console logs or debug code
- [x] No hardcoded secrets or credentials
- [x] Documentation updated (OpenAPI spec)

## Database Changes

- [x] No database changes (uses existing User table)

## Breaking Changes

- [x] No breaking changes

## Related Issues

Closes #123
Related to #45 (User management epic)

## API Documentation

**New Endpoint:**
```
POST /api/v1/auth/register
Content-Type: application/json

Request:
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "firstName": "John",
  "lastName": "Doe"
}

Response (201 Created):
{
  "id": 1,
  "email": "user@example.com",
  "firstName": "John",
  "lastName": "Doe",
  "createdAt": "2024-01-15T10:30:00Z"
}

Error Response (409 Conflict):
{
  "timestamp": "2024-01-15T10:30:00Z",
  "status": 409,
  "error": "Conflict",
  "message": "Email already registered",
  "path": "/api/v1/auth/register"
}
```

## Deployment Notes

No special deployment steps required. Backward compatible with existing code.

## Reviewer Notes

Please pay special attention to:
- Email validation regex in UserRegistrationRequest (line 23)
- Transaction handling in RegisterUserService (line 45)
- Error message internationalization approach
```

### Example 2: Bug Fix

```markdown
## Summary

Fix race condition in payment processing that caused duplicate charges when
multiple payment requests were submitted simultaneously for the same order.
The issue was identified in production and affected approximately 0.3% of
transactions during high-traffic periods.

## Type of Change

- [x] Bug fix (fix)

## Changes

- Added `@Version` field to Payment entity for optimistic locking
- Implemented idempotency key validation in PaymentService
- Added Redis-based distributed lock for payment processing
- Created `PaymentIdempotencyService` to manage idempotency keys
- Added retry logic with exponential backoff for lock acquisition
- Updated payment processing flow to check for duplicate payments
- Added 8 new unit tests for race condition scenarios
- Added integration test simulating concurrent payment requests

## Root Cause Analysis

**Problem:**
Multiple threads could process the same payment simultaneously because there
was no synchronization mechanism. The database constraint only prevented the
final save, but processing logic ran multiple times.

**Impact:**
- Duplicate payment attempts in 0.3% of high-traffic scenarios
- Customer confusion due to multiple authorization attempts
- Increased load on payment gateway
- Potential duplicate charges if gateway processed before database constraint

**Solution:**
1. Optimistic locking prevents database-level race conditions
2. Idempotency keys ensure requests are processed exactly once
3. Distributed lock prevents concurrent processing across instances
4. Retry logic handles transient lock failures gracefully

## Test Plan

- [x] Unit tests added (8 new tests for concurrency scenarios)
- [x] Integration test with 10 concurrent requests (all pass)
- [x] Load test with 100 concurrent users (0 duplicates)
- [x] All existing tests pass (145/145)
- [x] Code coverage: 91% (maintained)

**Specific tests:**
- Concurrent payment requests with same idempotency key
- Optimistic lock exception handling
- Redis connection failure fallback
- Idempotency key expiration
- Lock timeout scenarios

## Code Quality Checklist

- [x] Code follows project style guidelines
- [x] Self-review of code performed
- [x] Comments explaining the locking mechanism
- [x] No unnecessary console logs or debug code
- [x] No hardcoded secrets (Redis config externalized)
- [x] Documentation updated with new payment flow diagram

## Database Changes

- [x] Migration scripts added (V1.5__add_payment_version.sql)
- [x] Migration tested locally
- [x] Backward compatible (version column nullable initially)

**Migration:**
```sql
ALTER TABLE payments ADD COLUMN version BIGINT DEFAULT 0;
ALTER TABLE payments ADD COLUMN idempotency_key VARCHAR(255);
CREATE UNIQUE INDEX idx_payment_idempotency ON payments(idempotency_key);
```

## Breaking Changes

- [x] No breaking changes

**Note:** Added optional `Idempotency-Key` header to payment endpoint,
but requests without it still work (system generates one).

## Related Issues

Fixes #789 (Duplicate payment charges in production)
Related to #790 (Payment system reliability improvements)

## Performance Impact

**Before:**
- Average payment processing: 450ms
- P99 latency: 1200ms

**After:**
- Average payment processing: 475ms (+25ms for lock acquisition)
- P99 latency: 1250ms (+50ms acceptable overhead)
- Redis lock overhead: 15-30ms average
- Zero duplicate payments in testing

## Deployment Notes

**Prerequisites:**
1. Redis must be running and accessible
2. Run database migration before deploying
3. Configure Redis connection in application.yml:
   ```yaml
   spring:
     redis:
       host: ${REDIS_HOST:localhost}
       port: ${REDIS_PORT:6379}
       timeout: 2000ms
   ```

**Deployment steps:**
1. Deploy Redis if not already running
2. Run Flyway migration
3. Deploy application (zero-downtime compatible)
4. Monitor Redis connection pool metrics

**Rollback plan:**
If issues occur, the old code will still work. The version column being
nullable ensures backward compatibility. Simply redeploy previous version.

## Monitoring

Added new metrics:
- `payment.lock.acquired` - successful lock acquisitions
- `payment.lock.timeout` - lock timeout occurrences
- `payment.idempotency.duplicate` - duplicate request detections

## Reviewer Notes

Please review:
1. Redis lock implementation in PaymentService (lines 78-95)
2. Exception handling for OptimisticLockException (line 120)
3. Idempotency key generation strategy (line 45)
4. Lock timeout value (currently 5 seconds - is this appropriate?)
```

### Example 3: Refactoring

```markdown
## Summary

Extract validation logic from service classes into dedicated validator components
to improve code organization, testability, and reusability. This refactoring does
not change any functionality but makes the codebase more maintainable and easier
to test.

## Type of Change

- [x] Refactoring (refactor)

## Changes

- Created `UserValidator` class with dedicated validation methods
- Created `OrderValidator` class for order business rules
- Created `PaymentValidator` class for payment validations
- Extracted validation logic from UserService (150 lines reduced to 45)
- Extracted validation logic from OrderService (200 lines reduced to 60)
- Extracted validation logic from PaymentService (100 lines reduced to 35)
- Added `@Component` annotation to make validators Spring beans
- Updated services to inject and use validators
- Added 24 unit tests for validator classes
- Refactored existing service tests to be cleaner

## Motivation

**Before:**
- Service classes were bloated with validation logic
- Validation logic was difficult to test in isolation
- Duplicate validation code across different services
- Hard to maintain and extend validation rules

**After:**
- Services focus on business logic only
- Validators are easily testable in isolation
- Validation logic is centralized and reusable
- Clear separation of concerns
- Easier to add new validation rules

## Test Plan

- [x] Added 24 unit tests for new validator classes
- [x] All existing tests still pass (145/145)
- [x] Integration tests pass (25/25)
- [x] Code coverage maintained at 91%
- [x] Manual testing confirms no behavior changes

**Validation:**
- Compared output of old vs. new implementation (100% match)
- No changes to API contracts
- No changes to error messages
- No changes to validation rules

## Code Quality Checklist

- [x] Code follows project style guidelines
- [x] Self-review of code performed
- [x] Comments added for complex validation rules
- [x] No unnecessary console logs or debug code
- [x] No hardcoded values (all configurable)
- [x] JavaDoc added to all validator methods

## Database Changes

- [x] No database changes

## Breaking Changes

- [x] No breaking changes

**Internal API changes:**
- Service constructors now require validator dependencies
- This only affects tests and configuration (Spring handles injection)
- All existing functionality preserved

## Related Issues

Related to #321 (Code quality improvement initiative)
Part of technical debt reduction effort

## Code Metrics

### Lines of Code Reduced

| Class | Before | After | Reduction |
|-------|--------|-------|-----------|
| UserService | 350 | 200 | 43% |
| OrderService | 450 | 250 | 44% |
| PaymentService | 280 | 180 | 36% |
| **Total Services** | **1,080** | **630** | **42%** |

### New Validator Classes

| Class | Lines | Test Coverage |
|-------|-------|---------------|
| UserValidator | 120 | 95% |
| OrderValidator | 150 | 93% |
| PaymentValidator | 80 | 97% |

### Complexity Reduction

- Cyclomatic complexity of services reduced by average of 35%
- Average method length in services reduced from 25 to 12 lines
- Number of dependencies per service reduced by 2-3

## Code Examples

**Before:**
```java
@Service
public class UserService {
    public UserDTO createUser(UserRequest request) {
        // Validation logic (50 lines)
        if (request.getEmail() == null || request.getEmail().isBlank()) {
            throw new ValidationException("Email is required");
        }
        if (!request.getEmail().matches(EMAIL_REGEX)) {
            throw new ValidationException("Invalid email format");
        }
        // ... 45 more lines of validation ...

        // Actual business logic (10 lines)
        User user = userMapper.toEntity(request);
        return userMapper.toDTO(userRepository.save(user));
    }
}
```

**After:**
```java
@Service
public class UserService {
    private final UserValidator validator;

    public UserDTO createUser(UserRequest request) {
        validator.validateUserRequest(request);

        User user = userMapper.toEntity(request);
        return userMapper.toDTO(userRepository.save(user));
    }
}

@Component
public class UserValidator {
    public void validateUserRequest(UserRequest request) {
        validateEmail(request.getEmail());
        validatePassword(request.getPassword());
        validateName(request.getName());
    }

    private void validateEmail(String email) {
        if (email == null || email.isBlank()) {
            throw new ValidationException("Email is required");
        }
        if (!email.matches(EMAIL_REGEX)) {
            throw new ValidationException("Invalid email format");
        }
    }
    // ... more focused validation methods ...
}
```

## Migration Guide for Developers

If you have feature branches that modify services:

1. Update service constructor calls to include validators
2. Replace inline validation with validator method calls
3. Move any new validation logic to appropriate validator
4. Update tests to mock validators instead of testing validation in service tests

Example:
```java
// Old test
@Test
void shouldThrowExceptionWhenEmailIsInvalid() {
    // Testing validation in service test
}

// New test - move to ValidatorTest
@Test
void shouldThrowExceptionWhenEmailIsInvalid() {
    // Test in UserValidatorTest instead
}
```

## Deployment Notes

No special deployment considerations. This is a pure refactoring with no
functional changes. All changes are backward compatible.

## Reviewer Notes

Please verify:
1. No behavior changes (same validation rules, same error messages)
2. Test coverage maintained
3. Validator separation makes sense
4. No duplicate validation logic remains
5. Service classes are now cleaner and more focused

**Large files changed:**
This PR touches many files but each change is straightforward:
- Moving validation code to validators
- Updating service constructors
- Adding validator tests

Recommend reviewing validator classes first, then service changes.
```

## Writing Good PR Descriptions

### The Perfect PR Title

**Format:** `<type>(<scope>): <short description>`

**Examples:**
```
feat(auth): add two-factor authentication
fix(payment): resolve race condition in payment processing
refactor(service): extract validation logic to separate validators
perf(query): optimize user search with database indexes
docs(api): add OpenAPI documentation for new endpoints
```

### The Perfect PR Description

A good PR description should answer:

1. **What** changed?
2. **Why** was it changed?
3. **How** was it implemented?
4. **How** was it tested?
5. **What** is the impact?

### Description Template Structure

```markdown
## Summary (What & Why)
2-3 sentences explaining the change and motivation

## Changes (How - High Level)
Bullet list of main changes

## Technical Details (How - Implementation)
Deeper explanation of complex parts

## Test Plan (Verification)
How you verified it works

## Impact Analysis (Consequences)
Performance, breaking changes, deployment notes
```

## PR Size Guidelines

### Ideal PR Sizes

- **Small PR**: 1-200 lines (1-2 hours to review)
  - Single feature
  - Bug fix
  - Small refactoring
  - **Review time:** < 30 minutes

- **Medium PR**: 200-400 lines (2-4 hours to review)
  - Feature with tests
  - Multiple related changes
  - Moderate refactoring
  - **Review time:** 30-60 minutes

- **Large PR**: 400-800 lines (4-8 hours to review)
  - Complex feature
  - Large refactoring
  - **Review time:** 1-2 hours
  - **Action:** Consider splitting if possible

- **Very Large PR**: > 800 lines
  - **Warning:** Very difficult to review
  - **Strong recommendation:** Split into smaller PRs
  - **If unavoidable:** Provide detailed documentation and walkthrough

### How to Keep PRs Small

1. **Feature flags**: Deploy incomplete features behind flags
2. **Incremental development**: Break features into smaller chunks
3. **Separate refactoring**: Don't mix refactoring with new features
4. **Separate tests**: Can split tests into separate PR if needed
5. **Stacked PRs**: Create dependent PRs that build on each other

### When Large PRs Are Acceptable

- Generated code (migrations, OpenAPI specs)
- Package updates with many dependency changes
- Initial project setup
- Large-scale automated refactoring (with tool)

## Common PR Mistakes to Avoid

### 1. Vague Descriptions

**Bad:**
```markdown
## Summary
Fixed some bugs and added features
```

**Good:**
```markdown
## Summary
Fix race condition in payment processing that caused duplicate charges
in high-traffic scenarios. Added distributed locking using Redis and
idempotency key validation to ensure exactly-once processing.
```

### 2. Missing Test Information

**Bad:**
```markdown
## Test Plan
- [x] Tested manually
```

**Good:**
```markdown
## Test Plan
- [x] Added 12 unit tests covering edge cases
- [x] Added integration test with 10 concurrent requests
- [x] Manual testing with 100-user load test
- [x] Verified no duplicate payments in production-like environment
- [x] All 145 existing tests still pass
```

### 3. No Context for Reviewers

**Bad:**
```markdown
Changed the payment service
```

**Good:**
```markdown
## Summary
The payment service had a critical race condition causing duplicate charges.

## Root Cause
Multiple threads could process the same payment simultaneously without
synchronization, leading to duplicate database inserts until the unique
constraint failed.

## Solution
Implemented three-layer protection:
1. Optimistic locking at database level
2. Idempotency keys for request deduplication
3. Distributed locks for cross-instance synchronization

## Reviewer Focus Areas
Please pay special attention to:
- Lock timeout handling (is 5 seconds appropriate?)
- Redis failure fallback strategy
- Performance impact of lock acquisition
```

### 4. Mixing Unrelated Changes

**Bad:**
```markdown
- Add user registration
- Fix payment bug
- Refactor order service
- Update dependencies
- Fix typo in README
```

**Good:**
```markdown
Create separate PRs:
1. feat(auth): add user registration
2. fix(payment): resolve race condition
3. refactor(order): extract validation logic
4. chore(deps): update Spring Boot to 3.2.1
5. docs(readme): fix typo in installation section
```

### 5. No Breaking Change Documentation

**Bad:**
```markdown
Changed API endpoint from /login to /api/v1/auth/login
```

**Good:**
```markdown
## Breaking Changes

⚠️ **BREAKING CHANGE**: Authentication endpoint structure changed

**Old endpoint:**
```
POST /login
```

**New endpoint:**
```
POST /api/v1/auth/login
```

**Migration required for:**
- All client applications
- Mobile apps
- Third-party integrations

**Migration guide:**
See docs/migration/auth-endpoints-v2.md

**Timeline:**
- Old endpoint deprecated: 2024-02-01
- Old endpoint removed: 2024-03-01
```

### 6. Inadequate Testing

**Bad:**
```markdown
Tests:
- [x] It works
```

**Good:**
```markdown
## Test Coverage

**Unit Tests (15 new):**
- UserValidator: 8 tests (100% coverage)
- UserService: 5 tests (95% coverage)
- UserController: 2 tests (90% coverage)

**Integration Tests (3 new):**
- Complete registration flow (happy path)
- Duplicate email handling
- Invalid input validation

**Edge Cases Tested:**
- SQL injection attempts
- XSS in user input
- Concurrent registration attempts
- Database connection failure
- Email service unavailable

**Performance Test:**
- 1000 registrations/minute (all successful)
- Average response time: 145ms
- P99 latency: 320ms
```

### 7. No Deployment Considerations

**Bad:**
```markdown
Ready to merge
```

**Good:**
```markdown
## Deployment Notes

**Prerequisites:**
1. Redis server must be running
2. Environment variables required:
   - REDIS_HOST
   - REDIS_PORT
   - REDIS_PASSWORD

**Database Migration:**
Run `V1.5__add_payment_version.sql` before deployment

**Configuration Changes:**
Add to application.yml:
```yaml
app:
  payment:
    lock-timeout: 5000
    idempotency-ttl: 86400
```

**Rollback Procedure:**
If issues occur:
1. Redeploy previous version
2. Migration is backward compatible (no rollback needed)
3. Clear Redis cache: `redis-cli FLUSHDB`

**Monitoring:**
Watch these metrics for first 24 hours:
- payment.lock.timeout (should be < 0.1%)
- payment.processing.time (should be < 500ms avg)
```

### 8. Not Responding to Review Comments

**Bad:**
- Ignore reviewer comments
- Mark conversations as resolved without addressing
- Make changes without explaining

**Good:**
- Respond to every comment
- Explain your reasoning if you disagree
- Thank reviewers for catching issues
- Update code based on feedback
- Mark resolved only after addressing

## PR Review Best Practices

### For PR Authors

1. **Self-review first**: Review your own PR before requesting review
2. **Provide context**: Explain complex decisions in comments
3. **Highlight concerns**: Point out areas you're unsure about
4. **Be responsive**: Reply to comments within 24 hours
5. **Accept feedback**: Reviews make your code better
6. **Update promptly**: Address comments and update the PR
7. **Keep it updated**: Merge master regularly to avoid conflicts

### For Reviewers

1. **Review promptly**: Aim to review within 24 hours
2. **Be constructive**: Explain why, not just what
3. **Ask questions**: "Why did you choose this approach?"
4. **Approve with suggestions**: Don't block on minor issues
5. **Test locally**: Pull the branch and test if needed
6. **Check tests**: Verify tests are meaningful
7. **Verify documentation**: Ensure docs are updated

## PR Review Checklist

### Code Review

- [ ] Code is readable and well-organized
- [ ] No code duplication
- [ ] No hardcoded values (use configuration)
- [ ] Error handling is appropriate
- [ ] Logging is adequate but not excessive
- [ ] No security vulnerabilities
- [ ] Performance considerations addressed
- [ ] Memory leaks prevented

### Tests Review

- [ ] Tests are meaningful (not just for coverage)
- [ ] Edge cases are covered
- [ ] Error scenarios are tested
- [ ] Test names are descriptive
- [ ] No flaky tests
- [ ] Integration tests if needed
- [ ] Mocks are used appropriately

### Architecture Review

- [ ] Follows project patterns
- [ ] Separation of concerns maintained
- [ ] Dependency injection used properly
- [ ] No circular dependencies
- [ ] Database access optimized
- [ ] API design is RESTful
- [ ] DTOs used for data transfer

## Related Documentation

- [GitHub Workflow](./github-workflow.md) - Complete Git workflow guide
- [Commit Conventions](./commit-conventions.md) - How to write good commit messages
