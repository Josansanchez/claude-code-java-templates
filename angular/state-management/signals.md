# Signals - Angular 17+

## Overview

Signals are Angular's reactive primitive for managing state. They provide a simpler and more performant alternative to RxJS for many use cases.

## Why Signals?

**Before Signals (RxJS + ChangeDetection):**
- Complex RxJS operators
- Manual subscription management
- Zone.js overhead
- Difficult to debug

**With Signals:**
- Simpler API (`signal()`, `computed()`, `effect()`)
- Automatic change detection
- No subscription management needed
- Fine-grained reactivity
- Better performance

---

## Signal Basics

### Creating Signals

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <p>Count: {{ count() }}</p>
    <button (click)="increment()">+1</button>
  `
})
export class CounterComponent {
  // Writable signal with initial value
  readonly count = signal(0);

  increment(): void {
    this.count.update(value => value + 1);
  }
}
```

### Reading Signals

```typescript
// In template (automatically tracks dependencies)
<p>Count: {{ count() }}</p>

// In TypeScript (call as function)
const currentValue = this.count();
```

### Updating Signals

```typescript
readonly count = signal(0);

// Set to specific value
this.count.set(10);

// Update based on current value
this.count.update(value => value + 1);
```

---

## Computed Signals

Computed signals derive their value from other signals. They automatically recompute when dependencies change.

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-cart',
  standalone: true,
  template: `
    <p>Items: {{ itemCount() }}</p>
    <p>Total: ${{ total() }}</p>
    <p>Average: ${{ averagePrice() }}</p>
  `
})
export class CartComponent {
  readonly items = signal<CartItem[]>([
    { name: 'Book', price: 10, quantity: 2 },
    { name: 'Pen', price: 5, quantity: 3 }
  ]);

  // Computed: automatically updates when items change
  readonly itemCount = computed(() =>
    this.items().reduce((sum, item) => sum + item.quantity, 0)
  );

  readonly total = computed(() =>
    this.items().reduce((sum, item) => sum + (item.price * item.quantity), 0)
  );

  readonly averagePrice = computed(() => {
    const items = this.items();
    const total = this.total();
    return items.length > 0 ? total / items.length : 0;
  });
}
```

### Computed with Multiple Dependencies

```typescript
readonly firstName = signal('John');
readonly lastName = signal('Doe');
readonly age = signal(30);

// Depends on multiple signals
readonly fullName = computed(() =>
  `${this.firstName()} ${this.lastName()}`
);

readonly greeting = computed(() =>
  `Hello, ${this.fullName()}! You are ${this.age()} years old.`
);
```

---

## Effects

Effects run side effects when signals change. They automatically track signal dependencies.

```typescript
import { Component, signal, effect } from '@angular/core';

@Component({
  selector: 'app-theme',
  standalone: true,
  template: `
    <label>
      <input
        type="checkbox"
        [checked]="darkMode()"
        (change)="toggleDarkMode()" />
      Dark Mode
    </label>
  `
})
export class ThemeComponent {
  readonly darkMode = signal(false);

  constructor() {
    // Effect runs whenever darkMode changes
    effect(() => {
      const isDark = this.darkMode();
      console.log('Dark mode changed:', isDark);

      // Side effect: update DOM
      if (isDark) {
        document.body.classList.add('dark-theme');
      } else {
        document.body.classList.remove('dark-theme');
      }

      // Side effect: save to localStorage
      localStorage.setItem('darkMode', JSON.stringify(isDark));
    });

    // Load initial value
    const saved = localStorage.getItem('darkMode');
    if (saved) {
      this.darkMode.set(JSON.parse(saved));
    }
  }

  toggleDarkMode(): void {
    this.darkMode.update(value => !value);
  }
}
```

### Effect Cleanup

```typescript
constructor() {
  effect((onCleanup) => {
    const timer = setInterval(() => {
      console.log('Timer tick');
    }, 1000);

    // Cleanup function runs before next effect execution
    onCleanup(() => {
      clearInterval(timer);
    });
  });
}
```

---

## Signal Patterns

### 1. State Service with Signals

```typescript
// features/user/services/user-state.service.ts
import { Injectable, signal, computed } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class UserStateService {
  // Private writable signals
  private readonly _users = signal<User[]>([]);
  private readonly _selectedUserId = signal<string | null>(null);
  private readonly _isLoading = signal(false);

  // Public readonly signals
  readonly users = this._users.asReadonly();
  readonly isLoading = this._isLoading.asReadonly();

  // Computed signals
  readonly userCount = computed(() => this._users().length);

  readonly selectedUser = computed(() => {
    const id = this._selectedUserId();
    return id ? this._users().find(u => u.id === id) : null;
  });

  readonly activeUsers = computed(() =>
    this._users().filter(u => u.isActive)
  );

  // Actions
  setUsers(users: User[]): void {
    this._users.set(users);
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

  selectUser(id: string | null): void {
    this._selectedUserId.set(id);
  }

  setLoading(loading: boolean): void {
    this._isLoading.set(loading);
  }
}
```

### 2. Signals with HTTP

```typescript
import { Component, signal, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';

@Component({
  selector: 'app-user-list',
  standalone: true,
  template: `
    @if (isLoading()) {
      <p>Loading...</p>
    } @else if (error()) {
      <p class="error">{{ error() }}</p>
    } @else {
      @for (user of users(); track user.id) {
        <div>{{ user.name }}</div>
      }
    }
  `
})
export class UserListComponent {
  private readonly http = inject(HttpClient);

  readonly users = signal<User[]>([]);
  readonly isLoading = signal(false);
  readonly error = signal<string | null>(null);

  ngOnInit(): void {
    this.loadUsers();
  }

  private loadUsers(): void {
    this.isLoading.set(true);
    this.error.set(null);

    this.http.get<User[]>('/api/users').subscribe({
      next: (users) => {
        this.users.set(users);
        this.isLoading.set(false);
      },
      error: (err) => {
        this.error.set(err.message);
        this.isLoading.set(false);
      }
    });
  }
}
```

### 3. Signals with Forms

```typescript
import { Component, signal, computed } from '@angular/core';
import { FormBuilder, Validators } from '@angular/forms';

@Component({
  selector: 'app-user-form',
  standalone: true,
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input type="text" formControlName="name" />
      <input type="email" formControlName="email" />

      @if (isSubmitting()) {
        <p>Submitting...</p>
      }

      <button type="submit" [disabled]="isSubmitDisabled()">
        {{ submitButtonText() }}
      </button>
    </form>
  `
})
export class UserFormComponent {
  private readonly fb = inject(FormBuilder);

  readonly form = this.fb.group({
    name: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]]
  });

  readonly isSubmitting = signal(false);
  readonly error = signal<string | null>(null);

  readonly isSubmitDisabled = computed(() =>
    this.form.invalid || this.isSubmitting()
  );

  readonly submitButtonText = computed(() =>
    this.isSubmitting() ? 'Submitting...' : 'Submit'
  );

  onSubmit(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    this.isSubmitting.set(true);
    // ... submit logic
  }
}
```

---

## Signals + RxJS Integration

### toSignal() - Observable to Signal

```typescript
import { Component, inject } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import { interval } from 'rxjs';

@Component({
  selector: 'app-timer',
  standalone: true,
  template: `
    <p>Seconds: {{ seconds() }}</p>
  `
})
export class TimerComponent {
  // Convert Observable to Signal
  readonly seconds = toSignal(interval(1000), { initialValue: 0 });
}
```

### toSignal() with HTTP

```typescript
import { Component, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { toSignal } from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-users',
  standalone: true,
  template: `
    @if (users(); as userList) {
      @for (user of userList; track user.id) {
        <div>{{ user.name }}</div>
      }
    }
  `
})
export class UsersComponent {
  private readonly http = inject(HttpClient);

  readonly users = toSignal(
    this.http.get<User[]>('/api/users'),
    { initialValue: [] as User[] }
  );
}
```

### toObservable() - Signal to Observable

```typescript
import { Component, signal } from '@angular/core';
import { toObservable } from '@angular/core/rxjs-interop';
import { debounceTime, distinctUntilChanged, switchMap } from 'rxjs/operators';

@Component({
  selector: 'app-search',
  standalone: true,
  template: `
    <input
      type="text"
      [value]="query()"
      (input)="onQueryChange($event)" />

    @for (result of results(); track result.id) {
      <div>{{ result.name }}</div>
    }
  `
})
export class SearchComponent {
  private readonly http = inject(HttpClient);

  readonly query = signal('');

  // Convert signal to observable for RxJS operators
  private readonly query$ = toObservable(this.query);

  readonly results = toSignal(
    this.query$.pipe(
      debounceTime(300),
      distinctUntilChanged(),
      switchMap(q => this.http.get<SearchResult[]>(`/api/search?q=${q}`))
    ),
    { initialValue: [] as SearchResult[] }
  );

  onQueryChange(event: Event): void {
    const value = (event.target as HTMLInputElement).value;
    this.query.set(value);
  }
}
```

---

## Advanced Patterns

### Async Signal State

```typescript
interface AsyncState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

@Injectable({
  providedIn: 'root'
})
export class AsyncStateService<T> {
  private readonly _state = signal<AsyncState<T>>({
    data: null,
    loading: false,
    error: null
  });

  readonly state = this._state.asReadonly();
  readonly data = computed(() => this._state().data);
  readonly loading = computed(() => this._state().loading);
  readonly error = computed(() => this._state().error);

  setLoading(): void {
    this._state.update(s => ({ ...s, loading: true, error: null }));
  }

  setData(data: T): void {
    this._state.set({ data, loading: false, error: null });
  }

  setError(error: string): void {
    this._state.update(s => ({ ...s, loading: false, error }));
  }
}
```

### Pagination with Signals

```typescript
@Injectable({
  providedIn: 'root'
})
export class PaginationService {
  private readonly _page = signal(0);
  private readonly _size = signal(20);
  private readonly _total = signal(0);

  readonly page = this._page.asReadonly();
  readonly size = this._size.asReadonly();
  readonly total = this._total.asReadonly();

  readonly totalPages = computed(() =>
    Math.ceil(this._total() / this._size())
  );

  readonly hasNextPage = computed(() =>
    this._page() < this.totalPages() - 1
  );

  readonly hasPreviousPage = computed(() =>
    this._page() > 0
  );

  setPage(page: number): void {
    this._page.set(page);
  }

  nextPage(): void {
    if (this.hasNextPage()) {
      this._page.update(p => p + 1);
    }
  }

  previousPage(): void {
    if (this.hasPreviousPage()) {
      this._page.update(p => p - 1);
    }
  }

  setTotal(total: number): void {
    this._total.set(total);
  }
}
```

---

## Best Practices

### ✅ DO

1. **Use signals for simple state** management
2. **Use computed()** for derived values
3. **Use effect()** for side effects only
4. **Expose readonly signals** from services
5. **Use toSignal()** to convert observables
6. **Keep effects simple** and focused
7. **Use signal inputs** (Angular 17.1+)
8. **Prefer signals over RxJS** for simple cases

### ❌ DON'T

1. **Don't mutate signal values** directly
2. **Don't use effects for state updates** (use computed instead)
3. **Don't overuse signals** (use RxJS when appropriate)
4. **Don't expose writable signals** publicly
5. **Don't forget cleanup** in effects
6. **Don't use signals for everything** (RxJS still has its place)

### Code Examples

#### ✅ GOOD: Immutable updates

```typescript
readonly items = signal<Item[]>([]);

// ✅ GOOD: Create new array
addItem(item: Item): void {
  this.items.update(items => [...items, item]);
}
```

#### ❌ BAD: Mutating signal value

```typescript
// ❌ BAD: Mutates array
addItem(item: Item): void {
  this.items().push(item);  // Won't trigger updates!
}
```

#### ✅ GOOD: Computed for derived state

```typescript
readonly items = signal<Item[]>([]);

// ✅ GOOD: Use computed
readonly total = computed(() =>
  this.items().reduce((sum, item) => sum + item.price, 0)
);
```

#### ❌ BAD: Effect for derived state

```typescript
// ❌ BAD: Don't use effect for derived values
constructor() {
  effect(() => {
    const items = this.items();
    this.total.set(items.reduce((sum, item) => sum + item.price, 0));
  });
}
```

---

## Summary

| Feature | Use Case |
|---------|----------|
| **signal()** | Writable reactive state |
| **computed()** | Derived values from signals |
| **effect()** | Side effects (DOM, localStorage, logging) |
| **toSignal()** | Convert Observable → Signal |
| **toObservable()** | Convert Signal → Observable |
| **asReadonly()** | Expose read-only signals |

---

## Related Documentation

- See [services.md](../frontend/services.md) for state services
- See [components.md](../frontend/components.md) for component signals
- See [../testing/unit-tests.md](../testing/unit-tests.md) for testing signals
