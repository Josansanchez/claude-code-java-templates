# Routing - Angular 17+

## Overview

Angular routing enables navigation between views/components. Angular 17+ uses **standalone components** with **functional guards** and lazy loading by default.

## Basic Setup

### Root Routes

```typescript
// app.routes.ts
import { Routes } from '@angular/router';

export const routes: Routes = [
  {
    path: '',
    redirectTo: '/home',
    pathMatch: 'full'
  },
  {
    path: 'home',
    loadComponent: () => import('./features/home/home.component')
      .then(m => m.HomeComponent)
  },
  {
    path: 'about',
    loadComponent: () => import('./features/about/about.component')
      .then(m => m.AboutComponent)
  },
  {
    path: '**',
    loadComponent: () => import('./shared/components/not-found/not-found.component')
      .then(m => m.NotFoundComponent)
  }
];
```

### App Configuration

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes)
  ]
};
```

### Root Component

```typescript
// app.component.ts
import { Component } from '@angular/core';
import { RouterOutlet, RouterLink } from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, RouterLink],
  template: `
    <nav>
      <a routerLink="/home" routerLinkActive="active">Home</a>
      <a routerLink="/about" routerLinkActive="active">About</a>
    </nav>
    <router-outlet />
  `
})
export class AppComponent { }
```

---

## Lazy Loading

### Feature Routes (Lazy Loaded)

```typescript
// app.routes.ts
export const routes: Routes = [
  {
    path: 'users',
    loadChildren: () => import('./features/user/user.routes')
      .then(m => m.USER_ROUTES)
  },
  {
    path: 'products',
    loadChildren: () => import('./features/product/product.routes')
      .then(m => m.PRODUCT_ROUTES)
  }
];
```

### Feature Routes File

```typescript
// features/user/user.routes.ts
import { Routes } from '@angular/router';

export const USER_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () => import('./components/user-list/user-list.component')
      .then(m => m.UserListComponent)
  },
  {
    path: 'new',
    loadComponent: () => import('./components/user-form/user-form.component')
      .then(m => m.UserFormComponent)
  },
  {
    path: ':id',
    loadComponent: () => import('./components/user-detail/user-detail.component')
      .then(m => m.UserDetailComponent)
  },
  {
    path: ':id/edit',
    loadComponent: () => import('./components/user-form/user-form.component')
      .then(m => m.UserFormComponent)
  }
];
```

---

## Route Parameters

### Reading Route Parameters

```typescript
import { Component, OnInit, inject } from '@angular/core';
import { ActivatedRoute } from '@angular/router';

@Component({
  selector: 'app-user-detail',
  standalone: true,
  template: `
    <div>
      <h2>User Details</h2>
      <p>User ID: {{ userId }}</p>
    </div>
  `
})
export class UserDetailComponent implements OnInit {
  private readonly route = inject(ActivatedRoute);
  userId: string | null = null;

  ngOnInit(): void {
    // Get parameter once
    this.userId = this.route.snapshot.paramMap.get('id');

    // Subscribe to parameter changes (if route is reused)
    this.route.paramMap.subscribe(params => {
      this.userId = params.get('id');
    });
  }
}
```

### Route Parameters with Signals

```typescript
import { Component, inject, signal } from '@angular/core';
import { ActivatedRoute } from '@angular/router';
import { toSignal } from '@angular/core/rxjs-interop';
import { map } from 'rxjs/operators';

@Component({
  selector: 'app-user-detail',
  standalone: true,
  template: `
    <div>
      <h2>User ID: {{ userId() }}</h2>
    </div>
  `
})
export class UserDetailComponent {
  private readonly route = inject(ActivatedRoute);

  readonly userId = toSignal(
    this.route.paramMap.pipe(
      map(params => params.get('id'))
    )
  );
}
```

### Query Parameters

```typescript
// Navigate with query params
this.router.navigate(['/users'], {
  queryParams: { page: 1, size: 20 }
});

// Read query params
readonly route = inject(ActivatedRoute);

ngOnInit(): void {
  this.route.queryParamMap.subscribe(params => {
    const page = params.get('page');
    const size = params.get('size');
    console.log('Page:', page, 'Size:', size);
  });
}
```

---

## Route Guards

### Functional Guards (Recommended)

```typescript
// core/guards/auth.guard.ts
import { inject } from '@angular/core';
import { Router, CanActivateFn } from '@angular/router';
import { AuthService } from '../services/auth.service';

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

**Using guard:**

```typescript
export const routes: Routes = [
  {
    path: 'dashboard',
    loadComponent: () => import('./features/dashboard/dashboard.component')
      .then(m => m.DashboardComponent),
    canActivate: [authGuard]
  }
];
```

### Role-Based Guard

```typescript
// core/guards/role.guard.ts
import { inject } from '@angular/core';
import { Router, CanActivateFn } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const roleGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);

  const requiredRole = route.data['role'] as string;
  const userRole = authService.getUserRole();

  if (userRole === requiredRole) {
    return true;
  }

  return router.createUrlTree(['/forbidden']);
};
```

**Using with data:**

```typescript
{
  path: 'admin',
  loadComponent: () => import('./features/admin/admin.component')
    .then(m => m.AdminComponent),
  canActivate: [authGuard, roleGuard],
  data: { role: 'admin' }
}
```

### CanDeactivate Guard (Unsaved Changes)

```typescript
// core/guards/unsaved-changes.guard.ts
import { CanDeactivateFn } from '@angular/router';

export interface CanComponentDeactivate {
  canDeactivate: () => boolean;
}

export const unsavedChangesGuard: CanDeactivateFn<CanComponentDeactivate> = (component) => {
  if (component.canDeactivate()) {
    return true;
  }

  return confirm('You have unsaved changes. Do you really want to leave?');
};
```

**Component implementing interface:**

```typescript
export class UserFormComponent implements CanComponentDeactivate {
  form = this.fb.group({ name: [''], email: [''] });

  canDeactivate(): boolean {
    return !this.form.dirty;
  }
}
```

**Using guard:**

```typescript
{
  path: 'edit',
  component: UserFormComponent,
  canDeactivate: [unsavedChangesGuard]
}
```

---

## Resolvers

### Functional Resolver

```typescript
// features/user/resolvers/user.resolver.ts
import { inject } from '@angular/core';
import { ResolveFn } from '@angular/router';
import { UserService } from '../services/user.service';
import { User } from '../models/user.model';

export const userResolver: ResolveFn<User> = (route, state) => {
  const userService = inject(UserService);
  const id = route.paramMap.get('id')!;

  return userService.getUser(id);
};
```

**Using resolver:**

```typescript
{
  path: ':id',
  loadComponent: () => import('./components/user-detail/user-detail.component')
    .then(m => m.UserDetailComponent),
  resolve: {
    user: userResolver
  }
}
```

**Accessing resolved data:**

```typescript
@Component({
  selector: 'app-user-detail',
  standalone: true,
  template: `
    <h2>{{ user.name }}</h2>
    <p>{{ user.email }}</p>
  `
})
export class UserDetailComponent implements OnInit {
  private readonly route = inject(ActivatedRoute);
  user!: User;

  ngOnInit(): void {
    this.user = this.route.snapshot.data['user'];
  }
}
```

---

## Child Routes

```typescript
// features/admin/admin.routes.ts
export const ADMIN_ROUTES: Routes = [
  {
    path: '',
    loadComponent: () => import('./components/admin-layout/admin-layout.component')
      .then(m => m.AdminLayoutComponent),
    canActivate: [authGuard, roleGuard],
    data: { role: 'admin' },
    children: [
      {
        path: '',
        redirectTo: 'users',
        pathMatch: 'full'
      },
      {
        path: 'users',
        loadComponent: () => import('./components/admin-users/admin-users.component')
          .then(m => m.AdminUsersComponent)
      },
      {
        path: 'products',
        loadComponent: () => import('./components/admin-products/admin-products.component')
          .then(m => m.AdminProductsComponent)
      },
      {
        path: 'settings',
        loadComponent: () => import('./components/admin-settings/admin-settings.component')
          .then(m => m.AdminSettingsComponent)
      }
    ]
  }
];
```

**Parent component with child outlet:**

```typescript
@Component({
  selector: 'app-admin-layout',
  standalone: true,
  imports: [RouterOutlet, RouterLink],
  template: `
    <div class="admin-layout">
      <nav>
        <a routerLink="users" routerLinkActive="active">Users</a>
        <a routerLink="products" routerLinkActive="active">Products</a>
        <a routerLink="settings" routerLinkActive="active">Settings</a>
      </nav>
      <main>
        <router-outlet />
      </main>
    </div>
  `
})
export class AdminLayoutComponent { }
```

---

## Programmatic Navigation

```typescript
import { Component, inject } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-user-list',
  standalone: true
})
export class UserListComponent {
  private readonly router = inject(Router);

  // Navigate to route
  goToUser(id: string): void {
    this.router.navigate(['/users', id]);
  }

  // Navigate with query params
  goToUsersPage(page: number): void {
    this.router.navigate(['/users'], {
      queryParams: { page, size: 20 }
    });
  }

  // Navigate back
  goBack(): void {
    this.router.navigate(['..'], { relativeTo: this.route });
  }

  // Replace history (no back navigation)
  replaceUrl(): void {
    this.router.navigate(['/users'], { replaceUrl: true });
  }
}
```

---

## Best Practices

### ✅ DO

1. **Use lazy loading** for feature routes
2. **Use functional guards** (canActivate, canDeactivate)
3. **Use resolvers** to pre-fetch data
4. **Use route parameters** for dynamic data
5. **Use child routes** for nested views
6. **Use signals** to read route params reactively
7. **Protect routes** with guards
8. **Use redirects** for default paths

### ❌ DON'T

1. **Don't forget 404 route** (`path: '**'`)
2. **Don't load all features eagerly**
3. **Don't put business logic** in guards
4. **Don't forget** `pathMatch: 'full'` for redirects
5. **Don't hardcode URLs** (use router.navigate)

### Code Examples

#### ✅ GOOD: Lazy loading

```typescript
{
  path: 'users',
  loadChildren: () => import('./features/user/user.routes')
    .then(m => m.USER_ROUTES)
}
```

#### ❌ BAD: Eager loading everything

```typescript
// ❌ Don't import all components at root level
import { UserListComponent } from './features/user/user-list.component';
import { ProductListComponent } from './features/product/product-list.component';
// ... loads too much upfront
```

#### ✅ GOOD: Functional guard

```typescript
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  return authService.isAuthenticated();
};
```

---

## Summary

| Feature | Recommendation |
|---------|---------------|
| **Lazy Loading** | Use loadChildren/loadComponent |
| **Guards** | Functional guards (CanActivateFn) |
| **Resolvers** | Functional resolvers (ResolveFn) |
| **Parameters** | Use signals with toSignal() |
| **Navigation** | Programmatic with Router.navigate() |
| **Child Routes** | For nested layouts |

---

## Related Documentation

- See [components.md](./components.md) for component patterns
- See [../architecture/project-structure.md](../architecture/project-structure.md) for routing structure
- See [../security/authentication.md](../security/authentication.md) for auth guards
