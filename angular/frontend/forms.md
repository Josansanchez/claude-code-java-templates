# Forms - Angular 17+

## Overview

Angular provides two approaches for handling forms: **Reactive Forms** (recommended) and Template-Driven Forms. This guide focuses on **Reactive Forms** with TypeScript strict typing.

## Reactive Forms

### Setup

```typescript
// app.config.ts
import { ApplicationConfig } from '@angular/core';
import { provideRouter } from '@angular/router';
import { provideHttpClient } from '@angular/common/http';

export const appConfig: ApplicationConfig = {
  providers: [
    provideRouter(routes),
    provideHttpClient()
    // ReactiveFormsModule is imported per component in standalone apps
  ]
};
```

### Basic Form

```typescript
import { Component, inject } from '@angular/core';
import { ReactiveFormsModule, FormBuilder, Validators } from '@angular/forms';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-user-form',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div>
        <label for="name">Name:</label>
        <input id="name" type="text" formControlName="name" />
        @if (form.controls.name.invalid && form.controls.name.touched) {
          <div class="error">Name is required</div>
        }
      </div>

      <div>
        <label for="email">Email:</label>
        <input id="email" type="email" formControlName="email" />
        @if (form.controls.email.invalid && form.controls.email.touched) {
          <div class="error">
            @if (form.controls.email.errors?.['required']) {
              Email is required
            }
            @if (form.controls.email.errors?.['email']) {
              Invalid email format
            }
          </div>
        }
      </div>

      <button type="submit" [disabled]="form.invalid">Submit</button>
    </form>
  `
})
export class UserFormComponent {
  private readonly fb = inject(FormBuilder);

  readonly form = this.fb.group({
    name: ['', [Validators.required, Validators.minLength(3)]],
    email: ['', [Validators.required, Validators.email]]
  });

  onSubmit(): void {
    if (this.form.valid) {
      console.log('Form value:', this.form.value);
    } else {
      this.form.markAllAsTouched();
    }
  }
}
```

### Typed Forms (Recommended)

```typescript
import { Component, inject } from '@angular/core';
import { ReactiveFormsModule, FormBuilder, FormGroup, FormControl } from '@angular/forms';

interface UserForm {
  name: FormControl<string>;
  email: FormControl<string>;
  age: FormControl<number | null>;
}

@Component({
  selector: 'app-typed-form',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input type="text" formControlName="name" />
      <input type="email" formControlName="email" />
      <input type="number" formControlName="age" />
      <button type="submit">Submit</button>
    </form>
  `
})
export class TypedFormComponent {
  private readonly fb = inject(FormBuilder);

  readonly form = this.fb.group<UserForm>({
    name: this.fb.control('', { nonNullable: true }),
    email: this.fb.control('', { nonNullable: true }),
    age: this.fb.control<number | null>(null)
  });

  onSubmit(): void {
    const value = this.form.value;
    // value is typed as { name: string; email: string; age: number | null }
    console.log(value);
  }
}
```

---

## Form Validation

### Built-in Validators

```typescript
import { Validators } from '@angular/forms';

readonly form = this.fb.group({
  name: ['', [
    Validators.required,
    Validators.minLength(3),
    Validators.maxLength(50)
  ]],
  email: ['', [
    Validators.required,
    Validators.email
  ]],
  age: [null, [
    Validators.required,
    Validators.min(18),
    Validators.max(120)
  ]],
  website: ['', Validators.pattern(/^https?:\/\/.+/)],
  terms: [false, Validators.requiredTrue]
});
```

### Custom Validators

```typescript
// validators/custom-validators.ts
import { AbstractControl, ValidationErrors, ValidatorFn } from '@angular/forms';

/**
 * Validates that password contains at least one uppercase, lowercase, and number
 */
export function passwordStrengthValidator(): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const value = control.value;

    if (!value) {
      return null;
    }

    const hasUpperCase = /[A-Z]/.test(value);
    const hasLowerCase = /[a-z]/.test(value);
    const hasNumber = /[0-9]/.test(value);

    const valid = hasUpperCase && hasLowerCase && hasNumber;

    return valid ? null : { passwordStrength: true };
  };
}

/**
 * Validates that two fields match
 */
export function matchFieldsValidator(field1: string, field2: string): ValidatorFn {
  return (control: AbstractControl): ValidationErrors | null => {
    const value1 = control.get(field1)?.value;
    const value2 = control.get(field2)?.value;

    return value1 === value2 ? null : { fieldsMismatch: true };
  };
}

/**
 * Async validator: check if email exists
 */
export function emailExistsValidator(userService: UserService): AsyncValidatorFn {
  return (control: AbstractControl): Observable<ValidationErrors | null> => {
    if (!control.value) {
      return of(null);
    }

    return userService.checkEmailExists(control.value).pipe(
      map(exists => exists ? { emailExists: true } : null),
      catchError(() => of(null))
    );
  };
}
```

**Using custom validators:**

```typescript
import { passwordStrengthValidator, matchFieldsValidator } from './validators/custom-validators';

readonly form = this.fb.group({
  email: ['', [Validators.required, Validators.email]],
  password: ['', [
    Validators.required,
    Validators.minLength(8),
    passwordStrengthValidator()
  ]],
  confirmPassword: ['', Validators.required]
}, {
  validators: matchFieldsValidator('password', 'confirmPassword')
});
```

### Displaying Validation Errors

```typescript
@Component({
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div>
        <input type="password" formControlName="password" />
        <app-field-error [control]="form.controls.password" />
      </div>

      <div>
        <input type="password" formControlName="confirmPassword" />
        @if (form.errors?.['fieldsMismatch'] && form.touched) {
          <div class="error">Passwords do not match</div>
        }
      </div>
    </form>
  `
})
export class RegistrationFormComponent {
  readonly form = this.fb.group({
    password: ['', [Validators.required, passwordStrengthValidator()]],
    confirmPassword: ['', Validators.required]
  }, {
    validators: matchFieldsValidator('password', 'confirmPassword')
  });

  onSubmit(): void {
    if (this.form.valid) {
      console.log(this.form.value);
    }
  }
}
```

**Reusable error component:**

```typescript
// shared/components/field-error/field-error.component.ts
@Component({
  selector: 'app-field-error',
  standalone: true,
  imports: [CommonModule],
  template: `
    @if (control.invalid && (control.dirty || control.touched)) {
      <div class="error">
        @if (control.errors?.['required']) {
          This field is required
        }
        @if (control.errors?.['email']) {
          Invalid email format
        }
        @if (control.errors?.['minlength']) {
          Minimum length is {{ control.errors?.['minlength'].requiredLength }}
        }
        @if (control.errors?.['passwordStrength']) {
          Password must contain uppercase, lowercase, and number
        }
      </div>
    }
  `
})
export class FieldErrorComponent {
  @Input({ required: true }) control!: FormControl;
}
```

---

## Nested Forms

### FormGroup within FormGroup

```typescript
readonly form = this.fb.group({
  personalInfo: this.fb.group({
    firstName: ['', Validators.required],
    lastName: ['', Validators.required],
    dateOfBirth: [null as Date | null]
  }),
  address: this.fb.group({
    street: [''],
    city: ['', Validators.required],
    zipCode: ['', Validators.required],
    country: ['', Validators.required]
  }),
  contact: this.fb.group({
    email: ['', [Validators.required, Validators.email]],
    phone: ['']
  })
});
```

**Template:**

```html
<form [formGroup]="form" (ngSubmit)="onSubmit()">
  <div formGroupName="personalInfo">
    <input type="text" formControlName="firstName" placeholder="First Name" />
    <input type="text" formControlName="lastName" placeholder="Last Name" />
  </div>

  <div formGroupName="address">
    <input type="text" formControlName="street" placeholder="Street" />
    <input type="text" formControlName="city" placeholder="City" />
    <input type="text" formControlName="zipCode" placeholder="Zip Code" />
    <input type="text" formControlName="country" placeholder="Country" />
  </div>

  <div formGroupName="contact">
    <input type="email" formControlName="email" placeholder="Email" />
    <input type="tel" formControlName="phone" placeholder="Phone" />
  </div>

  <button type="submit">Submit</button>
</form>
```

---

## Dynamic Forms (FormArray)

### Managing Arrays of Form Controls

```typescript
import { FormArray } from '@angular/forms';

@Component({
  selector: 'app-task-form',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div formArrayName="tasks">
        @for (task of tasks.controls; track $index; let i = $index) {
          <div [formGroupName]="i">
            <input type="text" formControlName="title" placeholder="Task title" />
            <input type="checkbox" formControlName="completed" />
            <button type="button" (click)="removeTask(i)">Remove</button>
          </div>
        }
      </div>

      <button type="button" (click)="addTask()">Add Task</button>
      <button type="submit">Submit</button>
    </form>
  `
})
export class TaskFormComponent {
  private readonly fb = inject(FormBuilder);

  readonly form = this.fb.group({
    tasks: this.fb.array([
      this.createTaskFormGroup()
    ])
  });

  get tasks(): FormArray {
    return this.form.get('tasks') as FormArray;
  }

  private createTaskFormGroup(): FormGroup {
    return this.fb.group({
      title: ['', Validators.required],
      completed: [false]
    });
  }

  addTask(): void {
    this.tasks.push(this.createTaskFormGroup());
  }

  removeTask(index: number): void {
    this.tasks.removeAt(index);
  }

  onSubmit(): void {
    console.log('Tasks:', this.form.value);
  }
}
```

---

## Form State with Signals

```typescript
import { Component, signal, computed } from '@angular/core';
import { ReactiveFormsModule, FormBuilder, Validators } from '@angular/forms';

@Component({
  selector: 'app-signal-form',
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input type="text" formControlName="name" />
      <input type="email" formControlName="email" />

      @if (isLoading()) {
        <p>Submitting...</p>
      }

      @if (error()) {
        <div class="error">{{ error() }}</div>
      }

      <button type="submit" [disabled]="form.invalid || isLoading()">
        {{ submitButtonText() }}
      </button>
    </form>
  `
})
export class SignalFormComponent {
  private readonly fb = inject(FormBuilder);
  private readonly userService = inject(UserService);

  readonly form = this.fb.group({
    name: ['', Validators.required],
    email: ['', [Validators.required, Validators.email]]
  });

  readonly isLoading = signal(false);
  readonly error = signal<string | null>(null);

  readonly submitButtonText = computed(() =>
    this.isLoading() ? 'Submitting...' : 'Submit'
  );

  onSubmit(): void {
    if (this.form.invalid) {
      this.form.markAllAsTouched();
      return;
    }

    this.isLoading.set(true);
    this.error.set(null);

    this.userService.createUser(this.form.value).subscribe({
      next: () => {
        this.isLoading.set(false);
        this.form.reset();
      },
      error: (err) => {
        this.error.set(err.message);
        this.isLoading.set(false);
      }
    });
  }
}
```

---

## Best Practices

### ✅ DO

1. **Use Reactive Forms** for complex forms
2. **Enable strict typing** with typed FormGroups
3. **Use FormBuilder** for cleaner code
4. **Create custom validators** for business rules
5. **Mark as touched on submit** to show all errors
6. **Use signals** for form state (loading, error)
7. **Extract reusable validation** components
8. **Disable submit button** when form is invalid

### ❌ DON'T

1. **Don't use template-driven forms** for complex scenarios
2. **Don't forget to mark fields as touched** on submit
3. **Don't put business logic** in templates
4. **Don't mutate form values** directly
5. **Don't forget validation feedback**

### Code Examples

#### ✅ GOOD: Clean form with FormBuilder

```typescript
readonly form = this.fb.group({
  name: ['', Validators.required],
  email: ['', [Validators.required, Validators.email]]
});
```

#### ❌ BAD: Manual form creation

```typescript
// ❌ Avoid this approach
readonly form = new FormGroup({
  name: new FormControl('', Validators.required),
  email: new FormControl('', [Validators.required, Validators.email])
});
```

#### ✅ GOOD: Typed form

```typescript
interface UserForm {
  name: FormControl<string>;
  email: FormControl<string>;
}

readonly form = this.fb.group<UserForm>({
  name: this.fb.control('', { nonNullable: true }),
  email: this.fb.control('', { nonNullable: true })
});
```

---

## Summary

| Feature | Recommendation |
|---------|---------------|
| **Form Type** | Reactive Forms |
| **Typing** | Strongly typed FormGroups |
| **Builder** | Use FormBuilder |
| **Validation** | Built-in + custom validators |
| **Error Display** | Show errors on touch/submit |
| **State** | Use signals for loading/error |
| **Arrays** | FormArray for dynamic fields |

---

## Related Documentation

- See [components.md](./components.md) for component patterns
- See [services.md](./services.md) for form submission
- See [../testing/unit-tests.md](../testing/unit-tests.md) for form testing
