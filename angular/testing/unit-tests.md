# Unit Tests - Angular 17+

## Overview

Unit tests verify that individual components, services, and pipes work correctly in isolation. Angular uses **Jasmine** as the default testing framework with **Karma** as the test runner.

## Setup

### Test File Structure

```
src/app/features/user/
├── user-list.component.ts
├── user-list.component.spec.ts    # Test file
├── user.service.ts
└── user.service.spec.ts           # Test file
```

### Running Tests

```bash
# Run all tests
ng test

# Run tests once (CI mode)
ng test --watch=false

# Run tests with code coverage
ng test --code-coverage

# Run specific test file
ng test --include='**/*user.service.spec.ts'
```

---

## Testing Components

### Basic Component Test

```typescript
// user-card.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { UserCardComponent } from './user-card.component';

describe('UserCardComponent', () => {
  let component: UserCardComponent;
  let fixture: ComponentFixture<UserCardComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [UserCardComponent]  // Standalone component
    }).compileComponents();

    fixture = TestBed.createComponent(UserCardComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });

  it('should display user name', () => {
    const user: User = { id: '1', name: 'John Doe', email: 'john@example.com' };
    component.user = user;
    fixture.detectChanges();

    const compiled = fixture.nativeElement as HTMLElement;
    expect(compiled.querySelector('h3')?.textContent).toContain('John Doe');
  });

  it('should emit delete event', () => {
    spyOn(component.delete, 'emit');

    const user: User = { id: '1', name: 'John', email: 'john@example.com' };
    component.user = user;
    component.onDelete();

    expect(component.delete.emit).toHaveBeenCalledWith('1');
  });
});
```

### Testing Components with Signals

```typescript
// counter.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { CounterComponent } from './counter.component';

describe('CounterComponent', () => {
  let component: CounterComponent;
  let fixture: ComponentFixture<CounterComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [CounterComponent]
    }).compileComponents();

    fixture = TestBed.createComponent(CounterComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should initialize count to 0', () => {
    expect(component.count()).toBe(0);
  });

  it('should increment count', () => {
    component.increment();
    expect(component.count()).toBe(1);
  });

  it('should update computed signal', () => {
    expect(component.doubleCount()).toBe(0);

    component.increment();
    expect(component.doubleCount()).toBe(2);
  });

  it('should display count in template', () => {
    component.increment();
    fixture.detectChanges();

    const compiled = fixture.nativeElement as HTMLElement;
    expect(compiled.querySelector('p')?.textContent).toContain('Count: 1');
  });
});
```

### Testing Component with Dependencies

```typescript
// user-list.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { UserListComponent } from './user-list.component';
import { UserService } from '../services/user.service';
import { of } from 'rxjs';

describe('UserListComponent', () => {
  let component: UserListComponent;
  let fixture: ComponentFixture<UserListComponent>;
  let userService: jasmine.SpyObj<UserService>;

  beforeEach(async () => {
    const userServiceSpy = jasmine.createSpyObj('UserService', ['getUsers', 'deleteUser']);

    await TestBed.configureTestingModule({
      imports: [UserListComponent],
      providers: [
        { provide: UserService, useValue: userServiceSpy }
      ]
    }).compileComponents();

    fixture = TestBed.createComponent(UserListComponent);
    component = fixture.componentInstance;
    userService = TestBed.inject(UserService) as jasmine.SpyObj<UserService>;
  });

  it('should load users on init', () => {
    const mockUsers: User[] = [
      { id: '1', name: 'John', email: 'john@example.com' }
    ];
    userService.getUsers.and.returnValue(of(mockUsers));

    component.ngOnInit();

    expect(userService.getUsers).toHaveBeenCalled();
    expect(component.users().length).toBe(1);
  });

  it('should delete user', () => {
    const mockUsers: User[] = [
      { id: '1', name: 'John', email: 'john@example.com' }
    ];
    userService.getUsers.and.returnValue(of(mockUsers));
    userService.deleteUser.and.returnValue(of(void 0));

    component.ngOnInit();
    component.deleteUser('1');

    expect(userService.deleteUser).toHaveBeenCalledWith('1');
  });
});
```

---

## Testing Services

### Basic Service Test

```typescript
// user.service.spec.ts
import { TestBed } from '@angular/core/testing';
import { UserService } from './user.service';
import { UserApiService } from './user-api.service';
import { of, throwError } from 'rxjs';

describe('UserService', () => {
  let service: UserService;
  let userApiService: jasmine.SpyObj<UserApiService>;

  beforeEach(() => {
    const apiServiceSpy = jasmine.createSpyObj('UserApiService', [
      'getUsers',
      'createUser',
      'updateUser',
      'deleteUser'
    ]);

    TestBed.configureTestingModule({
      providers: [
        UserService,
        { provide: UserApiService, useValue: apiServiceSpy }
      ]
    });

    service = TestBed.inject(UserService);
    userApiService = TestBed.inject(UserApiService) as jasmine.SpyObj<UserApiService>;
  });

  it('should be created', () => {
    expect(service).toBeTruthy();
  });

  it('should normalize user data', (done) => {
    const input = { name: '  John  ', email: 'JOHN@EXAMPLE.COM  ' };
    const normalized = { name: 'John', email: 'john@example.com' };

    userApiService.createUser.and.returnValue(of({ id: '1', ...normalized }));

    service.createUser(input).subscribe(user => {
      expect(userApiService.createUser).toHaveBeenCalledWith(normalized);
      done();
    });
  });

  it('should handle errors', (done) => {
    userApiService.getUsers.and.returnValue(
      throwError(() => new Error('API Error'))
    );

    service.getUsers().subscribe({
      error: (error) => {
        expect(error.message).toBe('API Error');
        done();
      }
    });
  });
});
```

### Testing HTTP Services

```typescript
// user-api.service.spec.ts
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

  it('should handle HTTP errors', () => {
    service.getUsers().subscribe({
      next: () => fail('should have failed'),
      error: (error) => {
        expect(error.message).toContain('500');
      }
    });

    const req = httpMock.expectOne('/api/users');
    req.flush('Server error', { status: 500, statusText: 'Internal Server Error' });
  });
});
```

---

## Testing Forms

```typescript
// user-form.component.spec.ts
import { ComponentFixture, TestBed } from '@angular/core/testing';
import { ReactiveFormsModule } from '@angular/forms';
import { UserFormComponent } from './user-form.component';

describe('UserFormComponent', () => {
  let component: UserFormComponent;
  let fixture: ComponentFixture<UserFormComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [UserFormComponent, ReactiveFormsModule]
    }).compileComponents();

    fixture = TestBed.createComponent(UserFormComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it('should create form with controls', () => {
    expect(component.form.contains('name')).toBeTruthy();
    expect(component.form.contains('email')).toBeTruthy();
  });

  it('should mark name as invalid when empty', () => {
    const nameControl = component.form.get('name');
    expect(nameControl?.valid).toBeFalsy();

    nameControl?.setValue('John');
    expect(nameControl?.valid).toBeTruthy();
  });

  it('should validate email format', () => {
    const emailControl = component.form.get('email');

    emailControl?.setValue('invalid');
    expect(emailControl?.errors?.['email']).toBeTruthy();

    emailControl?.setValue('valid@example.com');
    expect(emailControl?.valid).toBeTruthy();
  });

  it('should disable submit button when form is invalid', () => {
    const compiled = fixture.nativeElement as HTMLElement;
    const button = compiled.querySelector('button[type="submit"]') as HTMLButtonElement;

    expect(button.disabled).toBeTruthy();

    component.form.patchValue({
      name: 'John',
      email: 'john@example.com'
    });
    fixture.detectChanges();

    expect(button.disabled).toBeFalsy();
  });

  it('should call onSubmit when form is valid', () => {
    spyOn(component, 'onSubmit');

    component.form.patchValue({
      name: 'John',
      email: 'john@example.com'
    });

    const compiled = fixture.nativeElement as HTMLElement;
    const form = compiled.querySelector('form') as HTMLFormElement;
    form.dispatchEvent(new Event('submit'));

    expect(component.onSubmit).toHaveBeenCalled();
  });
});
```

---

## Testing Pipes

```typescript
// truncate.pipe.spec.ts
import { TruncatePipe } from './truncate.pipe';

describe('TruncatePipe', () => {
  let pipe: TruncatePipe;

  beforeEach(() => {
    pipe = new TruncatePipe();
  });

  it('should create', () => {
    expect(pipe).toBeTruthy();
  });

  it('should truncate long text', () => {
    const longText = 'This is a very long text that should be truncated';
    const result = pipe.transform(longText, 10);
    expect(result).toBe('This is a ...');
  });

  it('should not truncate short text', () => {
    const shortText = 'Short';
    const result = pipe.transform(shortText, 10);
    expect(result).toBe('Short');
  });

  it('should handle empty string', () => {
    const result = pipe.transform('', 10);
    expect(result).toBe('');
  });
});
```

---

## Testing Guards

```typescript
// auth.guard.spec.ts
import { TestBed } from '@angular/core/testing';
import { Router } from '@angular/router';
import { authGuard } from './auth.guard';
import { AuthService } from '../services/auth.service';

describe('authGuard', () => {
  let authService: jasmine.SpyObj<AuthService>;
  let router: jasmine.SpyObj<Router>;

  beforeEach(() => {
    const authServiceSpy = jasmine.createSpyObj('AuthService', ['isAuthenticated']);
    const routerSpy = jasmine.createSpyObj('Router', ['createUrlTree']);

    TestBed.configureTestingModule({
      providers: [
        { provide: AuthService, useValue: authServiceSpy },
        { provide: Router, useValue: routerSpy }
      ]
    });

    authService = TestBed.inject(AuthService) as jasmine.SpyObj<AuthService>;
    router = TestBed.inject(Router) as jasmine.SpyObj<Router>;
  });

  it('should allow authenticated users', () => {
    authService.isAuthenticated.and.returnValue(true);

    const result = TestBed.runInInjectionContext(() =>
      authGuard({} as any, {} as any)
    );

    expect(result).toBe(true);
  });

  it('should redirect unauthenticated users', () => {
    authService.isAuthenticated.and.returnValue(false);
    router.createUrlTree.and.returnValue({} as any);

    TestBed.runInInjectionContext(() =>
      authGuard({} as any, { url: '/dashboard' } as any)
    );

    expect(router.createUrlTree).toHaveBeenCalledWith(
      ['/login'],
      { queryParams: { returnUrl: '/dashboard' } }
    );
  });
});
```

---

## Best Practices

### ✅ DO

1. **Write tests for all components and services**
2. **Test user interactions** (clicks, inputs)
3. **Mock dependencies** with jasmine.createSpyObj
4. **Use HttpClientTestingModule** for HTTP tests
5. **Test edge cases** and error scenarios
6. **Aim for 80%+ code coverage**
7. **Keep tests simple and focused**
8. **Use descriptive test names**

### ❌ DON'T

1. **Don't test implementation details**
2. **Don't test Angular framework** (it's already tested)
3. **Don't make tests depend** on each other
4. **Don't forget to verify HTTP mocks** (httpMock.verify())
5. **Don't skip error cases**

### Code Examples

#### ✅ GOOD: Testing behavior

```typescript
it('should display user name after loading', () => {
  userService.getUser.and.returnValue(of(mockUser));
  component.ngOnInit();
  fixture.detectChanges();

  expect(compiled.querySelector('h2')?.textContent).toContain('John Doe');
});
```

#### ❌ BAD: Testing implementation

```typescript
// ❌ BAD: Testing private implementation details
it('should call private method', () => {
  spyOn(component as any, 'privateMethod');
  component.publicMethod();
  expect((component as any).privateMethod).toHaveBeenCalled();
});
```

---

## Summary

| What to Test | How |
|--------------|-----|
| **Components** | TestBed, ComponentFixture, spy on services |
| **Services** | TestBed, spy on dependencies |
| **HTTP** | HttpClientTestingModule, HttpTestingController |
| **Forms** | ReactiveFormsModule, test validity |
| **Pipes** | Direct instantiation, transform tests |
| **Guards** | TestBed.runInInjectionContext, spy on Router |

---

## Related Documentation

- See [components.md](../frontend/components.md) for component patterns
- See [services.md](../frontend/services.md) for service patterns
- See [forms.md](../frontend/forms.md) for form patterns
