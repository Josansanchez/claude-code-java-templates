# Authentication - Angular 17+

## Overview

This guide covers JWT-based authentication for Angular applications, including login/logout flow, token management, guards, and interceptors.

## Auth Service

```typescript
// core/services/auth.service.ts
import { Injectable, signal, computed, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Router } from '@angular/router';
import { Observable, tap } from 'rxjs';
import { StorageService } from './storage.service';

interface LoginRequest {
  email: string;
  password: string;
}

interface LoginResponse {
  token: string;
  refreshToken: string;
  user: User;
}

interface User {
  id: string;
  name: string;
  email: string;
  role: string;
}

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  private readonly http = inject(HttpClient);
  private readonly router = inject(Router);
  private readonly storage = inject(StorageService);

  private readonly TOKEN_KEY = 'auth_token';
  private readonly REFRESH_TOKEN_KEY = 'refresh_token';
  private readonly USER_KEY = 'user';

  // Reactive state with signals
  private readonly _currentUser = signal<User | null>(null);
  private readonly _isAuthenticated = signal(false);

  readonly currentUser = this._currentUser.asReadonly();
  readonly isAuthenticated = this._isAuthenticated.asReadonly();
  readonly userRole = computed(() => this._currentUser()?.role ?? null);

  constructor() {
    this.loadUserFromStorage();
  }

  /**
   * Login with email and password
   */
  login(credentials: LoginRequest): Observable<LoginResponse> {
    return this.http.post<LoginResponse>('/api/auth/login', credentials).pipe(
      tap(response => {
        this.setSession(response);
      })
    );
  }

  /**
   * Logout user
   */
  logout(): void {
    this.clearSession();
    this.router.navigate(['/login']);
  }

  /**
   * Refresh access token
   */
  refreshToken(): Observable<{ token: string }> {
    const refreshToken = this.storage.getItem<string>(this.REFRESH_TOKEN_KEY);

    return this.http.post<{ token: string }>('/api/auth/refresh', { refreshToken }).pipe(
      tap(response => {
        this.storage.setItem(this.TOKEN_KEY, response.token);
      })
    );
  }

  /**
   * Get access token
   */
  getToken(): string | null {
    return this.storage.getItem<string>(this.TOKEN_KEY);
  }

  /**
   * Check if token is expired
   */
  isTokenExpired(): boolean {
    const token = this.getToken();
    if (!token) return true;

    try {
      const payload = JSON.parse(atob(token.split('.')[1]));
      const exp = payload.exp * 1000; // Convert to milliseconds
      return Date.now() >= exp;
    } catch {
      return true;
    }
  }

  /**
   * Set session after login
   */
  private setSession(authResult: LoginResponse): void {
    this.storage.setItem(this.TOKEN_KEY, authResult.token);
    this.storage.setItem(this.REFRESH_TOKEN_KEY, authResult.refreshToken);
    this.storage.setItem(this.USER_KEY, authResult.user);

    this._currentUser.set(authResult.user);
    this._isAuthenticated.set(true);
  }

  /**
   * Clear session on logout
   */
  private clearSession(): void {
    this.storage.removeItem(this.TOKEN_KEY);
    this.storage.removeItem(this.REFRESH_TOKEN_KEY);
    this.storage.removeItem(this.USER_KEY);

    this._currentUser.set(null);
    this._isAuthenticated.set(false);
  }

  /**
   * Load user from storage on app init
   */
  private loadUserFromStorage(): void {
    const user = this.storage.getItem<User>(this.USER_KEY);
    const token = this.getToken();

    if (user && token && !this.isTokenExpired()) {
      this._currentUser.set(user);
      this._isAuthenticated.set(true);
    } else {
      this.clearSession();
    }
  }
}
```

---

## Auth Interceptor

```typescript
// core/interceptors/auth.interceptor.ts
import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from '../services/auth.service';
import { Router } from '@angular/router';
import { catchError, throwError } from 'rxjs';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  // Skip auth header for login/register endpoints
  if (req.url.includes('/auth/login') || req.url.includes('/auth/register')) {
    return next(req);
  }

  const token = authService.getToken();

  // Clone request and add Authorization header
  if (token && !authService.isTokenExpired()) {
    req = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
  }

  return next(req).pipe(
    catchError((error: HttpErrorResponse) => {
      // Handle 401 Unauthorized
      if (error.status === 401) {
        authService.logout();
        router.navigate(['/login']);
      }

      return throwError(() => error);
    })
  );
};
```

**Register interceptor:**

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { authInterceptor } from './core/interceptors/auth.interceptor';

export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor])
    )
  ]
};
```

---

## Auth Guard

```typescript
// core/guards/auth.guard.ts
import { inject } from '@angular/core';
import { Router, CanActivateFn } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  if (authService.isAuthenticated() && !authService.isTokenExpired()) {
    return true;
  }

  // Redirect to login with return URL
  return router.createUrlTree(['/login'], {
    queryParams: { returnUrl: state.url }
  });
};
```

**Usage:**

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: 'login',
    loadComponent: () => import('./features/auth/login/login.component')
      .then(m => m.LoginComponent)
  },
  {
    path: 'dashboard',
    loadComponent: () => import('./features/dashboard/dashboard.component')
      .then(m => m.DashboardComponent),
    canActivate: [authGuard]  // Protected route
  }
];
```

---

## Login Component

```typescript
// features/auth/login/login.component.ts
import { Component, inject, signal } from '@angular/core';
import { FormBuilder, ReactiveFormsModule, Validators } from '@angular/forms';
import { Router, ActivatedRoute } from '@angular/router';
import { AuthService } from '../../../core/services/auth.service';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-login',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <div class="login-container">
      <h2>Login</h2>

      <form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
        <div>
          <label for="email">Email:</label>
          <input
            id="email"
            type="email"
            formControlName="email"
            placeholder="email@example.com" />
        </div>

        <div>
          <label for="password">Password:</label>
          <input
            id="password"
            type="password"
            formControlName="password"
            placeholder="Password" />
        </div>

        @if (errorMessage()) {
          <div class="error">{{ errorMessage() }}</div>
        }

        <button
          type="submit"
          [disabled]="loginForm.invalid || isLoading()">
          {{ isLoading() ? 'Logging in...' : 'Login' }}
        </button>
      </form>
    </div>
  `
})
export class LoginComponent {
  private readonly fb = inject(FormBuilder);
  private readonly authService = inject(AuthService);
  private readonly router = inject(Router);
  private readonly route = inject(ActivatedRoute);

  readonly isLoading = signal(false);
  readonly errorMessage = signal<string | null>(null);

  readonly loginForm = this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    password: ['', Validators.required]
  });

  onSubmit(): void {
    if (this.loginForm.invalid) {
      this.loginForm.markAllAsTouched();
      return;
    }

    this.isLoading.set(true);
    this.errorMessage.set(null);

    this.authService.login(this.loginForm.value as any).subscribe({
      next: () => {
        this.isLoading.set(false);

        // Redirect to return URL or dashboard
        const returnUrl = this.route.snapshot.queryParams['returnUrl'] || '/dashboard';
        this.router.navigate([returnUrl]);
      },
      error: (error) => {
        this.errorMessage.set(error.message || 'Login failed');
        this.isLoading.set(false);
      }
    });
  }
}
```

---

## Role-Based Guard

```typescript
// core/guards/role.guard.ts
import { inject } from '@angular/core';
import { Router, CanActivateFn } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const roleGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  const requiredRole = route.data['role'] as string;
  const userRole = authService.userRole();

  if (userRole === requiredRole) {
    return true;
  }

  return router.createUrlTree(['/forbidden']);
};
```

**Usage:**

```typescript
{
  path: 'admin',
  loadComponent: () => import('./features/admin/admin.component')
    .then(m => m.AdminComponent),
  canActivate: [authGuard, roleGuard],
  data: { role: 'admin' }
}
```

---

## Header Component with Auth

```typescript
// layout/header/header.component.ts
import { Component, inject } from '@angular/core';
import { CommonModule } from '@angular/common';
import { RouterLink } from '@angular/router';
import { AuthService } from '../../core/services/auth.service';

@Component({
  selector: 'app-header',
  standalone: true,
  imports: [CommonModule, RouterLink],
  template: `
    <header>
      <nav>
        <a routerLink="/home">Home</a>

        @if (authService.isAuthenticated()) {
          <a routerLink="/dashboard">Dashboard</a>

          @if (authService.userRole() === 'admin') {
            <a routerLink="/admin">Admin</a>
          }

          <span>Welcome, {{ authService.currentUser()?.name }}</span>
          <button (click)="logout()">Logout</button>
        } @else {
          <a routerLink="/login">Login</a>
        }
      </nav>
    </header>
  `
})
export class HeaderComponent {
  readonly authService = inject(AuthService);

  logout(): void {
    this.authService.logout();
  }
}
```

---

## Best Practices

### ✅ DO

1. **Store tokens in localStorage** or sessionStorage
2. **Use HTTP interceptors** for adding tokens
3. **Implement token refresh** before expiration
4. **Use route guards** to protect routes
5. **Handle 401 errors** globally
6. **Check token expiration** before requests
7. **Clear session on logout**
8. **Use HTTPS** in production

### ❌ DON'T

1. **Don't store sensitive data** in localStorage without encryption
2. **Don't send tokens** in URL query params
3. **Don't trust client-side** token validation alone
4. **Don't forget to** logout on 401 errors
5. **Don't hardcode** API URLs

---

## Summary

| Feature | Implementation |
|---------|---------------|
| **Auth Service** | Manages login, logout, token storage |
| **Interceptor** | Adds Authorization header automatically |
| **Auth Guard** | Protects routes from unauthorized access |
| **Role Guard** | Restricts access based on user role |
| **Token Refresh** | Refresh expired tokens automatically |
| **Signals** | Reactive auth state management |

---

## Related Documentation

- See [routing.md](../frontend/routing.md) for guard implementation
- See [services.md](../frontend/services.md) for service patterns
- See [../testing/unit-tests.md](../testing/unit-tests.md) for testing auth
