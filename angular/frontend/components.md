# Components - Angular 17+

## Overview

Components are the fundamental building blocks of Angular applications. Angular 17+ emphasizes **standalone components** with **signals** for reactive state management.

## Table of Contents

1. [Standalone Components](#standalone-components)
2. [Component Lifecycle](#component-lifecycle)
3. [Signals in Components](#signals-in-components)
4. [Input and Output](#input-and-output)
5. [Change Detection](#change-detection)
6. [Component Communication](#component-communication)
7. [Smart vs Presentational Components](#smart-vs-presentational-components)
8. [Component Composition](#component-composition)
9. [Best Practices](#best-practices)

---

## Standalone Components

### What Are Standalone Components?

Standalone components are self-contained components that don't require NgModules. This is the **default and recommended approach** in Angular 17+.

### Creating a Standalone Component

```typescript
// user-card.component.ts
import { Component, ChangeDetectionStrategy } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-user-card',
  standalone: true,                              // ✅ Standalone flag
  imports: [CommonModule],                       // ✅ Import dependencies directly
  templateUrl: './user-card.component.html',
  styleUrls: ['./user-card.component.scss'],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserCardComponent {
  // Component logic
}
```

### CLI Command

```bash
# Generate standalone component (default in Angular 17+)
ng generate component features/user/user-card

# Or shorthand
ng g c features/user/user-card
```

### Importing Other Components

```typescript
@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [
    CommonModule,           // NgIf, NgFor, etc.
    RouterLink,             // Router directives
    UserCardComponent,      // Custom component
    ButtonComponent         // Shared component
  ],
  template: `
    <div class="user-list">
      @for (user of users; track user.id) {
        <app-user-card [user]="user" />
      }
      <app-button (click)="addUser()">Add User</app-button>
    </div>
  `
})
export class UserListComponent { }
```

---

## Component Lifecycle

### Lifecycle Hooks

Angular components have lifecycle hooks that allow you to tap into key moments.

```typescript
import {
  Component,
  OnInit,
  OnChanges,
  OnDestroy,
  AfterViewInit,
  SimpleChanges
} from '@angular/core';

@Component({
  selector: 'app-lifecycle-demo',
  standalone: true,
  template: `<p>Lifecycle Demo</p>`
})
export class LifecycleDemoComponent implements OnInit, OnChanges, OnDestroy, AfterViewInit {

  /**
   * Called when input properties change
   */
  ngOnChanges(changes: SimpleChanges): void {
    console.log('Input changed:', changes);
  }

  /**
   * Called once after the first ngOnChanges
   * Perfect for initialization logic
   */
  ngOnInit(): void {
    console.log('Component initialized');
    this.loadData();
  }

  /**
   * Called after view initialization
   * Safe to access @ViewChild elements
   */
  ngAfterViewInit(): void {
    console.log('View initialized');
  }

  /**
   * Called just before the component is destroyed
   * Perfect for cleanup (unsubscribe, clear timers)
   */
  ngOnDestroy(): void {
    console.log('Component destroyed');
    this.cleanup();
  }

  private loadData(): void {
    // Load initial data
  }

  private cleanup(): void {
    // Cleanup resources
  }
}
```

### Modern Approach: DestroyRef (Angular 16+)

Instead of implementing `OnDestroy`, use `DestroyRef` for cleanup:

```typescript
import { Component, OnInit, inject, DestroyRef } from '@angular/core';
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

@Component({
  selector: 'app-user-list',
  standalone: true,
  template: `...`
})
export class UserListComponent implements OnInit {
  private readonly userService = inject(UserService);
  private readonly destroyRef = inject(DestroyRef);

  ngOnInit(): void {
    // Automatically unsubscribes when component is destroyed
    this.userService.getUsers()
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(users => {
        console.log('Users loaded:', users);
      });
  }
}
```

**Even better with signals:**

```typescript
@Component({
  selector: 'app-user-list',
  standalone: true,
  template: `
    @for (user of users(); track user.id) {
      <app-user-card [user]="user" />
    }
  `
})
export class UserListComponent implements OnInit {
  private readonly userService = inject(UserService);
  private readonly destroyRef = inject(DestroyRef);

  readonly users = signal<User[]>([]);

  ngOnInit(): void {
    this.userService.getUsers()
      .pipe(takeUntilDestroyed(this.destroyRef))
      .subscribe(users => this.users.set(users));
  }
}
```

---

## Signals in Components

### What Are Signals?

Signals are Angular's new reactive primitive for managing state and computed values.

### Basic Signal Usage

```typescript
import { Component, signal, computed } from '@angular/core';

@Component({
  selector: 'app-counter',
  standalone: true,
  template: `
    <div class="counter">
      <p>Count: {{ count() }}</p>
      <p>Double: {{ doubleCount() }}</p>
      <p>Is even: {{ isEven() }}</p>

      <button (click)="increment()">+1</button>
      <button (click)="decrement()">-1</button>
      <button (click)="reset()">Reset</button>
    </div>
  `
})
export class CounterComponent {
  // Writable signal
  readonly count = signal(0);

  // Computed signals (derived state)
  readonly doubleCount = computed(() => this.count() * 2);
  readonly isEven = computed(() => this.count() % 2 === 0);

  // Methods to update signal
  increment(): void {
    this.count.update(c => c + 1);
  }

  decrement(): void {
    this.count.update(c => c - 1);
  }

  reset(): void {
    this.count.set(0);
  }
}
```

### Signals with Objects

```typescript
import { Component, signal, computed } from '@angular/core';

interface User {
  id: string;
  name: string;
  email: string;
  isActive: boolean;
}

@Component({
  selector: 'app-user-profile',
  standalone: true,
  template: `
    <div class="profile">
      <h2>{{ user().name }}</h2>
      <p>{{ user().email }}</p>
      <p>Status: {{ statusText() }}</p>

      <button (click)="toggleActive()">
        {{ user().isActive ? 'Deactivate' : 'Activate' }}
      </button>
    </div>
  `
})
export class UserProfileComponent {
  readonly user = signal<User>({
    id: '1',
    name: 'John Doe',
    email: 'john@example.com',
    isActive: true
  });

  readonly statusText = computed(() =>
    this.user().isActive ? 'Active' : 'Inactive'
  );

  toggleActive(): void {
    this.user.update(u => ({
      ...u,
      isActive: !u.isActive
    }));
  }

  updateEmail(newEmail: string): void {
    this.user.update(u => ({
      ...u,
      email: newEmail
    }));
  }
}
```

### Signals with Arrays

```typescript
@Component({
  selector: 'app-todo-list',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="todo-list">
      <p>Total: {{ todos().length }} | Completed: {{ completedCount() }}</p>

      @for (todo of todos(); track todo.id) {
        <div class="todo-item">
          <input
            type="checkbox"
            [checked]="todo.completed"
            (change)="toggleTodo(todo.id)" />
          <span [class.completed]="todo.completed">{{ todo.text }}</span>
          <button (click)="deleteTodo(todo.id)">Delete</button>
        </div>
      }
    </div>
  `
})
export class TodoListComponent {
  readonly todos = signal<Todo[]>([
    { id: '1', text: 'Learn Angular', completed: false },
    { id: '2', text: 'Build app', completed: false }
  ]);

  readonly completedCount = computed(() =>
    this.todos().filter(t => t.completed).length
  );

  addTodo(text: string): void {
    const newTodo: Todo = {
      id: crypto.randomUUID(),
      text,
      completed: false
    };
    this.todos.update(todos => [...todos, newTodo]);
  }

  toggleTodo(id: string): void {
    this.todos.update(todos =>
      todos.map(t => t.id === id ? { ...t, completed: !t.completed } : t)
    );
  }

  deleteTodo(id: string): void {
    this.todos.update(todos => todos.filter(t => t.id !== id));
  }
}
```

### Effects (Side Effects with Signals)

```typescript
import { Component, signal, effect } from '@angular/core';

@Component({
  selector: 'app-user-settings',
  standalone: true,
  template: `
    <div>
      <label>
        <input
          type="checkbox"
          [checked]="darkMode()"
          (change)="toggleDarkMode()" />
        Dark Mode
      </label>
    </div>
  `
})
export class UserSettingsComponent {
  readonly darkMode = signal(false);

  constructor() {
    // Effect runs whenever darkMode changes
    effect(() => {
      const isDark = this.darkMode();
      console.log('Dark mode:', isDark);

      // Side effect: update DOM
      if (isDark) {
        document.body.classList.add('dark-theme');
      } else {
        document.body.classList.remove('dark-theme');
      }

      // Side effect: save to localStorage
      localStorage.setItem('darkMode', JSON.stringify(isDark));
    });

    // Load initial value from localStorage
    const saved = localStorage.getItem('darkMode');
    if (saved) {
      this.darkMode.set(JSON.parse(saved));
    }
  }

  toggleDarkMode(): void {
    this.darkMode.update(dark => !dark);
  }
}
```

---

## Input and Output

### Input Properties

```typescript
import { Component, Input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  standalone: true,
  template: `
    <div class="card">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
      @if (showDetails) {
        <p>ID: {{ user.id }}</p>
      }
    </div>
  `
})
export class UserCardComponent {
  // Required input
  @Input({ required: true }) user!: User;

  // Optional input with default value
  @Input() showDetails = false;

  // Input with alias
  @Input('isActive') active = true;

  // Input with transform
  @Input({ transform: booleanAttribute }) disabled = false;
}
```

**Using in parent:**

```html
<app-user-card
  [user]="selectedUser"
  [showDetails]="true"
  [isActive]="user.active"
  disabled />
```

### Signal Inputs (Angular 17.1+)

New reactive input approach using signals:

```typescript
import { Component, input } from '@angular/core';

@Component({
  selector: 'app-user-card',
  standalone: true,
  template: `
    <div class="card">
      <h3>{{ user().name }}</h3>
      <p>{{ user().email }}</p>
      <p>Full name: {{ fullName() }}</p>
    </div>
  `
})
export class UserCardComponent {
  // Signal input (required)
  readonly user = input.required<User>();

  // Signal input (optional with default)
  readonly showDetails = input(false);

  // Computed value based on signal input
  readonly fullName = computed(() => {
    const u = this.user();
    return `${u.firstName} ${u.lastName}`;
  });
}
```

### Output Events

```typescript
import { Component, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-user-card',
  standalone: true,
  template: `
    <div class="card">
      <h3>{{ user.name }}</h3>
      <button (click)="onEdit()">Edit</button>
      <button (click)="onDelete()">Delete</button>
    </div>
  `
})
export class UserCardComponent {
  @Input({ required: true }) user!: User;

  // Output events
  @Output() edit = new EventEmitter<User>();
  @Output() delete = new EventEmitter<string>();

  onEdit(): void {
    this.edit.emit(this.user);
  }

  onDelete(): void {
    this.delete.emit(this.user.id);
  }
}
```

**Using in parent:**

```typescript
@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [UserCardComponent],
  template: `
    @for (user of users; track user.id) {
      <app-user-card
        [user]="user"
        (edit)="handleEdit($event)"
        (delete)="handleDelete($event)" />
    }
  `
})
export class UserListComponent {
  users: User[] = [];

  handleEdit(user: User): void {
    console.log('Edit user:', user);
  }

  handleDelete(userId: string): void {
    console.log('Delete user:', userId);
  }
}
```

### Two-Way Binding

```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-search-box',
  standalone: true,
  imports: [FormsModule],
  template: `
    <input
      type="text"
      [value]="query"
      (input)="onQueryChange($event)" />
  `
})
export class SearchBoxComponent {
  @Input() query = '';
  @Output() queryChange = new EventEmitter<string>();

  onQueryChange(event: Event): void {
    const value = (event.target as HTMLInputElement).value;
    this.queryChange.emit(value);
  }
}
```

**Using with two-way binding:**

```html
<!-- Parent component -->
<app-search-box [(query)]="searchQuery" />

<!-- Equivalent to: -->
<app-search-box
  [query]="searchQuery"
  (queryChange)="searchQuery = $event" />
```

---

## Change Detection

### OnPush Strategy (Recommended)

Use `OnPush` for better performance. Change detection only runs when:
1. Input properties change (by reference)
2. Events occur in the component
3. Async pipe emits a new value
4. Change detection is manually triggered

```typescript
import { Component, ChangeDetectionStrategy } from '@angular/core';

@Component({
  selector: 'app-user-list',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,  // ✅ OnPush
  template: `
    @for (user of users; track user.id) {
      <app-user-card [user]="user" />
    }
  `
})
export class UserListComponent {
  users: User[] = [];

  // ✅ CORRECT: Create new array reference
  addUser(user: User): void {
    this.users = [...this.users, user];
  }

  // ❌ WRONG: Mutates array, OnPush won't detect
  addUserWrong(user: User): void {
    this.users.push(user);  // Won't trigger change detection
  }
}
```

### Manual Change Detection

```typescript
import { Component, ChangeDetectionStrategy, ChangeDetectorRef, inject } from '@angular/core';

@Component({
  selector: 'app-manual-detection',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<p>{{ count }}</p>`
})
export class ManualDetectionComponent {
  private readonly cdr = inject(ChangeDetectorRef);
  count = 0;

  incrementOutsideZone(): void {
    // If this runs outside Angular's zone
    setTimeout(() => {
      this.count++;
      this.cdr.markForCheck();  // Manually trigger change detection
    }, 1000);
  }
}
```

### Signals + OnPush = Best Performance

Signals work perfectly with OnPush:

```typescript
@Component({
  selector: 'app-optimized',
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,  // ✅
  template: `
    <p>Count: {{ count() }}</p>
    <p>Double: {{ double() }}</p>
  `
})
export class OptimizedComponent {
  readonly count = signal(0);
  readonly double = computed(() => this.count() * 2);

  increment(): void {
    this.count.update(c => c + 1);  // Automatically triggers change detection
  }
}
```

---

## Component Communication

### 1. Parent → Child (via @Input)

```typescript
// Parent
@Component({
  template: `<app-child [data]="parentData" />`
})
export class ParentComponent {
  parentData = 'Hello from parent';
}

// Child
@Component({
  selector: 'app-child'
})
export class ChildComponent {
  @Input({ required: true }) data!: string;
}
```

### 2. Child → Parent (via @Output)

```typescript
// Child
@Component({
  selector: 'app-child',
  template: `<button (click)="notify()">Click</button>`
})
export class ChildComponent {
  @Output() message = new EventEmitter<string>();

  notify(): void {
    this.message.emit('Hello from child');
  }
}

// Parent
@Component({
  template: `<app-child (message)="handleMessage($event)" />`
})
export class ParentComponent {
  handleMessage(msg: string): void {
    console.log(msg);
  }
}
```

### 3. Service (for Sibling Communication)

```typescript
// Shared service
@Injectable({ providedIn: 'root' })
export class MessageService {
  private readonly messageSubject = new BehaviorSubject<string>('');
  readonly message$ = this.messageSubject.asObservable();

  sendMessage(message: string): void {
    this.messageSubject.next(message);
  }
}

// Component A
export class ComponentA {
  private readonly messageService = inject(MessageService);

  send(): void {
    this.messageService.sendMessage('Hello');
  }
}

// Component B
export class ComponentB implements OnInit {
  private readonly messageService = inject(MessageService);

  ngOnInit(): void {
    this.messageService.message$.subscribe(msg => console.log(msg));
  }
}
```

### 4. ViewChild (Parent accessing Child)

```typescript
import { Component, ViewChild, AfterViewInit } from '@angular/core';

@Component({
  selector: 'app-parent',
  template: `<app-child />`,
  imports: [ChildComponent]
})
export class ParentComponent implements AfterViewInit {
  @ViewChild(ChildComponent) child!: ChildComponent;

  ngAfterViewInit(): void {
    // Access child component
    this.child.doSomething();
  }
}
```

---

## Smart vs Presentational Components

### Smart Components (Containers)

**Characteristics:**
- Manage state
- Call services
- Handle business logic
- Pass data to presentational components

```typescript
// features/user/user-list/user-list.component.ts
@Component({
  selector: 'app-user-list',
  standalone: true,
  imports: [CommonModule, UserCardComponent],
  template: `
    <div class="user-list">
      @if (isLoading()) {
        <app-loader />
      } @else {
        @for (user of users(); track user.id) {
          <app-user-card
            [user]="user"
            (delete)="deleteUser($event)" />
        }
      }
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserListComponent implements OnInit {
  private readonly userService = inject(UserService);

  readonly users = signal<User[]>([]);
  readonly isLoading = signal(false);

  ngOnInit(): void {
    this.loadUsers();
  }

  loadUsers(): void {
    this.isLoading.set(true);
    this.userService.getUsers().subscribe(users => {
      this.users.set(users);
      this.isLoading.set(false);
    });
  }

  deleteUser(id: string): void {
    this.userService.deleteUser(id).subscribe(() => this.loadUsers());
  }
}
```

### Presentational Components (Dumb)

**Characteristics:**
- Only display data
- No service dependencies
- Receive data via @Input
- Emit events via @Output
- Highly reusable

```typescript
// shared/components/user-card/user-card.component.ts
@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="card">
      <h3>{{ user.name }}</h3>
      <p>{{ user.email }}</p>
      <button (click)="onDelete()">Delete</button>
    </div>
  `,
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

---

## Component Composition

### Example: Composing Components

```typescript
// Presentational components
@Component({
  selector: 'app-card',
  standalone: true,
  template: `
    <div class="card">
      <ng-content />
    </div>
  `
})
export class CardComponent { }

@Component({
  selector: 'app-button',
  standalone: true,
  template: `
    <button [disabled]="disabled" [class]="variant">
      <ng-content />
    </button>
  `
})
export class ButtonComponent {
  @Input() disabled = false;
  @Input() variant: 'primary' | 'secondary' = 'primary';
}

// Composed component
@Component({
  selector: 'app-user-profile',
  standalone: true,
  imports: [CardComponent, ButtonComponent],
  template: `
    <app-card>
      <h2>{{ user.name }}</h2>
      <p>{{ user.email }}</p>
      <app-button variant="primary" (click)="edit()">Edit</app-button>
      <app-button variant="secondary" (click)="delete()">Delete</app-button>
    </app-card>
  `
})
export class UserProfileComponent {
  @Input({ required: true }) user!: User;
  @Output() edited = new EventEmitter<void>();
  @Output() deleted = new EventEmitter<void>();

  edit(): void {
    this.edited.emit();
  }

  delete(): void {
    this.deleted.emit();
  }
}
```

---

## Best Practices

### ✅ DO

1. **Use standalone components** as default
2. **Use OnPush change detection** for performance
3. **Use signals** for reactive state
4. **Keep components small** (under 200 lines)
5. **Separate smart and presentational** components
6. **Use trackBy** with ngFor
7. **Use required inputs** for mandatory data
8. **Use computed()** for derived values
9. **Use effect()** for side effects
10. **Import only what you need** in standalone components

### ❌ DON'T

1. **Don't put business logic** in components
2. **Don't make HTTP calls** directly in components
3. **Don't use component inheritance** (use composition)
4. **Don't forget trackBy** in ngFor
5. **Don't mutate objects** with OnPush
6. **Don't call functions** in templates
7. **Don't create large monolithic** components
8. **Don't use @ViewChild** before AfterViewInit

### Code Examples

#### ✅ GOOD: Small, focused component

```typescript
@Component({
  selector: 'app-user-card',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div class="card">
      <h3>{{ user().name }}</h3>
      <p>{{ user().email }}</p>
    </div>
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserCardComponent {
  readonly user = input.required<User>();
}
```

#### ❌ BAD: Logic in template

```typescript
// ❌ BAD
@Component({
  template: `
    <p>{{ calculateTotal(items) }}</p>  <!-- Function call in template -->
  `
})

// ✅ GOOD
@Component({
  template: `
    <p>{{ total() }}</p>  <!-- Signal -->
  `
})
export class GoodComponent {
  readonly items = signal<Item[]>([]);
  readonly total = computed(() =>
    this.items().reduce((sum, item) => sum + item.price, 0)
  );
}
```

#### ✅ GOOD: TrackBy function

```typescript
@Component({
  template: `
    @for (user of users(); track user.id) {
      <app-user-card [user]="user" />
    }
  `
})
export class UserListComponent {
  readonly users = signal<User[]>([]);
}
```

---

## Summary

| Concept | Recommendation |
|---------|---------------|
| **Component Type** | Standalone components (Angular 17+) |
| **State Management** | Signals (signal, computed, effect) |
| **Change Detection** | OnPush strategy |
| **Input/Output** | Use required inputs, typed outputs |
| **Lifecycle** | Use DestroyRef + takeUntilDestroyed |
| **Communication** | @Input/@Output for parent-child, services for siblings |
| **Size** | < 200 lines per component |
| **Composition** | Prefer composition over inheritance |

---

## Related Documentation

- See [architecture/layers.md](../architecture/layers.md) for component layers
- See [state-management/signals.md](../state-management/signals.md) for signals details
- See [services.md](./services.md) for service patterns
- See [../testing/unit-tests.md](../testing/unit-tests.md) for component testing
