# Services - Angular 17+

## Overview

Services in Angular handle business logic, data access, and shared state. They follow the **Single Responsibility Principle** and are injected using Angular's dependency injection system.

## Table of Contents

1. [Creating Services](#creating-services)
2. [Dependency Injection](#dependency-injection)
3. [Service Types](#service-types)
4. [HTTP Services](#http-services)
5. [State Management Services](#state-management-services)
6. [RxJS Patterns](#rxjs-patterns)
7. [Error Handling](#error-handling)
8. [Testing Services](#testing-services)
9. [Best Practices](#best-practices)

---

## Creating Services

### Basic Service

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'  // Singleton service available app-wide
})
export class UserService {
  private users: User[] = [];

  getUsers(): User[] {
    return this.users;
  }

  addUser(user: User): void {
    this.users.push(user);
  }
}
```

### CLI Command

```bash
# Generate service in features/user/services/
ng generate service features/user/services/user

# Or shorthand
ng g s features/user/services/user
```

Generated structure:
```
features/user/services/
├── user.service.ts
└── user.service.spec.ts
```

---

## Dependency Injection

### Constructor Injection (Traditional)

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  constructor(private readonly http: HttpClient) {}

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
}
```

### inject() Function (Modern - Recommended)

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  private readonly http = inject(HttpClient);
  private readonly logger = inject(LoggerService);

  getUsers(): Observable<User[]> {
    this.logger.log('Fetching users...');
    return this.http.get<User[]>('/api/users');
  }
}
```

### Injection Tokens

For configuration values:

```typescript
// core/tokens/api.token.ts
import { InjectionToken } from '@angular/core';

export const API_BASE_URL = new InjectionToken<string>('API_BASE_URL');
```

**Providing the value:**

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { API_BASE_URL } from './core/tokens/api.token';

export const appConfig: ApplicationConfig = {
  providers: [
    { provide: API_BASE_URL, useValue: 'https://api.example.com' }
  ]
};
```

**Using the token:**

```typescript
@Injectable({
  providedIn: 'root'
})
export class UserApiService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = inject(API_BASE_URL);

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>(`${this.baseUrl}/users`);
  }
}
```

### Service Scope

```typescript
// ✅ Singleton (app-wide)
@Injectable({
  providedIn: 'root'
})
export class AuthService { }

// ✅ Feature-scoped (lazy-loaded module)
@Injectable({
  providedIn: 'any'  // New instance per lazy-loaded module
})
export class FeatureService { }

// ✅ Component-scoped
@Component({
  selector: 'app-user-list',
  providers: [UserListService]  // New instance per component
})
export class UserListComponent { }
```

---

## Service Types

### 1. Business Logic Services

Handle business rules, validations, and orchestration.

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
   * Creates a user with business validation
   */
  createUser(userData: CreateUserDto): Observable<User> {
    // Business validation
    if (!this.canCreateUser()) {
      return throwError(() => new Error('Insufficient permissions'));
    }

    // Business logic: normalize data
    const normalizedData = this.normalizeUserData(userData);

    // Delegate to API service
    return this.userApiService.createUser(normalizedData).pipe(
      tap(user => {
        this.notificationService.success(`User ${user.name} created`);
      }),
      catchError(error => {
        this.notificationService.error('Failed to create user');
        return throwError(() => error);
      })
    );
  }

  /**
   * Business rule: check permissions
   */
  private canCreateUser(): boolean {
    const currentUser = this.authService.currentUser();
    return currentUser?.role === 'admin';
  }

  /**
   * Business logic: normalize user data
   */
  private normalizeUserData(data: CreateUserDto): CreateUserDto {
    return {
      ...data,
      email: data.email.toLowerCase().trim(),
      name: data.name.trim()
    };
  }

  /**
   * Complex workflow: activate user
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
        return this.userApiService.sendActivationEmail(user.id).pipe(
          map(() => user)
        );
      }),
      tap(user => {
        this.notificationService.success(`User activated: ${user.name}`);
      })
    );
  }
}
```

### 2. API Services (Data Access)

Handle HTTP communication with backend.

```typescript
// features/user/services/user-api.service.ts
@Injectable({
  providedIn: 'root'
})
export class UserApiService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = '/api/users';

  /**
   * GET /api/users
   */
  getUsers(): Observable<User[]> {
    return this.http.get<UserResponse[]>(this.baseUrl).pipe(
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
    return this.http.post<UserResponse>(this.baseUrl, data).pipe(
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
   * GET /api/users with pagination
   */
  getUsersPaginated(page: number, size: number): Observable<PaginatedResponse<User>> {
    const params = new HttpParams()
      .set('page', page.toString())
      .set('size', size.toString());

    return this.http.get<PaginatedResponse<UserResponse>>(this.baseUrl, { params }).pipe(
      map(response => ({
        ...response,
        content: response.content.map(this.mapToUser)
      })),
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
      createdAt: new Date(response.created_at),
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
      errorMessage = `Error: ${error.error.message}`;
    } else {
      // Server-side error
      errorMessage = `Error Code: ${error.status}\nMessage: ${error.message}`;
    }

    console.error(errorMessage);
    return throwError(() => new Error(errorMessage));
  }
}
```

### 3. Utility Services

Provide reusable utilities.

```typescript
// core/services/storage.service.ts
@Injectable({
  providedIn: 'root'
})
export class StorageService {
  /**
   * Save item to localStorage
   */
  setItem<T>(key: string, value: T): void {
    try {
      const serialized = JSON.stringify(value);
      localStorage.setItem(key, serialized);
    } catch (error) {
      console.error('Error saving to localStorage:', error);
    }
  }

  /**
   * Get item from localStorage
   */
  getItem<T>(key: string): T | null {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : null;
    } catch (error) {
      console.error('Error reading from localStorage:', error);
      return null;
    }
  }

  /**
   * Remove item from localStorage
   */
  removeItem(key: string): void {
    localStorage.removeItem(key);
  }

  /**
   * Clear all localStorage
   */
  clear(): void {
    localStorage.clear();
  }
}
```

---

## HTTP Services

### Basic HTTP Operations

```typescript
import { HttpClient, HttpHeaders, HttpParams } from '@angular/common/http';
import { Injectable, inject } from '@angular/core';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class ProductApiService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = '/api/products';

  // GET request
  getProducts(): Observable<Product[]> {
    return this.http.get<Product[]>(this.baseUrl);
  }

  // GET with query parameters
  searchProducts(query: string, category?: string): Observable<Product[]> {
    let params = new HttpParams().set('q', query);
    if (category) {
      params = params.set('category', category);
    }
    return this.http.get<Product[]>(`${this.baseUrl}/search`, { params });
  }

  // GET with headers
  getProduct(id: string): Observable<Product> {
    const headers = new HttpHeaders({
      'X-Custom-Header': 'value'
    });
    return this.http.get<Product>(`${this.baseUrl}/${id}`, { headers });
  }

  // POST request
  createProduct(product: CreateProductDto): Observable<Product> {
    return this.http.post<Product>(this.baseUrl, product);
  }

  // PUT request
  updateProduct(id: string, product: UpdateProductDto): Observable<Product> {
    return this.http.put<Product>(`${this.baseUrl}/${id}`, product);
  }

  // PATCH request
  partialUpdate(id: string, changes: Partial<Product>): Observable<Product> {
    return this.http.patch<Product>(`${this.baseUrl}/${id}`, changes);
  }

  // DELETE request
  deleteProduct(id: string): Observable<void> {
    return this.http.delete<void>(`${this.baseUrl}/${id}`);
  }

  // Upload file
  uploadImage(file: File): Observable<string> {
    const formData = new FormData();
    formData.append('file', file);
    return this.http.post<string>(`${this.baseUrl}/upload`, formData);
  }

  // Download file
  downloadReport(): Observable<Blob> {
    return this.http.get(`${this.baseUrl}/report`, {
      responseType: 'blob'
    });
  }
}
```

### HTTP Interceptors

```typescript
// core/interceptors/auth.interceptor.ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from '../services/auth.service';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getToken();

  if (token) {
    const cloned = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
    return next(cloned);
  }

  return next(req);
};
```

```typescript
// core/interceptors/error.interceptor.ts
import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { catchError, throwError } from 'rxjs';
import { NotificationService } from '../services/notification.service';

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
  const notificationService = inject(NotificationService);

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      let errorMessage = 'An error occurred';

      if (error.error instanceof ErrorEvent) {
        // Client-side error
        errorMessage = `Error: ${error.error.message}`;
      } else {
        // Server-side error
        switch (error.status) {
          case 400:
            errorMessage = 'Bad Request';
            break;
          case 401:
            errorMessage = 'Unauthorized';
            break;
          case 403:
            errorMessage = 'Forbidden';
            break;
          case 404:
            errorMessage = 'Not Found';
            break;
          case 500:
            errorMessage = 'Internal Server Error';
            break;
          default:
            errorMessage = `Error Code: ${error.status}`;
        }
      }

      notificationService.error(errorMessage);
      return throwError(() => new Error(errorMessage));
    })
  );
};
```

**Registering interceptors:**

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './core/interceptors/auth.interceptor';
import { errorInterceptor } from './core/interceptors/error.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor, errorInterceptor])
    )
  ]
};
```

---

## State Management Services

### Signals-Based State Service

```typescript
// features/user/services/user-state.service.ts
import { Injectable, signal, computed, inject } from '@angular/core';
import { UserApiService } from './user-api.service';

@Injectable({
  providedIn: 'root'
})
export class UserStateService {
  private readonly userApiService = inject(UserApiService);

  // Private writable signals
  private readonly _users = signal<User[]>([]);
  private readonly _selectedUser = signal<User | null>(null);
  private readonly _isLoading = signal(false);
  private readonly _error = signal<string | null>(null);

  // Public readonly signals
  readonly users = this._users.asReadonly();
  readonly selectedUser = this._selectedUser.asReadonly();
  readonly isLoading = this._isLoading.asReadonly();
  readonly error = this._error.asReadonly();

  // Computed signals
  readonly userCount = computed(() => this._users().length);
  readonly hasUsers = computed(() => this._users().length > 0);
  readonly activeUsers = computed(() =>
    this._users().filter(u => u.status === 'active')
  );

  /**
   * Load all users
   */
  loadUsers(): void {
    this._isLoading.set(true);
    this._error.set(null);

    this.userApiService.getUsers().subscribe({
      next: (users) => {
        this._users.set(users);
        this._isLoading.set(false);
      },
      error: (error) => {
        this._error.set(error.message);
        this._isLoading.set(false);
      }
    });
  }

  /**
   * Select a user
   */
  selectUser(user: User | null): void {
    this._selectedUser.set(user);
  }

  /**
   * Add a new user
   */
  addUser(user: User): void {
    this._users.update(users => [...users, user]);
  }

  /**
   * Update existing user
   */
  updateUser(updatedUser: User): void {
    this._users.update(users =>
      users.map(u => u.id === updatedUser.id ? updatedUser : u)
    );
  }

  /**
   * Delete user
   */
  deleteUser(id: string): void {
    this._users.update(users => users.filter(u => u.id !== id));
  }

  /**
   * Clear all state
   */
  clear(): void {
    this._users.set([]);
    this._selectedUser.set(null);
    this._error.set(null);
  }
}
```

### BehaviorSubject-Based State Service (Legacy/RxJS)

```typescript
// features/user/services/user-state-rxjs.service.ts
import { Injectable, inject } from '@angular/core';
import { BehaviorSubject, Observable, combineLatest } from 'rxjs';
import { map } from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class UserStateRxjsService {
  private readonly userApiService = inject(UserApiService);

  // Private subjects
  private readonly usersSubject = new BehaviorSubject<User[]>([]);
  private readonly selectedUserSubject = new BehaviorSubject<User | null>(null);
  private readonly isLoadingSubject = new BehaviorSubject<boolean>(false);

  // Public observables
  readonly users$ = this.usersSubject.asObservable();
  readonly selectedUser$ = this.selectedUserSubject.asObservable();
  readonly isLoading$ = this.isLoadingSubject.asObservable();

  // Derived observables
  readonly userCount$ = this.users$.pipe(
    map(users => users.length)
  );

  readonly activeUsers$ = this.users$.pipe(
    map(users => users.filter(u => u.status === 'active'))
  );

  loadUsers(): void {
    this.isLoadingSubject.next(true);

    this.userApiService.getUsers().subscribe({
      next: (users) => {
        this.usersSubject.next(users);
        this.isLoadingSubject.next(false);
      },
      error: () => {
        this.isLoadingSubject.next(false);
      }
    });
  }

  selectUser(user: User | null): void {
    this.selectedUserSubject.next(user);
  }

  addUser(user: User): void {
    const current = this.usersSubject.value;
    this.usersSubject.next([...current, user]);
  }

  updateUser(updatedUser: User): void {
    const current = this.usersSubject.value;
    const updated = current.map(u =>
      u.id === updatedUser.id ? updatedUser : u
    );
    this.usersSubject.next(updated);
  }

  deleteUser(id: string): void {
    const current = this.usersSubject.value;
    this.usersSubject.next(current.filter(u => u.id !== id));
  }
}
```

---

## RxJS Patterns

### Common Operators

```typescript
import { Injectable, inject } from '@angular/core';
import { Observable, of, throwError, timer, combineLatest, forkJoin } from 'rxjs';
import {
  map,
  filter,
  switchMap,
  mergeMap,
  catchError,
  retry,
  debounceTime,
  distinctUntilChanged,
  tap,
  shareReplay,
  take,
  takeUntil
} from 'rxjs/operators';

@Injectable({
  providedIn: 'root'
})
export class RxjsPatternsService {
  private readonly http = inject(HttpClient);

  /**
   * map: Transform data
   */
  getUsers(): Observable<string[]> {
    return this.http.get<User[]>('/api/users').pipe(
      map(users => users.map(u => u.name))
    );
  }

  /**
   * filter: Filter results
   */
  getActiveUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users').pipe(
      map(users => users.filter(u => u.status === 'active'))
    );
  }

  /**
   * switchMap: Switch to new observable (cancels previous)
   */
  searchUsers(query$: Observable<string>): Observable<User[]> {
    return query$.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(query => this.http.get<User[]>(`/api/users/search?q=${query}`))
    );
  }

  /**
   * mergeMap: Merge multiple observables
   */
  getUsersWithDetails(userIds: string[]): Observable<User[]> {
    return of(userIds).pipe(
      mergeMap(ids => ids.map(id => this.http.get<User>(`/api/users/${id}`))),
      // mergeAll(),
      // toArray()
    );
  }

  /**
   * catchError: Handle errors
   */
  getUserSafe(id: string): Observable<User | null> {
    return this.http.get<User>(`/api/users/${id}`).pipe(
      catchError(error => {
        console.error('Error fetching user:', error);
        return of(null);
      })
    );
  }

  /**
   * retry: Retry on failure
   */
  getUserWithRetry(id: string): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`).pipe(
      retry(3),  // Retry up to 3 times
      catchError(error => throwError(() => error))
    );
  }

  /**
   * tap: Side effects
   */
  getUserWithLogging(id: string): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`).pipe(
      tap(user => console.log('User loaded:', user)),
      tap({
        next: (user) => console.log('Success:', user),
        error: (err) => console.error('Error:', err)
      })
    );
  }

  /**
   * shareReplay: Share and cache result
   */
  private readonly config$ = this.http.get<Config>('/api/config').pipe(
    shareReplay(1)  // Cache and share with all subscribers
  );

  getConfig(): Observable<Config> {
    return this.config$;  // All subscribers get the same cached result
  }

  /**
   * combineLatest: Combine multiple observables
   */
  getDashboardData(): Observable<DashboardData> {
    return combineLatest([
      this.http.get<User[]>('/api/users'),
      this.http.get<Product[]>('/api/products'),
      this.http.get<Order[]>('/api/orders')
    ]).pipe(
      map(([users, products, orders]) => ({
        userCount: users.length,
        productCount: products.length,
        orderCount: orders.length
      }))
    );
  }

  /**
   * forkJoin: Wait for all to complete
   */
  loadAllData(): Observable<{ users: User[]; products: Product[] }> {
    return forkJoin({
      users: this.http.get<User[]>('/api/users'),
      products: this.http.get<Product[]>('/api/products')
    });
  }
}
```

---

## Error Handling

### Centralized Error Handling

```typescript
// core/services/error-handler.service.ts
import { Injectable, inject } from '@angular/core';
import { HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { NotificationService } from './notification.service';

@Injectable({
  providedIn: 'root'
})
export class ErrorHandlerService {
  private readonly notificationService = inject(NotificationService);

  handleError(error: HttpErrorResponse): Observable<never> {
    let errorMessage = 'An error occurred';

    if (error.error instanceof ErrorEvent) {
      // Client-side error
      errorMessage = `Client Error: ${error.error.message}`;
    } else {
      // Server-side error
      switch (error.status) {
        case 400:
          errorMessage = this.handle400(error);
          break;
        case 401:
          errorMessage = 'Please login to continue';
          break;
        case 403:
          errorMessage = 'You do not have permission';
          break;
        case 404:
          errorMessage = 'Resource not found';
          break;
        case 500:
          errorMessage = 'Server error occurred';
          break;
        default:
          errorMessage = `Unexpected error: ${error.status}`;
      }
    }

    this.notificationService.error(errorMessage);
    console.error('Error details:', error);

    return throwError(() => new Error(errorMessage));
  }

  private handle400(error: HttpErrorResponse): string {
    if (error.error?.errors) {
      // Validation errors
      const validationErrors = Object.values(error.error.errors).join(', ');
      return `Validation failed: ${validationErrors}`;
    }
    return 'Bad request';
  }
}
```

---

## Testing Services

### Basic Service Test

```typescript
// user.service.spec.ts
import { TestBed } from '@angular/core/testing';
import { HttpClientTestingModule, HttpTestingController } from '@angular/common/http/testing';
import { UserApiService } from './user-api.service';

describe('UserApiService', () => {
  let service: UserApiService;
  let httpMock: HttpTestingController;

  beforeEach(() => {
    TestBed.configureTestingModule({
      imports: [HttpClientTestingModule],
      providers: [UserApiService]
    });

    service = TestBed.inject(UserApiService);
    httpMock = TestBed.inject(HttpTestingController);
  });

  afterEach(() => {
    httpMock.verify();  // Verify no outstanding requests
  });

  it('should fetch users', () => {
    const mockUsers: User[] = [
      { id: '1', name: 'John', email: 'john@example.com' }
    ];

    service.getUsers().subscribe(users => {
      expect(users.length).toBe(1);
      expect(users[0].name).toBe('John');
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('GET');
    req.flush(mockUsers);
  });

  it('should create user', () => {
    const newUser = { name: 'Jane', email: 'jane@example.com' };
    const createdUser = { id: '2', ...newUser };

    service.createUser(newUser).subscribe(user => {
      expect(user.id).toBe('2');
      expect(user.name).toBe('Jane');
    });

    const req = httpMock.expectOne('/api/users');
    expect(req.request.method).toBe('POST');
    expect(req.request.body).toEqual(newUser);
    req.flush(createdUser);
  });

  it('should handle errors', () => {
    service.getUsers().subscribe({
      next: () => fail('should have failed'),
      error: (error) => {
        expect(error.message).toContain('500');
      }
    });

    const req = httpMock.expectOne('/api/users');
    req.flush('Error', { status: 500, statusText: 'Server Error' });
  });
});
```

---

## Best Practices

### ✅ DO

1. **Use `providedIn: 'root'`** for singleton services
2. **Use `inject()` function** for dependency injection (modern approach)
3. **One service per feature** (Single Responsibility)
4. **Separate API services** from business logic services
5. **Use RxJS operators** for data transformation
6. **Handle errors** consistently
7. **Return observables** for async operations
8. **Use signals** for simple state management
9. **Document public methods** with JSDoc
10. **Write unit tests** for all services

### ❌ DON'T

1. **Don't put UI logic** in services
2. **Don't use field injection** (use constructor or inject())
3. **Don't create god services** (too many responsibilities)
4. **Don't forget to unsubscribe** (use takeUntilDestroyed or async pipe)
5. **Don't hardcode URLs** (use environment variables or tokens)
6. **Don't make services stateful** unless necessary
7. **Don't expose subjects** directly (use asObservable())

### Code Examples

#### ✅ GOOD: Separation of concerns

```typescript
// API Service - only HTTP
@Injectable({ providedIn: 'root' })
export class UserApiService {
  private readonly http = inject(HttpClient);

  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
}

// Business Service - logic + orchestration
@Injectable({ providedIn: 'root' })
export class UserService {
  private readonly userApiService = inject(UserApiService);

  getUsersSorted(): Observable<User[]> {
    return this.userApiService.getUsers().pipe(
      map(users => users.sort((a, b) => a.name.localeCompare(b.name)))
    );
  }
}
```

#### ❌ BAD: Everything in one service

```typescript
// ❌ DON'T: God service
@Injectable({ providedIn: 'root' })
export class AppService {
  getUsers() { }
  getProducts() { }
  getOrders() { }
  authenticateUser() { }
  validateForm() { }
  // ... too many responsibilities
}
```

---

## Summary

| Concept | Recommendation |
|---------|---------------|
| **Injection** | Use `inject()` function (modern approach) |
| **Scope** | `providedIn: 'root'` for singletons |
| **HTTP** | Use HttpClient + interceptors |
| **State** | Signals for simple state, NgRx for complex |
| **Error Handling** | Centralized with interceptors |
| **RxJS** | Use operators for transformation |
| **Testing** | HttpClientTestingModule for HTTP tests |

---

## Related Documentation

- See [architecture/layers.md](../architecture/layers.md) for service layers
- See [http.md](./http.md) for HTTP details
- See [../testing/unit-tests.md](../testing/unit-tests.md) for testing
- See [../state-management/signals.md](../state-management/signals.md) for state management
