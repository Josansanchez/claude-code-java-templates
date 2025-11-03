# Project Structure

## Overview

Angular 17+ projects use a feature-based structure with standalone components. This guide defines the recommended directory organization for scalable applications.

## Root Structure

```
my-angular-app/
├── src/
│   ├── app/
│   ├── assets/
│   ├── environments/
│   └── styles/
├── public/                   # Static assets (Angular 17+)
├── .vscode/
├── node_modules/
├── angular.json
├── package.json
├── tsconfig.json
├── README.md
└── .gitignore
```

## App Directory Structure

### Recommended Organization (Feature-Based)

```
src/app/
├── core/                        # Singleton services, app-wide singletons
│   ├── guards/
│   │   ├── auth.guard.ts
│   │   └── role.guard.ts
│   ├── interceptors/
│   │   ├── auth.interceptor.ts
│   │   ├── error.interceptor.ts
│   │   └── loading.interceptor.ts
│   ├── services/
│   │   ├── auth.service.ts
│   │   ├── storage.service.ts
│   │   └── logger.service.ts
│   └── models/
│       ├── user.model.ts
│       └── api-response.model.ts
│
├── shared/                      # Reusable components, directives, pipes
│   ├── components/
│   │   ├── button/
│   │   │   ├── button.component.ts
│   │   │   ├── button.component.html
│   │   │   ├── button.component.scss
│   │   │   └── button.component.spec.ts
│   │   ├── modal/
│   │   ├── card/
│   │   └── loader/
│   ├── directives/
│   │   ├── highlight.directive.ts
│   │   └── tooltip.directive.ts
│   ├── pipes/
│   │   ├── truncate.pipe.ts
│   │   └── date-format.pipe.ts
│   └── utils/
│       ├── validators.ts
│       └── helpers.ts
│
├── features/                    # Feature modules/components
│   ├── auth/
│   │   ├── components/
│   │   │   ├── login/
│   │   │   │   ├── login.component.ts
│   │   │   │   ├── login.component.html
│   │   │   │   ├── login.component.scss
│   │   │   │   └── login.component.spec.ts
│   │   │   ├── register/
│   │   │   └── forgot-password/
│   │   ├── services/
│   │   │   └── auth-api.service.ts
│   │   ├── models/
│   │   │   └── login-request.model.ts
│   │   └── auth.routes.ts       # Feature routes
│   │
│   ├── user/
│   │   ├── components/
│   │   │   ├── user-list/
│   │   │   ├── user-detail/
│   │   │   └── user-form/
│   │   ├── services/
│   │   │   ├── user.service.ts
│   │   │   └── user-api.service.ts
│   │   ├── state/              # Feature state (if using NgRx)
│   │   │   ├── user.actions.ts
│   │   │   ├── user.reducer.ts
│   │   │   ├── user.effects.ts
│   │   │   └── user.selectors.ts
│   │   └── user.routes.ts
│   │
│   ├── product/
│   │   ├── components/
│   │   ├── services/
│   │   └── product.routes.ts
│   │
│   └── dashboard/
│       ├── components/
│       ├── services/
│       └── dashboard.routes.ts
│
├── layout/                      # Layout components
│   ├── header/
│   │   ├── header.component.ts
│   │   ├── header.component.html
│   │   └── header.component.scss
│   ├── footer/
│   ├── sidebar/
│   └── main-layout/
│
├── app.component.ts             # Root component
├── app.component.html
├── app.component.scss
├── app.config.ts                # Application configuration
└── app.routes.ts                # Root routes
```

## Directory Conventions

### 1. Core Directory

**Purpose**: Singleton services used throughout the application

**Rules**:
- Services should be application-wide singletons
- Guards, interceptors, and core models
- Provided in root by default
- **Should NOT** contain components

```typescript
// core/services/auth.service.ts
@Injectable({
  providedIn: 'root'  // Singleton
})
export class AuthService {
  // Application-wide authentication logic
}
```

### 2. Shared Directory

**Purpose**: Reusable UI components, directives, and pipes

**Rules**:
- All components should be standalone
- Should NOT have dependencies on features
- Should be pure presentational components
- Can be used across multiple features

```typescript
// shared/components/button/button.component.ts
@Component({
  selector: 'app-button',
  standalone: true,
  // ...
})
export class ButtonComponent {
  // Reusable button component
}
```

### 3. Features Directory

**Purpose**: Feature-specific components and logic

**Rules**:
- One directory per feature
- Each feature is self-contained
- Has its own routing file (`.routes.ts`)
- Can have feature-specific services

**Feature Structure**:
```
features/user/
├── components/          # Feature components
├── services/            # Feature-specific services
├── models/              # Feature-specific models
├── state/               # Feature state (NgRx)
└── user.routes.ts       # Feature routes
```

### 4. Layout Directory

**Purpose**: Application layout components (header, footer, sidebar)

**Rules**:
- Layout components that wrap feature content
- Usually standalone components
- Minimal business logic

## File Naming Conventions

### Components

```
✅ CORRECT:
user-list.component.ts
user-list.component.html
user-list.component.scss
user-list.component.spec.ts

❌ WRONG:
UserList.component.ts
userList.component.ts
user_list.component.ts
```

### Services

```
✅ CORRECT:
user.service.ts
user.service.spec.ts
user-api.service.ts

❌ WRONG:
UserService.ts
userService.ts
user_service.ts
```

### Models/Interfaces

```
✅ CORRECT:
user.model.ts
user-response.model.ts
user.interface.ts

❌ WRONG:
User.model.ts
userModel.ts
```

### Guards

```
✅ CORRECT:
auth.guard.ts
role.guard.ts

❌ WRONG:
AuthGuard.ts
auth_guard.ts
```

## Routing Structure

### Root Routes (app.routes.ts)

```typescript
export const routes: Routes = [
  {
    path: '',
    redirectTo: '/dashboard',
    pathMatch: 'full'
  },
  {
    path: 'auth',
    loadChildren: () => import('./features/auth/auth.routes')
      .then(m => m.AUTH_ROUTES)
  },
  {
    path: 'users',
    loadChildren: () => import('./features/user/user.routes')
      .then(m => m.USER_ROUTES)
  },
  {
    path: '**',
    component: NotFoundComponent
  }
];
```

### Feature Routes (features/user/user.routes.ts)

```typescript
export const USER_ROUTES: Routes = [
  {
    path: '',
    component: UserListComponent
  },
  {
    path: ':id',
    component: UserDetailComponent,
    resolve: {
      user: UserResolver
    }
  },
  {
    path: 'new',
    component: UserFormComponent,
    canActivate: [AuthGuard, RoleGuard]
  }
];
```

## Module Organization (Legacy)

If you need to use NgModules (legacy code):

```
src/app/
├── core/
│   └── core.module.ts           # Singleton module, import once
├── shared/
│   └── shared.module.ts         # Reusable module, import everywhere
└── features/
    └── user/
        └── user.module.ts       # Feature module, lazy loaded
```

## Assets Organization

```
src/assets/
├── images/
│   ├── logo.svg
│   └── icons/
├── fonts/
├── i18n/                        # Translation files
│   ├── en.json
│   └── es.json
└── data/
    └── mock-data.json
```

## Styles Organization

```
src/styles/
├── _variables.scss              # SCSS variables
├── _mixins.scss                 # SCSS mixins
├── _typography.scss             # Typography styles
├── _utilities.scss              # Utility classes
└── styles.scss                  # Main styles file
```

Or with component-level styles:

```
src/app/features/user/user-list/
├── user-list.component.ts
├── user-list.component.html
├── user-list.component.scss     # Component-specific styles
└── user-list.component.spec.ts
```

## Environment Configuration

```
src/environments/
├── environment.ts               # Development
├── environment.prod.ts          # Production
└── environment.staging.ts       # Staging
```

**Angular 17+ (using replacement in angular.json)**:
```typescript
// environment.ts
export const environment = {
  production: false,
  apiUrl: 'http://localhost:3000/api',
  enableDebug: true
};
```

## State Management Structure

### With NgRx

```
src/app/
├── store/                       # Global store
│   ├── actions/
│   ├── reducers/
│   ├── effects/
│   └── selectors/
└── features/
    └── user/
        └── state/               # Feature state
            ├── user.actions.ts
            ├── user.reducer.ts
            ├── user.effects.ts
            └── user.selectors.ts
```

### With Services (Simpler)

```
src/app/features/user/
├── services/
│   ├── user.service.ts          # Business logic + state
│   └── user-api.service.ts      # API calls
```

## Testing Structure

```
src/app/features/user/
├── user-list.component.ts
├── user-list.component.spec.ts  # Unit tests (co-located)

tests/
├── e2e/                         # E2E tests
│   ├── user/
│   │   └── user-list.e2e.ts
│   └── auth/
└── fixtures/
    └── user-data.ts
```

## Best Practices

### 1. Feature-Based Organization

✅ **GOOD**: Organize by feature
```
features/
├── user/
├── product/
└── order/
```

❌ **BAD**: Organize by type
```
app/
├── components/
├── services/
└── models/
```

### 2. Standalone Components

✅ **GOOD**: Use standalone components (Angular 17+)
```typescript
@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule]
})
```

❌ **BAD**: Use modules for everything
```typescript
@NgModule({
  declarations: [UserListComponent],
  imports: [CommonModule]
})
```

### 3. Lazy Loading

✅ **GOOD**: Lazy load feature routes
```typescript
{
  path: 'users',
  loadChildren: () => import('./features/user/user.routes')
}
```

### 4. Barrel Exports

✅ **GOOD**: Use index.ts for public API
```typescript
// features/user/index.ts
export * from './services/user.service';
export * from './models/user.model';
```

### 5. Keep Components Small

✅ **GOOD**: Small, focused components (< 200 lines)
❌ **BAD**: Large components (> 500 lines)

## Organization Checklist

✅ **Directory Structure**:
- [ ] Core directory for singletons
- [ ] Shared directory for reusables
- [ ] Features directory for features
- [ ] Layout directory for layouts

✅ **File Naming**:
- [ ] Kebab-case for all files
- [ ] Descriptive suffixes (.component, .service, .guard)
- [ ] Spec files co-located with source

✅ **Routing**:
- [ ] Root routes in app.routes.ts
- [ ] Feature routes in feature.routes.ts
- [ ] Lazy loading for features

✅ **State Management**:
- [ ] Services for simple state
- [ ] NgRx for complex state
- [ ] Signals for reactive state

## Related Documentation

- See [layers.md](./layers.md) for application layers
- See [naming-conventions.md](./naming-conventions.md) for detailed naming rules
- See [../frontend/components.md](../frontend/components.md) for component patterns
