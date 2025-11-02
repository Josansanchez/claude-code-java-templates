# GitHub Workflow - Spring Boot Projects

## Complete Workflow from Development to Production Integration

This document defines the complete workflow from development to production integration.

## 🌳 Branching Strategy

### Main Branches

```
master (or main)
  ↑
  └── Pull Requests from feature branches
```

### Branch Types

1. **`master` (or `main`)**: Main production branch
   - Must always be stable and ready to deploy
   - Only updated via reviewed Pull Requests
   - Protected with branch protection rules

2. **`feature/<name>`**: New feature development branches
   - Format: `feature/add-user-authentication`
   - Created from `master`
   - Deleted after merge

3. **`bugfix/<name>`**: Bug fix branches
   - Format: `bugfix/fix-login-validation`
   - Created from `master`
   - Deleted after merge

4. **`hotfix/<name>`**: Urgent production fix branches
   - Format: `hotfix/fix-critical-security-issue`
   - Created from `master`
   - Direct merge to `master` after quick review

5. **`refactor/<name>`**: Code refactoring branches
   - Format: `refactor/improve-service-layer`
   - Should not include new features

## 🔄 Complete Workflow

### 1. Create New Branch

```bash
# Ensure you're on master and up to date
git checkout master
git pull origin master

# Create and switch to new branch
git checkout -b feature/add-user-registration
```

### 2. Development

While developing:
- Make frequent, atomic commits
- Follow [commit conventions](./commit-conventions.md)
- Run tests locally before each commit

```bash
# Develop your feature
# Make frequent commits
git add .
git commit -m "feat(auth): add user registration endpoint

- Add UserRegistrationRequest DTO with validations
- Create RegisterUserService with email validation
- Add RegisterUserController with proper HTTP codes
- Include unit and integration tests"
```

### 3. Before Push: Mandatory Validations

**CRITICAL**: Before pushing, ALWAYS execute:

```bash
# 1. Run all tests
./mvnw test

# 2. Verify build works
./mvnw clean install

# 3. Verify code quality (if configured)
./mvnw checkstyle:check
./mvnw spotbugs:check

# 4. If everything passes, push
git push -u origin feature/add-user-registration
```

### 4. Create Pull Request

Once your branch is ready:

```bash
# Option 1: Using GitHub CLI (recommended)
gh pr create --title "feat: Add user registration" --body "$(cat <<'EOF'
## Summary
- Implemented user registration endpoint
- Added email validation and duplicate check
- Created comprehensive tests

## Changes
- New endpoint: POST /api/v1/auth/register
- New DTOs: UserRegistrationRequest, UserRegistrationResponse
- New service: RegisterUserService
- Added 15 unit tests and 3 integration tests

## Test Plan
- [x] Unit tests pass (100% coverage)
- [x] Integration tests pass
- [x] Manual testing with Postman
- [x] Tested duplicate email scenario
- [x] Tested validation rules

## Breaking Changes
None

## Related Issues
Closes #123
EOF
)"

# Option 2: Create PR from GitHub web interface
# Go to https://github.com/username/repo/pulls and create new PR
```

### 5. Review Process

The reviewer must verify:

**Code**:
- [ ] Follows project best practices
- [ ] No duplicated code
- [ ] No hardcoded secrets
- [ ] Appropriate logging
- [ ] Correct error handling

**Tests**:
- [ ] Tests pass in CI/CD
- [ ] Minimum 80% coverage
- [ ] Tests are meaningful (not just for coverage)

**Security**:
- [ ] No vulnerabilities introduced
- [ ] Input validations are correct
- [ ] Authorization implemented where required

**Documentation**:
- [ ] OpenAPI/Swagger updated
- [ ] README updated if necessary
- [ ] Comments on complex code

### 6. CI/CD Checks (GitHub Actions)

Your PR must pass all automatic checks:

```yaml
# .github/workflows/ci.yml (example)
name: CI

on:
  pull_request:
    branches: [ master, main ]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up JDK 17
        uses: actions/setup-java@v3
        with:
          java-version: '17'
      - name: Run tests
        run: ./mvnw test
      - name: Build
        run: ./mvnw clean install
      - name: Code quality
        run: ./mvnw checkstyle:check
```

### 7. Merge to Master

Once PR is approved:

```bash
# Option 1: Squash and merge (recommended for feature branches)
# - Keeps history clean
# - Use from GitHub interface or:
gh pr merge --squash --delete-branch

# Option 2: Merge commit (to maintain detailed history)
gh pr merge --merge --delete-branch

# Option 3: Rebase and merge (linear history)
gh pr merge --rebase --delete-branch
```

**Recommendation**: Use **Squash and merge** to maintain clean history in master.

### 8. After Merge

```bash
# Update your local master
git checkout master
git pull origin master

# Delete local branch (if not automatically deleted)
git branch -d feature/add-user-registration

# Verify remote branch was deleted
git remote prune origin
```

## 🚨 Pre-Push Validations

### Pre-Push Hook Script (Optional)

Create `.git/hooks/pre-push`:

```bash
#!/bin/bash

echo "🧪 Running tests before push..."
./mvnw test

if [ $? -ne 0 ]; then
    echo "❌ Tests failed! Push aborted."
    exit 1
fi

echo "🏗️ Building project..."
./mvnw clean install -DskipTests

if [ $? -ne 0 ]; then
    echo "❌ Build failed! Push aborted."
    exit 1
fi

echo "✅ All checks passed! Proceeding with push..."
exit 0
```

```bash
chmod +x .git/hooks/pre-push
```

## 📋 Checklist Before Creating PR

### Code
- [ ] All tests pass locally (`./mvnw test`)
- [ ] Successful build (`./mvnw clean install`)
- [ ] No compilation warnings
- [ ] Code style verified (`./mvnw checkstyle:check`)
- [ ] No unnecessary commented code
- [ ] No `System.out.println()` or debug logs
- [ ] No critical pending TODOs

### Tests
- [ ] Unit tests for new functionality
- [ ] Integration tests if necessary
- [ ] Minimum 80% coverage
- [ ] Tests properly named (`shouldXxxWhenYyy`)

### Security
- [ ] No hardcoded secrets
- [ ] No `.env` files in commit
- [ ] Input validations implemented
- [ ] `@PreAuthorize` on protected endpoints

### Documentation
- [ ] OpenAPI/Swagger updated
- [ ] README updated if configuration changes
- [ ] Comments on complex code

### Database
- [ ] Flyway migrations created if schema changes
- [ ] Migrations tested in local environment
- [ ] Don't use `ddl-auto=update` in production

## 🔐 Branch Protection Rules

Configure in GitHub Settings → Branches → Branch protection rules for `master`:

- [x] Require a pull request before merging
  - [x] Require approvals: 1 (or more depending on team)
  - [x] Dismiss stale pull request approvals when new commits are pushed
- [x] Require status checks to pass before merging
  - [x] Require branches to be up to date before merging
  - [x] Status checks: CI/CD pipeline
- [x] Require conversation resolution before merging
- [x] Do not allow bypassing the above settings

## 🎯 Best Practices

### DO ✅
- Create branches from updated `master`
- Run tests before each push
- Make atomic, descriptive commits
- Keep PRs small and focused (< 400 lines)
- Respond to review comments quickly
- Delete branches after merge
- Update your branch if master advances during development
- Write detailed PR descriptions

### DON'T ❌
- Push without running tests
- Make direct commits to `master`
- Create giant PRs (> 1000 lines)
- Ignore review comments
- Leave branches without deleting after merge
- Merge your own PR without review
- Include unrelated changes in the same PR
- Make commits with generic messages ("fix", "changes", etc.)

## 📊 Quality Metrics

Maintain these metrics in your team:

- **Test Coverage**: Minimum 80%
- **PR Size**: Ideal < 400 lines of code
- **Time to Review**: < 24 hours
- **Time to Merge**: < 48 hours after approval
- **Build Success Rate**: > 95%

## 🚀 GitHub Actions CI/CD

### Complete Pipeline Example

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ master, main ]
  pull_request:
    branches: [ master, main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v3

    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'

    - name: Cache Maven dependencies
      uses: actions/cache@v3
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        restore-keys: ${{ runner.os }}-m2

    - name: Run tests
      run: ./mvnw test

    - name: Build application
      run: ./mvnw clean install -DskipTests

    - name: Run code quality checks
      run: |
        ./mvnw checkstyle:check
        ./mvnw spotbugs:check

    - name: Generate test coverage report
      run: ./mvnw jacoco:report

    - name: Upload coverage to Codecov
      uses: codecov/codecov-action@v3
      with:
        file: ./target/site/jacoco/jacoco.xml

    - name: Build Docker image (only on master)
      if: github.ref == 'refs/heads/master'
      run: docker build -t myapp:${{ github.sha }} .
```

## 📞 Useful Commands

```bash
# View branch status
git branch -a

# View remote branches
git remote show origin

# Clean deleted remote branches
git remote prune origin

# View differences before commit
git diff

# View differences between branches
git diff master..feature/my-branch

# View log with graph
git log --oneline --graph --all

# Undo last commit (keeps changes)
git reset --soft HEAD~1

# Update branch with master changes
git checkout feature/my-branch
git rebase master

# Create PR from CLI
gh pr create

# View PR status
gh pr list

# View PR checks
gh pr checks

# Merge PR from CLI
gh pr merge 123 --squash --delete-branch
```

## Related Documentation
- [Commit Conventions](./commit-conventions.md) - How to write good commit messages
- [PR Template](./pr-template.md) - Pull Request template
- [Best Practices](../best-practices.md) - General project best practices
