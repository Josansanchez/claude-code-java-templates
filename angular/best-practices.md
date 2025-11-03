# Best Practices - Angular Application

## Comprehensive list of 50+ best practices for Angular 17+ applications.

## Architecture and Organization (8)

1. **Standalone Components First**: Use standalone components as default, modules only when necessary
2. **Feature-Based Structure**: Organize code by features, not by technical types
3. **One Component Per File**: Each component in its own file with co-located template and styles
4. **Core Module for Singletons**: Use core directory for app-wide singleton services
5. **Shared Module for Reusables**: Shared components should be standalone and feature-independent
6. **Lazy Loading**: Load feature routes lazily for better performance
7. **Smart vs Presentational**: Separate container (smart) from presentational (dumb) components
8. **Module Boundaries**: Features should not import from other features

## Component Best Practices (10)

9. **Use OnPush Change Detection**: Set `changeDetection: ChangeDetectionStrategy.OnPush` for better performance
10. **Leverage Signals**: Use Angular signals for reactive state management
11. **Avoid Logic in Templates**: Keep templates simple, move logic to component class or pipes
12. **Use TrackBy with ngFor**: Always provide trackBy function for lists
13. **Unsubscribe from Observables**: Use async pipe or takeUntilDestroyed() to prevent memory leaks
14. **Component Size**: Keep components under 200 lines, extract to child components if larger
15. **Input/Output Naming**: Use descriptive names, avoid generic names like 'data' or 'value'
16. **Standalone Imports**: Import only what you need in standalone components
17. **ViewChild with read**: Specify `read` option when querying for specific types
18. **Avoid Component Inheritance**: Prefer composition over inheritance

## Services and Dependency Injection (8)

19. **ProvidedIn Root**: Use `providedIn: 'root'` for singleton services
20. **Single Responsibility**: Each service should have one clear purpose
21. **API Service Separation**: Separate HTTP calls from business logic services
22. **Inject in Constructor**: Use constructor injection, not field injection
23. **Use RxJS Operators**: Leverage RxJS operators for data transformation
24. **Error Handling**: Implement global error handling with interceptors
25. **Service Naming**: Suffix with `.service.ts`, be specific (e.g., `user-api.service.ts`)
26. **Avoid Circular Dependencies**: Restructure code to prevent circular imports

## State Management (6)

27. **Use Signals for Simple State**: Leverage built-in signals for local component state
28. **Computed Values**: Use `computed()` for derived state
29. **Effects for Side Effects**: Use `effect()` for reacting to signal changes
30. **Services for Shared State**: Use services with signals for shared state
31. **NgRx for Complex State**: Use NgRx only when state management is complex
32. **Immutable Updates**: Always update state immutably

## TypeScript and Types (6)

33. **Strict Mode**: Enable TypeScript strict mode in tsconfig.json
34. **Interface Over Type**: Prefer interfaces for object shapes, types for unions
35. **Avoid Any**: Never use `any`, use `unknown` if type is truly unknown
36. **Typed Forms**: Use strongly typed reactive forms
37. **Explicit Return Types**: Always specify return types for public methods
38. **Use Enums**: Use enums for fixed sets of values

## Forms (5)

39. **Reactive Forms**: Prefer reactive forms over template-driven
40. **Typed Form Groups**: Use typed FormGroup, FormControl in Angular 14+
41. **Validators**: Create custom validators for complex validations
42. **Form Builders**: Use FormBuilder for cleaner form creation
43. **Mark as Touched**: Mark fields as touched on submit for validation display

## Routing (5)

44. **Lazy Load Routes**: Use loadChildren for feature routes
45. **Route Guards**: Implement canActivate, canDeactivate guards
46. **Resolvers for Data**: Use resolvers to pre-fetch data before navigation
47. **Named Outlets**: Use named router outlets for complex layouts
48. **Route Params as Observables**: Subscribe to params as observables for dynamic updates

## Performance (8)

49. **OnPush Change Detection**: Use OnPush strategy everywhere possible
50. **Virtual Scrolling**: Use CDK virtual scrolling for large lists
51. **Lazy Load Images**: Implement lazy loading for images
52. **Pure Pipes**: Make custom pipes pure for better performance
53. **Avoid Function Calls in Templates**: Bind to properties, not function calls
54. **Optimize Bundle Size**: Use source-map-explorer to analyze bundles
55. **Preload Strategies**: Configure preloading for faster navigation
56. **Memoization**: Cache expensive computations with memoization

## Security (6)

57. **Sanitize User Input**: Use DomSanitizer for dynamic content
58. **Avoid innerHTML**: Use textContent or Angular's sanitization
59. **HTTP Interceptors**: Implement authentication and error interceptors
60. **Environment Variables**: Store API URLs and secrets in environment files
61. **CSRF Protection**: Implement CSRF tokens for state-changing operations
62. **Content Security Policy**: Configure CSP headers

## Testing (5)

63. **Unit Test Components**: Write unit tests for all components and services
64. **Test Coverage**: Maintain at least 80% code coverage
65. **Mock HTTP**: Use HttpClientTestingModule for HTTP testing
66. **Test User Interactions**: Test button clicks, form submissions
67. **E2E Tests**: Write E2E tests for critical user journeys

## Code Quality (5)

68. **ESLint + Prettier**: Use linting and formatting tools
69. **Consistent Naming**: Follow Angular naming conventions
70. **JSDoc Comments**: Document public APIs with JSDoc
71. **Code Reviews**: Require code reviews before merging
72. **Git Commit Conventions**: Use Conventional Commits

---

## Quick Reference by Category

### DO ✅
- Use standalone components
- Leverage Angular signals
- Enable TypeScript strict mode
- Use OnPush change detection
- Implement lazy loading
- Use reactive forms
- Inject in constructor
- Unsubscribe from observables (async pipe or takeUntilDestroyed)
- Write unit tests
- Use ESLint and Prettier
- Follow Angular style guide
- Keep components small (< 200 lines)
- Use trackBy with ngFor
- Sanitize user input
- Use environment variables

### DON'T ❌
- Use modules for everything (use standalone)
- Use `any` type
- Call functions in templates
- Forget to unsubscribe from observables
- Use template-driven forms for complex forms
- Put business logic in templates
- Use component inheritance
- Expose services publicly from features
- Use `innerHTML` without sanitization
- Hardcode API URLs
- Skip unit tests
- Ignore ESLint warnings
- Create circular dependencies
- Use field injection
- Forget trackBy in ngFor

## Angular Style Guide Compliance

This template follows the [Official Angular Style Guide](https://angular.dev/style-guide):

- ✅ File naming conventions (kebab-case)
- ✅ Component class naming (PascalCase, suffix with 'Component')
- ✅ Service class naming (PascalCase, suffix with 'Service')
- ✅ Single responsibility principle
- ✅ Folder structure by feature
- ✅ Lazy loading implementation
- ✅ Symbol naming conventions

## Related Documentation

- See [architecture/project-structure.md](./architecture/project-structure.md) for directory organization
- See [architecture/naming-conventions.md](./architecture/naming-conventions.md) for detailed naming rules
- See [frontend/components.md](./frontend/components.md) for component patterns
- See [testing/unit-tests.md](./testing/unit-tests.md) for testing guidelines
