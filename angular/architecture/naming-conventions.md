# Naming Conventions

## General Rules

**Files**: kebab-case with descriptive suffixes
**Classes**: PascalCase with descriptive suffixes
**Variables/Functions**: camelCase
**Constants**: UPPER_SNAKE_CASE
**Interfaces**: PascalCase without 'I' prefix

## Components

### File Naming

```
✅ CORRECT:
user-list.component.ts
user-list.component.html
user-list.component.scss
user-list.component.spec.ts

product-card.component.ts
dashboard-widget.component.ts

❌ WRONG:
UserList.component.ts
userList.component.ts
user_list.component.ts
userListComponent.ts
```

### Class Naming

```typescript
✅ CORRECT:
export class UserListComponent { }
export class ProductCardComponent { }
export class DashboardWidgetComponent { }

❌ WRONG:
export class UserList { }
export class userListComponent { }
export class User_List_Component { }
```

### Selector Naming

```typescript
✅ CORRECT (with app prefix):
@Component({
  selector: 'app-user-list'
})

@Component({
  selector: 'app-product-card'
})

❌ WRONG:
@Component({
  selector: 'UserList'  // Not kebab-case
})

@Component({
  selector: 'user-list'  // Missing prefix
})
```

## Services

### File Naming

```
✅ CORRECT:
user.service.ts
user.service.spec.ts
user-api.service.ts
auth.service.ts
storage.service.ts

❌ WRONG:
UserService.ts
userService.ts
user_service.ts
```

### Class Naming

```typescript
✅ CORRECT:
export class UserService { }
export class UserApiService { }
export class AuthService { }

❌ WRONG:
export class User { }
export class userService { }
export class UserSrv { }
```

## Directives

### Structural Directives

```
✅ CORRECT:
app-if-role.directive.ts

@Directive({
  selector: '[appIfRole]',
  standalone: true
})
export class IfRoleDirective { }

❌ WRONG:
if-role.directive.ts (missing 'app' prefix)
AppIfRole.directive.ts (wrong casing)
```

### Attribute Directives

```
✅ CORRECT:
highlight.directive.ts

@Directive({
  selector: '[appHighlight]',
  standalone: true
})
export class HighlightDirective { }
```

## Pipes

### File Naming

```
✅ CORRECT:
truncate.pipe.ts
date-format.pipe.ts
filter.pipe.ts

❌ WRONG:
TruncatePipe.ts
truncate_pipe.ts
```

### Class Naming

```typescript
✅ CORRECT:
@Pipe({
  name: 'truncate',
  standalone: true
})
export class TruncatePipe { }

@Pipe({
  name: 'dateFormat',
  standalone: true
})
export class DateFormatPipe { }

❌ WRONG:
@Pipe({
  name: 'Truncate'  // Should be camelCase
})

@Pipe({
  name: 'truncate_pipe'  // No underscores
})
```

## Guards

### File Naming

```
✅ CORRECT:
auth.guard.ts
role.guard.ts
unsaved-changes.guard.ts

❌ WRONG:
AuthGuard.ts
auth_guard.ts
```

### Function Naming (Functional Guards)

```typescript
✅ CORRECT:
export const authGuard: CanActivateFn = () => { }
export const roleGuard: CanActivateFn = () => { }

❌ WRONG:
export const AuthGuard = () => { }
export const auth_guard = () => { }
```

## Interceptors

### File Naming

```
✅ CORRECT:
auth.interceptor.ts
error.interceptor.ts
logging.interceptor.ts

❌ WRONG:
AuthInterceptor.ts
auth_interceptor.ts
```

### Function Naming

```typescript
✅ CORRECT:
export const authInterceptor: HttpInterceptorFn = () => { }
export const errorInterceptor: HttpInterceptorFn = () => { }
```

## Models & Interfaces

### File Naming

```
✅ CORRECT:
user.model.ts
user-response.model.ts
product.interface.ts
api-response.interface.ts

❌ WRONG:
User.model.ts
userModel.ts
IUser.ts
```

### Interface/Type Naming

```typescript
✅ CORRECT:
export interface User {
  id: string;
  name: string;
}

export interface UserResponse {
  data: User;
  message: string;
}

export type UserId = string;
export type UserRole = 'admin' | 'user';

❌ WRONG:
export interface IUser { }  // Don't prefix with 'I'
export interface user { }   // Not PascalCase
export interface USER { }   // Not PascalCase
```

## Enums

### File Naming

```
✅ CORRECT:
user-role.enum.ts
order-status.enum.ts

❌ WRONG:
UserRole.enum.ts
user_role.enum.ts
```

### Enum Naming

```typescript
✅ CORRECT:
export enum UserRole {
  Admin = 'ADMIN',
  User = 'USER',
  Guest = 'GUEST'
}

export enum OrderStatus {
  Pending = 'PENDING',
  Completed = 'COMPLETED',
  Cancelled = 'CANCELLED'
}

❌ WRONG:
export enum userRole { }  // Not PascalCase
export enum USER_ROLE { }  // Not PascalCase
```

## Constants

### File Naming

```
✅ CORRECT:
app.constants.ts
api.constants.ts

❌ WRONG:
AppConstants.ts
app_constants.ts
```

### Constant Naming

```typescript
✅ CORRECT:
export const API_BASE_URL = 'https://api.example.com';
export const DEFAULT_PAGE_SIZE = 20;
export const MAX_FILE_SIZE = 5 * 1024 * 1024;

export const APP_CONFIG = {
  apiUrl: 'https://api.example.com',
  timeout: 30000
} as const;

❌ WRONG:
export const apiBaseUrl = '...';  // Not UPPER_SNAKE_CASE
export const Api_Base_Url = '...';  // Inconsistent
```

## Routes

### File Naming

```
✅ CORRECT:
app.routes.ts
user.routes.ts
auth.routes.ts

❌ WRONG:
AppRoutes.ts
app_routes.ts
routes.ts (too generic)
```

### Route Naming

```typescript
✅ CORRECT:
export const routes: Routes = [
  { path: 'users', component: UserListComponent },
  { path: 'users/:id', component: UserDetailComponent },
  { path: 'users/new', component: UserFormComponent }
];

export const USER_ROUTES: Routes = [ ];

❌ WRONG:
export const Routes = [ ];  // Too generic
export const userRoutes = [ ];  // Not UPPER_SNAKE_CASE for const
```

## Validators

### File Naming

```
✅ CORRECT:
email.validator.ts
password.validator.ts
custom-validators.ts

❌ WRONG:
EmailValidator.ts
email_validator.ts
```

### Function Naming

```typescript
✅ CORRECT:
export function emailValidator(): ValidatorFn { }
export function passwordStrengthValidator(): ValidatorFn { }
export function matchFieldsValidator(field1: string, field2: string): ValidatorFn { }

❌ WRONG:
export function EmailValidator() { }  // Not camelCase
export function email_validator() { }  // No underscores
```

## Variables and Methods

### Component/Service Properties

```typescript
✅ CORRECT:
export class UserListComponent {
  // Properties (camelCase)
  users: User[] = [];
  selectedUser: User | null = null;
  isLoading = false;

  // Signals
  readonly count = signal(0);
  readonly users = signal<User[]>([]);

  // Private properties (prefix with _)
  private _internalState = 0;

  // Methods (camelCase)
  loadUsers(): void { }
  selectUser(user: User): void { }
  private handleError(error: Error): void { }
}

❌ WRONG:
Users: User[] = [];  // Not camelCase
is_loading = false;  // No underscores
LoadUsers(): void { }  // Not camelCase
```

### Boolean Variables

```typescript
✅ CORRECT:
isLoading = false;
hasError = false;
canEdit = true;
shouldShow = true;

❌ WRONG:
loading = false;  // Not descriptive
error = false;  // Unclear
editMode = true;  // Not boolean naming
```

## Observables and Signals

### Observable Naming

```typescript
✅ CORRECT:
users$ = new BehaviorSubject<User[]>([]);
selectedUser$ = new Subject<User>();
isLoading$ = new BehaviorSubject<boolean>(false);

❌ WRONG:
users = new BehaviorSubject<User[]>([]);  // Missing $
usersObservable = new Subject<User>();  // Don't use 'Observable' suffix
```

### Signal Naming

```typescript
✅ CORRECT:
readonly count = signal(0);
readonly users = signal<User[]>([]);
readonly isLoading = signal(false);

// Computed signals
readonly doubleCount = computed(() => this.count() * 2);

❌ WRONG:
readonly countSignal = signal(0);  // Don't use 'Signal' suffix
readonly Count = signal(0);  // Not camelCase
```

## Template Variables

```html
✅ CORRECT:
<div *ngFor="let user of users; trackBy: trackByUserId">
  {{ user.name }}
</div>

<ng-container *ngIf="users$ | async as users">
  <div>{{ users.length }}</div>
</ng-container>

❌ WRONG:
<div *ngFor="let u of users">  <!-- Not descriptive -->
<div *ngFor="let User of users">  <!-- Not camelCase -->
```

## Test Files

### Naming

```
✅ CORRECT:
user-list.component.spec.ts
user.service.spec.ts
auth.guard.spec.ts

❌ WRONG:
user-list.test.ts
UserList.spec.ts
user_list.spec.ts
```

### Describe Blocks

```typescript
✅ CORRECT:
describe('UserListComponent', () => {
  describe('loadUsers', () => {
    it('should load users on init', () => { });
    it('should handle errors gracefully', () => { });
  });
});

❌ WRONG:
describe('user list component', () => { });  // Not PascalCase
describe('User List', () => { });  // Inconsistent
```

## Folder Naming

```
✅ CORRECT:
features/
  user/
  product/
  auth/

shared/
  components/
  directives/
  pipes/

❌ WRONG:
Features/
User/
Shared_Components/
```

## Summary Table

| Type | File Naming | Class/Function Naming | Example |
|------|-------------|----------------------|---------|
| Component | kebab-case.component.ts | PascalCase + Component | user-list.component.ts → UserListComponent |
| Service | kebab-case.service.ts | PascalCase + Service | user.service.ts → UserService |
| Directive | kebab-case.directive.ts | PascalCase + Directive | highlight.directive.ts → HighlightDirective |
| Pipe | kebab-case.pipe.ts | PascalCase + Pipe | truncate.pipe.ts → TruncatePipe |
| Guard | kebab-case.guard.ts | camelCase + Guard | auth.guard.ts → authGuard |
| Interceptor | kebab-case.interceptor.ts | camelCase + Interceptor | auth.interceptor.ts → authInterceptor |
| Interface | kebab-case.interface.ts | PascalCase | user.interface.ts → User |
| Enum | kebab-case.enum.ts | PascalCase | user-role.enum.ts → UserRole |
| Constant | kebab-case.constants.ts | UPPER_SNAKE_CASE | app.constants.ts → API_BASE_URL |
| Routes | kebab-case.routes.ts | UPPER_SNAKE_CASE | user.routes.ts → USER_ROUTES |

## Related Documentation

- See [project-structure.md](./project-structure.md) for directory organization
- See [../frontend/components.md](../frontend/components.md) for component patterns
- See [../best-practices.md](../best-practices.md) for general best practices
