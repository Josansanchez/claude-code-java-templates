# Application Layers

## Overview

Angular applications follow a layered architecture that separates concerns and promotes maintainability. This document defines the standard layers for Angular 17+ applications using standalone components.

## Layer Architecture

```
┌─────────────────────────────────────────────────────┐
│          PRESENTATION LAYER (Components)            │
│  - Smart Components (Containers)                    │
│  - Presentational Components (Dumb)                 │
│  - Layout Components                                │
└─────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────┐
│        STATE MANAGEMENT LAYER                       │
│  - Signals (signal, computed, effect)               │
│  - Services with State                              │
│  - NgRx Store (optional, for complex state)         │
└─────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────┐
│        BUSINESS LOGIC LAYER (Services)              │
│  - Feature Services                                 │
│  - Utility Services                                 │
│  - Validators                                       │
└─────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────┐
│        DATA ACCESS LAYER (API Services)             │
│  - HTTP Services                                    │
│  - API Clients                                      │
│  - Data Transformation (DTOs)                       │
└─────────────────────────────────────────────────────┘
                         ↓
┌─────────────────────────────────────────────────────┐
│              CORE LAYER                             │
│  - Guards                                           │
│  - Interceptors                                     │
│  - Error Handlers                                   │
│  - Constants & Enums                                │
└─────────────────────────────────────────────────────┘
```

## 1. Presentation Layer (Components)

### Purpose
Display UI, handle user interactions, and delegate logic to services.

### Responsibilities
- Render HTML templates
- Handle user input
- Display data from services
- Emit events to parent components
- **NO business logic**
- **NO direct HTTP calls**

### Types of Components

#### Smart Components (Containers)

Also called "container components" - they manage data and state.

**Characteristics:**
- Connected to services and state
- Handle data fetching
- Manage local component state
- Pass data to presentational components
- Located in `features/` directory

**Example:**
```typescript
// features/user/components/user-list/user-list.component.ts
@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule, UserCardComponent],
  templateUrl: './user-list.component.html',
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserListComponent implements OnInit {
  private readonly userService = inject(UserService);

  // State
  readonly users = signal<User[]>([]);
  readonly isLoading = signal(false);
  readonly error = signal<string | null>(null);

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.isLoading.set(true);
    this.userService.getUsers().subscribe({
      next: (users) => {
        this.users.set(users);
        this.isLoading.set(false);
      },
      error: (err) => {
        this.error.set('Failed to load users');
        this.isLoading.set(false);
      }
    });
  }

  deleteUser(id: string): void {
    this.userService.deleteUser(id).subscribe({
      next: () => this.loadUsers(),
      error: (err) => this.error.set('Failed to delete user')
    });
  }
}
```

**Template:**
```html
<div class="user-list">
  @if (isLoading()) {
    <app-loader />
  } @else if (error()) {
    <app-error-message [message]="error()" />
  } @else {
    @for (user of users(); track user.id) {
      <app-user-card
        [user]="user"
        (delete)="deleteUser($event)" />
    }
  }
</div>
```

#### Presentational Components (Dumb)

Also called "dumb components" - they only display data.

**Characteristics:**
- Receive data via @Input()
- Emit events via @Output()
- No service dependencies
- Highly reusable
- Located in `shared/components/` directory

**Example:**
```typescript
// shared/components/user-card/user-card.component.ts
@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './user-card.component.html',
  styleUrls: ['./user-card.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserCardComponent {
  @Input({ required: true }) user!: User;
  @Output() delete = new EventEmitter<string>();

  onDelete(): void {
    this.delete.emit(this.user.id);
  }
}
```

**Template:**
```html
<div class="user-card">
  <h3>{{ user.name }}</h3>
  <p>{{ user.email }}</p>
  <button (click)="onDelete()">Delete</button>
</div>
```

#### Layout Components

Components that define the structure of the application.

**Example:**
```typescript
// layout/main-layout/main-layout.component.ts
@Component({
  selector: 'app-main-layout',
  standalone: true,
  imports: [CommonModule, RouterOutlet, HeaderComponent, FooterComponent],
  template: `
    <div class="main-layout">
      <app-header />
      <main class="content">
        <router-outlet />
      </main>
      <app-footer />
    </div>
  `
})
export class MainLayoutComponent { }
```

### Best Practices

✅ **DO:**
- Keep components under 200 lines
- Use OnPush change detection
- Use signals for reactive state
- Separate smart and presentational components
- Use trackBy with ngFor

❌ **DON'T:**
- Put business logic in components
- Make HTTP calls directly in components
- Create large monolithic components
- Use component inheritance (prefer composition)

---

## 2. State Management Layer

### Purpose
Manage application state reactively.

### Strategies

#### Strategy 1: Signals (Recommended for Simple State)

**Use for:** Local component state, simple shared state

```typescript
// features/user/services/user-state.service.ts
@Injectable({
  providedIn: 'root'
})
export class UserStateService {
  private readonly userApiService = inject(UserApiService);

  // Private writable signals
  private readonly _users = signal<User[]>([]);
  private readonly _selectedUser = signal<User | null>(null);
  private readonly _isLoading = signal(false);

  // Public readonly signals
  readonly users = this._users.asReadonly();
  readonly selectedUser = this._selectedUser.asReadonly();
  readonly isLoading = this._isLoading.asReadonly();

  // Computed signals
  readonly userCount = computed(() => this._users().length);
  readonly hasUsers = computed(() => this._users().length > 0);

  loadUsers(): void {
    this._isLoading.set(true);
    this.userApiService.getUsers().subscribe({
      next: (users) => {
        this._users.set(users);
        this._isLoading.set(false);
      },
      error: () => this._isLoading.set(false)
    });
  }

  selectUser(user: User): void {
    this._selectedUser.set(user);
  }

  addUser(user: User): void {
    this._users.update(users => [...users, user]);
  }

  updateUser(updatedUser: User): void {
    this._users.update(users =>
      users.map(u => u.id === updatedUser.id ? updatedUser : u)
    );
  }

  deleteUser(id: string): void {
    this._users.update(users => users.filter(u => u.id !== id));
  }
}
```

**Using in component:**
```typescript
@Component({
  selector: 'app-user-dashboard',
  standalone: true,
  template: `
    <div>
      <p>Total users: {{ userState.userCount() }}</p>

      @if (userState.isLoading()) {
        <app-loader />
      } @else {
        @for (user of userState.users(); track user.id) {
          <app-user-card [user]="user" />
        }
      }
    </div>
  `
})
export class UserDashboardComponent implements OnInit {
  readonly userState = inject(UserStateService);

  ngOnInit(): void {
    this.userState.loadUsers();
  }
}
```

#### Strategy 2: Services with BehaviorSubject (Legacy/RxJS)

**Use for:** When you need RxJS operators

```typescript
@Injectable({
  providedIn: 'root'
})
export class UserStateService {
  private readonly users$ = new BehaviorSubject<User[]>([]);
  private readonly isLoading$ = new BehaviorSubject<boolean>(false);

  readonly users = this.users$.asObservable();
  readonly isLoading = this.isLoading$.asObservable();

  readonly userCount = this.users$.pipe(
    map(users => users.length)
  );

  loadUsers(): void {
    this.isLoading$.next(true);
    this.userApiService.getUsers().subscribe({
      next: (users) => {
        this.users$.next(users);
        this.isLoading$.next(false);
      }
    });
  }
}
```

#### Strategy 3: NgRx (For Complex State)

**Use for:** Large applications with complex state management needs

```typescript
// features/user/state/user.actions.ts
export const UserActions = createActionGroup({
  source: 'User',
  events: {
    'Load Users': emptyProps(),
    'Load Users Success': props<{ users: User[] }>(),
    'Load Users Failure': props<{ error: string }>(),
    'Add User': props<{ user: User }>(),
    'Delete User': props<{ id: string }>()
  }
});

// features/user/state/user.reducer.ts
export interface UserState {
  users: User[];
  selectedUser: User | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: UserState = {
  users: [],
  selectedUser: null,
  isLoading: false,
  error: null
};

export const userReducer = createReducer(
  initialState,
  on(UserActions.loadUsers, (state) => ({
    ...state,
    isLoading: true,
    error: null
  })),
  on(UserActions.loadUsersSuccess, (state, { users }) => ({
    ...state,
    users,
    isLoading: false
  })),
  on(UserActions.loadUsersFailure, (state, { error }) => ({
    ...state,
    error,
    isLoading: false
  }))
);

// features/user/state/user.selectors.ts
export const selectUserState = createFeatureSelector<UserState>('users');

export const selectAllUsers = createSelector(
  selectUserState,
  (state) => state.users
);

export const selectIsLoading = createSelector(
  selectUserState,
  (state) => state.isLoading
);

export const selectUserCount = createSelector(
  selectAllUsers,
  (users) => users.length
);
```

### Best Practices

✅ **DO:**
- Use signals for simple state
- Use computed() for derived values
- Use effect() for side effects
- Make state immutable
- Use NgRx only when complexity justifies it

❌ **DON'T:**
- Mutate state directly
- Use NgRx for simple applications
- Share mutable objects
- Forget to unsubscribe from observables

---

## 3. Business Logic Layer (Services)

### Purpose
Implement business rules and orchestrate data flow.

### Responsibilities
- Business validations
- Data transformations
- Orchestrate multiple API calls
- Implement business workflows
- **NO direct HTTP calls** (delegate to API services)
- **NO UI logic**

### Example

```typescript
// features/user/services/user.service.ts
@Injectable({
  providedIn: 'root'
})
export class UserService {
  private readonly userApiService = inject(UserApiService);
  private readonly authService = inject(AuthService);
  private readonly notificationService = inject(NotificationService);

  /**
   * Creates a new user with validation and notifications
   */
  createUser(userData: CreateUserDto): Observable<User> {
    // Business validation
    if (!this.canCreateUser()) {
      return throwError(() => new Error('Insufficient permissions'));
    }

    // Business logic: transform data
    const normalizedData = this.normalizeUserData(userData);

    // Delegate HTTP call to API service
    return this.userApiService.createUser(normalizedData).pipe(
      tap(user => {
        // Side effect: show notification
        this.notificationService.success(`User ${user.name} created`);
      }),
      catchError(error => {
        this.notificationService.error('Failed to create user');
        return throwError(() => error);
      })
    );
  }

  /**
   * Business rule: check if current user can create users
   */
  private canCreateUser(): boolean {
    const currentUser = this.authService.currentUser();
    return currentUser?.role === 'admin';
  }

  /**
   * Business logic: normalize user data
   */
  private normalizeUserData(userData: CreateUserDto): CreateUserDto {
    return {
      ...userData,
      email: userData.email.toLowerCase().trim(),
      name: userData.name.trim()
    };
  }

  /**
   * Complex business workflow: activate user
   */
  activateUser(userId: string): Observable<User> {
    return this.userApiService.getUser(userId).pipe(
      switchMap(user => {
        if (user.status === 'active') {
          throw new Error('User already active');
        }
        return this.userApiService.updateUser(userId, { status: 'active' });
      }),
      switchMap(user => {
        // Send activation email
        return this.userApiService.sendActivationEmail(user.id).pipe(
          map(() => user)
        );
      }),
      tap(user => {
        this.notificationService.success(`User ${user.name} activated`);
      })
    );
  }
}
```

### Best Practices

✅ **DO:**
- One service per feature
- Use descriptive method names
- Document business rules
- Return observables for async operations
- Handle errors appropriately
- Use dependency injection

❌ **DON'T:**
- Make HTTP calls directly (use API services)
- Put UI logic in services
- Create god services (too many responsibilities)
- Use field injection (use constructor)

---

## 4. Data Access Layer (API Services)

### Purpose
Handle HTTP communication with backend APIs.

### Responsibilities
- Make HTTP requests
- Transform API responses to domain models
- Handle API-specific errors
- **NO business logic**
- **NO state management**

### Example

```typescript
// features/user/services/user-api.service.ts
@Injectable({
  providedIn: 'root'
})
export class UserApiService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = inject(API_BASE_URL); // '/api/users'

  /**
   * GET /api/users
   */
  getUsers(): Observable<User[]> {
    return this.http.get<UserResponse[]>(`${this.baseUrl}`).pipe(
      map(response => response.map(this.mapToUser)),
      catchError(this.handleError)
    );
  }

  /**
   * GET /api/users/:id
   */
  getUser(id: string): Observable<User> {
    return this.http.get<UserResponse>(`${this.baseUrl}/${id}`).pipe(
      map(this.mapToUser),
      catchError(this.handleError)
    );
  }

  /**
   * POST /api/users
   */
  createUser(data: CreateUserDto): Observable<User> {
    return this.http.post<UserResponse>(`${this.baseUrl}`, data).pipe(
      map(this.mapToUser),
      catchError(this.handleError)
    );
  }

  /**
   * PUT /api/users/:id
   */
  updateUser(id: string, data: UpdateUserDto): Observable<User> {
    return this.http.put<UserResponse>(`${this.baseUrl}/${id}`, data).pipe(
      map(this.mapToUser),
      catchError(this.handleError)
    );
  }

  /**
   * DELETE /api/users/:id
   */
  deleteUser(id: string): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/${id}`).pipe(
      catchError(this.handleError)
    );
  }

  /**
   * Transform API response to domain model
   */
  private mapToUser(response: UserResponse): User {
    return {
      id: response.id,
      name: response.name,
      email: response.email,
      status: response.status,
      createdAt: new Date(response.created_at), // Transform snake_case to camelCase
      updatedAt: new Date(response.updated_at)
    };
  }

  /**
   * Handle HTTP errors
   */
  private handleError(error: HttpErrorResponse): Observable<never> {
    let errorMessage = 'An error occurred';

    if (error.error instanceof ErrorEvent) {
      // Client-side error
      errorMessage = error.error.message;
    } else {
      // Server-side error
      errorMessage = `Error Code: ${error.status}\nMessage: ${error.message}`;
    }

    console.error(errorMessage);
    return throwError(() => new Error(errorMessage));
  }
}
```

### DTOs (Data Transfer Objects)

```typescript
// features/user/models/user-response.model.ts
export interface UserResponse {
  id: string;
  name: string;
  email: string;
  status: string;
  created_at: string;
  updated_at: string;
}

// features/user/models/create-user-dto.model.ts
export interface CreateUserDto {
  name: string;
  email: string;
  password: string;
}

// features/user/models/update-user-dto.model.ts
export interface UpdateUserDto {
  name?: string;
  email?: string;
  status?: string;
}

// features/user/models/user.model.ts (Domain model)
export interface User {
  id: string;
  name: string;
  email: string;
  status: string;
  createdAt: Date;
  updatedAt: Date;
}
```

### Best Practices

✅ **DO:**
- Suffix with `ApiService` or `-api.service.ts`
- Use DTOs for request/response
- Transform API data to domain models
- Handle errors consistently
- Use typed responses
- Centralize base URLs

❌ **DON'T:**
- Put business logic in API services
- Use API services directly in components (use business services)
- Expose raw HTTP responses
- Hardcode URLs

---

## 5. Core Layer

### Purpose
Provide app-wide utilities, guards, interceptors, and constants.

### Components

#### Guards

```typescript
// core/guards/auth.guard.ts
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};
```

#### Interceptors

```typescript
// core/interceptors/auth.interceptor.ts
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    req = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
  }

  return next(req);
};
```

#### Error Handler

```typescript
// core/services/global-error-handler.service.ts
@Injectable()
export class GlobalErrorHandler implements ErrorHandler {
  private readonly notificationService = inject(NotificationService);

  handleError(error: Error): void {
    console.error('Global error:', error);
    this.notificationService.error('An unexpected error occurred');
  }
}
```

#### Constants

```typescript
// core/constants/api.constants.ts
export const API_BASE_URL = new InjectionToken<string>('API_BASE_URL');
export const DEFAULT_PAGE_SIZE = 20;
export const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB
```

### Best Practices

✅ **DO:**
- Use functional guards and interceptors
- Centralize error handling
- Use InjectionToken for configuration
- Keep utilities pure and testable

❌ **DON'T:**
- Put feature-specific code in core
- Create circular dependencies
- Overuse core layer

---

## Layer Communication Rules

### ✅ Allowed Dependencies

```
Presentation Layer → State Management Layer
Presentation Layer → Business Logic Layer
State Management Layer → Business Logic Layer
Business Logic Layer → Data Access Layer
All Layers → Core Layer
```

### ❌ Forbidden Dependencies

```
Data Access Layer → Business Logic Layer ❌
Data Access Layer → Presentation Layer ❌
Core Layer → Feature Layers ❌
Business Logic Layer → Presentation Layer ❌
```

### Dependency Flow

```
Component
  ↓
Service (Business Logic)
  ↓
API Service (Data Access)
  ↓
HTTP Client
  ↓
Backend API
```

---

## Complete Example: User Feature

### Directory Structure

```
features/user/
├── components/
│   ├── user-list/
│   │   ├── user-list.component.ts       # Smart component
│   │   ├── user-list.component.html
│   │   └── user-list.component.spec.ts
│   ├── user-detail/
│   └── user-form/
├── services/
│   ├── user.service.ts                   # Business logic
│   ├── user-api.service.ts               # Data access
│   └── user-state.service.ts             # State management
├── models/
│   ├── user.model.ts                     # Domain model
│   ├── user-response.model.ts            # API response DTO
│   ├── create-user-dto.model.ts          # Request DTO
│   └── update-user-dto.model.ts          # Request DTO
└── user.routes.ts
```

### Flow Example: Creating a User

```typescript
// 1. PRESENTATION LAYER
@Component({ ... })
export class UserFormComponent {
  private readonly userService = inject(UserService);

  onSubmit(formData: CreateUserDto): void {
    this.userService.createUser(formData).subscribe({
      next: (user) => console.log('User created:', user),
      error: (err) => console.error('Error:', err)
    });
  }
}

// 2. BUSINESS LOGIC LAYER
@Injectable({ providedIn: 'root' })
export class UserService {
  private readonly userApiService = inject(UserApiService);
  private readonly notificationService = inject(NotificationService);

  createUser(data: CreateUserDto): Observable<User> {
    // Business validation
    const normalizedData = this.normalizeData(data);

    // Delegate to API service
    return this.userApiService.createUser(normalizedData).pipe(
      tap(user => this.notificationService.success('User created'))
    );
  }
}

// 3. DATA ACCESS LAYER
@Injectable({ providedIn: 'root' })
export class UserApiService {
  private readonly http = inject(HttpClient);

  createUser(data: CreateUserDto): Observable<User> {
    return this.http.post<UserResponse>('/api/users', data).pipe(
      map(response => this.mapToUser(response))
    );
  }
}
```

---

## Summary Table

| Layer | Location | Responsibilities | Can Depend On |
|-------|----------|------------------|---------------|
| **Presentation** | `features/*/components/`, `shared/components/` | Display UI, handle events | State, Business, Core |
| **State Management** | `features/*/services/*-state.service.ts` | Manage state reactively | Business, Core |
| **Business Logic** | `features/*/services/*.service.ts` | Business rules, validation | Data Access, Core |
| **Data Access** | `features/*/services/*-api.service.ts` | HTTP calls, data transformation | Core only |
| **Core** | `core/` | Guards, interceptors, utilities | Nothing (no dependencies) |

---

## Related Documentation

- See [project-structure.md](./project-structure.md) for directory organization
- See [naming-conventions.md](./naming-conventions.md) for naming rules
- See [../frontend/components.md](../frontend/components.md) for component patterns
- See [../frontend/services.md](../frontend/services.md) for service patterns
- See [../state-management/signals.md](../state-management/signals.md) for state management
