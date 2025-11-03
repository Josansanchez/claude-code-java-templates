# Angular Project Template - Claude Code Rules

> **Comprehensive rules and best practices template for Angular 17+ projects with TypeScript**

This template defines conventions, best practices, and development standards for Angular projects using standalone components, signals, and modern TypeScript.

## 🚀 How to Use This Template

### 1. Copy the Template to Your Project
```bash
# From this repository
cp -r angular/ /path/to/your/project/.claude/

# Or clone and copy
git clone <repo-url>
cp -r <repo>/angular /path/to/your/project/.claude
```

### 2. Customize for Your Project
- Edit `README.md` to include project-specific information
- Adjust rules based on your domain's particular needs
- Update the tech stack in README.md according to your project
- Configure TypeScript and ESLint based on team preferences

### 3. Configure Git Workflow (Optional)
- Review `git-workflow/github-workflow.md` to configure your workflow
- Adjust branching rules according to your team
- Configure hooks if needed in `settings.local.json`

## 📚 Documentation Index

### 🏗️ Architecture
- [Project Structure](architecture/project-structure.md) - Directory organization and modules
- [Application Layers](architecture/layers.md) - Components → Services → State → API
- [Naming Conventions](architecture/naming-conventions.md) - File and component naming
- [Module Organization](architecture/modules.md) - Feature modules, shared modules, core

### 💻 Frontend
- [**Components**](frontend/components.md) - **Standalone components, signals, lifecycle**
- [**Services**](frontend/services.md) - **Dependency injection, HTTP, business logic**
- [**Directives & Pipes**](frontend/directives-pipes.md) - **Custom directives and pipes**
- [**Routing**](frontend/routing.md) - **Lazy loading, guards, resolvers**
- [**Forms**](frontend/forms.md) - **Reactive forms, validation, dynamic forms**
- [**HTTP & APIs**](frontend/http.md) - **HttpClient, interceptors, error handling**

### 🔄 State Management
- [**Signals**](state-management/signals.md) - **Angular signals, computed, effects**
- [**Services State**](state-management/services.md) - **Service-based state management**
- [**NgRx**](state-management/ngrx.md) - **Actions, reducers, effects, selectors**

### 🔒 Security
- [XSS Protection](security/xss-protection.md) - Sanitization, DomSanitizer
- [CSRF Protection](security/csrf.md) - Token handling
- [Authentication](security/authentication.md) - JWT, guards, interceptors
- [Authorization](security/authorization.md) - Role-based access control

### ⚠️ Error Handling
- [Global Error Handler](error-handling/global-handler.md) - ErrorHandler, logging
- [HTTP Errors](error-handling/http-errors.md) - Interceptors, retry logic

### 🧪 Testing
- [Unit Tests](testing/unit-tests.md) - Jasmine/Karma, Jest, component testing
- [E2E Tests](testing/e2e-tests.md) - Cypress, Playwright
- [Testing Conventions](testing/test-conventions.md) - Naming and structure

### ⚡ Performance
- [**Lazy Loading**](performance/lazy-loading.md) - **Module and route lazy loading**
- [**Change Detection**](performance/change-detection.md) - **OnPush, signals optimization**
- [**Build Optimization**](performance/build-optimization.md) - **Bundle size, tree shaking**

### ⚙️ Configuration
- [Environment Configuration](configuration/environments.md) - Multi-environment setup
- [TypeScript Configuration](configuration/typescript.md) - tsconfig.json best practices
- [ESLint & Prettier](configuration/linting.md) - Code quality tools

### 📖 Documentation
- [JSDoc & TSDoc](documentation/jsdoc.md) - Documentation standards
- [Component Documentation](documentation/components.md) - Storybook integration

### 🔄 Git Workflow
- [GitHub Workflow](git-workflow/github-workflow.md) - Branching, testing, PR
- [Commit Conventions](git-workflow/commit-conventions.md) - Conventional Commits
- [PR Template](git-workflow/pr-template.md) - Pull Request template

### 🚀 Deployment
- [**Build & Deploy**](deployment/build.md) - **Production builds, optimization**
- [**Docker**](deployment/docker.md) - **Nginx, multi-stage builds**
- [**CI/CD**](deployment/cicd.md) - **GitHub Actions, automated deployment**

### ✅ Quick Guides
- [**Best Practices**](best-practices.md) - **50+ essential rules**
- [Checklist for New Features](checklist.md) - Steps to implement features

## 🎯 Key Principles

### 1. Standalone Components First
Angular 17+ promotes standalone components as the default. Use modules only when necessary.

```typescript
@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule, RouterLink],
  templateUrl: './user-list.component.html',
  styleUrls: ['./user-list.component.scss']
})
export class UserListComponent { }
```

### 2. Signals for Reactivity
Leverage Angular signals for reactive state management.

```typescript
export class ProductComponent {
  readonly count = signal(0);
  readonly doubleCount = computed(() => this.count() * 2);

  increment() {
    this.count.update(c => c + 1);
  }
}
```

### 3. TypeScript Strict Mode
Always enable strict mode for type safety.

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

### 4. Reactive Forms Over Template-Driven
Use reactive forms for better testability and type safety.

```typescript
export class UserFormComponent {
  readonly form = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(3)]],
    email: ['', [Validators.required, Validators.email]]
  });

  constructor(private fb: FormBuilder) {}
}
```

### 5. OnPush Change Detection
Use OnPush strategy for better performance.

```typescript
@Component({
  selector: 'app-product',
  changeDetection: ChangeDetectionStrategy.OnPush,
  // ...
})
```

## 🛠️ Technology Stack

**Required**:
- Angular 17+
- TypeScript 5.2+
- RxJS 7+
- Node.js 18+

**Recommended**:
- Angular Signals (built-in)
- Standalone Components (default)
- ESLint + Prettier
- Jest (or Jasmine/Karma)
- Cypress (or Playwright)

**Optional**:
- NgRx (for complex state)
- Angular Material
- TailwindCSS
- Storybook

## 📦 Project Structure Overview

```
src/
├── app/
│   ├── core/                    # Singleton services, guards, interceptors
│   │   ├── guards/
│   │   ├── interceptors/
│   │   └── services/
│   ├── shared/                  # Reusable components, directives, pipes
│   │   ├── components/
│   │   ├── directives/
│   │   └── pipes/
│   ├── features/                # Feature modules/components
│   │   ├── user/
│   │   │   ├── components/
│   │   │   ├── services/
│   │   │   └── user.routes.ts
│   │   └── product/
│   └── app.config.ts            # App configuration
├── assets/
├── environments/
└── styles/
```

## 🚀 Quick Start Commands

```bash
# Create new Angular project with standalone components
ng new my-app --standalone --routing --style=scss

# Generate standalone component
ng g c features/user/user-list --standalone

# Generate service
ng g s features/user/services/user

# Run development server
ng serve

# Run tests
ng test

# Build for production
ng build --configuration production

# Run E2E tests
ng e2e
```

## 📋 Best Practices Summary

1. ✅ Use standalone components as default
2. ✅ Leverage Angular signals for state
3. ✅ Enable TypeScript strict mode
4. ✅ Use OnPush change detection
5. ✅ Implement lazy loading for routes
6. ✅ Use reactive forms
7. ✅ Write unit tests for components and services
8. ✅ Use ESLint and Prettier
9. ✅ Follow naming conventions
10. ✅ Keep components small and focused

## 🔗 Useful Resources

- [Angular Official Documentation](https://angular.dev)
- [Angular Style Guide](https://angular.dev/style-guide)
- [Angular Update Guide](https://update.angular.io)
- [RxJS Documentation](https://rxjs.dev)

## 📝 Template Version

- **Version**: 1.0.0
- **Last Updated**: 2025-11-03
- **Compatible with**: Angular 17+, TypeScript 5.2+
- **Features**:
  - Standalone components architecture
  - Angular signals integration
  - Modern TypeScript patterns
  - Comprehensive testing strategies
  - Performance optimization
  - Security best practices

## 🤝 Contributing

Improvements to this template are welcome:

1. Use the template in a real Angular project
2. Identify improvements or missing documentation
3. Update the template
4. Test thoroughly
5. Submit a pull request

---

**Maintained by**: Josansanchez
**Repository**: https://github.com/Josansanchez/claude-code-templates
