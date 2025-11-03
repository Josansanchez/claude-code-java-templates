# Best Practices - Spring Boot Application

## Comprehensive list of 54 best practices for the Spring Boot Application project.

## Architecture and Organization (5)

1. **One Method Public per File**: Each Controller and Service must have exactly one public method
2. **Organize by Resources**: Create directories by resource within each capa (user, product, auth, mqtt)
3. **Directories in Lowercase**: All directories must be in lowercase (user, product, auth, mqtt)
4. **Separation of Responsibilities**: Each class must have a clear and unique responsibility (SRP)
5. **API Versioning**: Include version in routes (`/api/v1/...`)

## Dependency Injection (3)

6. **Constructor Injection**: Use constructor injection, NOT `@Autowired` on fields
7. **Immutability**: Use `final` on injected fields
8. **NO @Autowired on Fields**: Always use constructor injection

## DTOs and Validations (3)

9. **DTOs**: NEVER expose entities directly in controllers
10. **Validation in DTOs**: Business validations in DTOs/Request objects, NOT in entities
11. **Validation in Multiple Layers**: Controller with `@Valid`, service with business logic

## Security (4)

12. **CORS Centralized**: Configure globally in `WebSecurityConfig`, NO `@CrossOrigin` on controllers
13. **Always @PreAuthorize**: Apply on protected endpoints
14. **PasswordEncoder**: ALWAYS use for encrypting passwords
15. **Environment Variables**: For secrets in production (JWT secret, DB password)

## Transactions and Database (6)

16. **@Transactional at Method Level**: NO at class level
17. **@Transactional(readOnly = true)**: For read-only operations
18. **Lazy Loading**: Use `FetchType.LAZY` by default in JPA relationships
19. **Database Migrations**: Use Flyway or Liquibase, NEVER `ddl-auto=update` in production
20. **ddl-auto: validate**: In production, only validate schema (set in application-prod.yml)
21. **Never Modify Applied Migrations**: Create new migration files, don't edit existing ones

## Database Queries with Specifications (5)

22. **Use Specifications**: For complex queries, NOT `@Query` or native SQL
23. **JpaSpecificationExecutor**: All repositories must extend this interface
24. **Specifications Classes**: Create `*Specifications.java` for each entity in same directory as repository
25. **Type-Safe Queries**: NO string-based JPQL or native SQL queries
26. **Composable Specifications**: Build dynamic queries by combining specifications with `and()` and `or()`

## Exception Handling (3)

27. **Custom Exceptions**: With `@ResponseStatus`, NO generic `RuntimeException`
28. **@ControllerAdvice**: Centralized handling, NO try-catch in controllers
29. **Optional.orElseThrow()**: Use correctly for handling resources not found

## Mapping (2)

30. **MapStruct or ModelMapper**: For entity/DTO mapping, NO manual mapping
31. **Mapper Components**: Create `@Component` or `@Mapper` reusable

## HTTP Codes (3)

32. **201 Created**: For successful POST (with `Location` header)
33. **204 No Content**: For DELETE without response body
34. **ResponseEntity Appropriate**: Use correct HTTP codes (200, 201, 204, 400, 404, etc.)

## Testing (5)

35. **@WebMvcTest**: For controller tests (without full context)
36. **@DataJpaTest**: For repository tests
37. **Mockito**: For service unit tests
38. **@SpringBootTest**: Only for complete integration tests
39. **Minimum 80% Coverage**: Focus on critical business logic

## Documentation (2)

40. **OpenAPI/Swagger**: Document API with `@Operation`, `@ApiResponses`
41. **Swagger UI**: Accessible at `/swagger-ui.html`

## Logging (2)

42. **@Slf4j from Lombok**: For logging
43. **Appropriate Levels**: INFO, DEBUG, ERROR according to environment

## Lombok (4)

44. **NO @Data on Entities**: With JPA relationships (use `@Getter`, `@Setter`, `@Builder`)
45. **@Data on DTOs**: Without relationships
46. **@ToString(exclude)**: Exclude relationships to avoid lazy loading issues
47. **@EqualsAndHashCode(of = "id")**: Only by ID in entities

## Configuration (3)

48. **YAML instead of Properties**: For better readability
49. **Profiles by Environment**: dev, test, prod
50. **NO Serializable**: Unless really necessary (distributed cache)

## Pagination (2)

51. **Pageable**: For GET ALL operations with pagination
52. **Page<DTO>**: Return instead of List for large datasets

## Private Methods (2)

53. **Allowed for Auxiliary Logic**: Validations, code generation, etc.
54. **NO for Mapping**: Use Mapper component

---

## Quick Reference by Category

### DO ✅
- One public method per file (Controller/Service)
- Constructor injection with `final`
- DTOs for API exposure
- Validations in DTOs
- Global CORS configuration
- `@PreAuthorize` for authorization
- Environment variables for secrets
- `@Transactional` at method level
- `@Transactional(readOnly = true)` for reads
- **JPA Specifications for complex queries**
- **Extend JpaSpecificationExecutor in repositories**
- **Create *Specifications.java classes**
- **Composable and type-safe queries**
- Custom exceptions with `@ResponseStatus`
- `@ControllerAdvice` for error handling
- MapStruct/ModelMapper for mapping
- Correct HTTP status codes
- `@WebMvcTest` for controller tests
- `@DataJpaTest` for repository tests
- Mockito for service tests
- OpenAPI documentation
- `@Slf4j` for logging
- `@Data` on DTOs (without relationships)
- `@Builder` on entities
- YAML configuration files
- Profiles per environment (dev, test, prod)
- Pageable for GET ALL
- **Flyway/Liquibase for database migrations**
- **ddl-auto: validate in production**
- **Version-controlled schema changes**

### DON'T ❌
- Multiple public methods per file
- `@Autowired` on fields
- Expose entities in controllers
- Validations in entities
- `@CrossOrigin` on controllers
- Hardcoded secrets
- `@Transactional` at class level
- **ddl-auto=update in production**
- **ddl-auto=create or create-drop in production**
- **Modify existing migration files after applying**
- **@Query annotations (use Specifications)**
- **Native SQL queries (use Specifications)**
- **JPQL string queries (use Specifications)**
- Generic `RuntimeException`
- try-catch in controllers
- Manual mapping in services
- Incorrect HTTP codes
- `@SpringBootTest` for unit tests
- `@Data` on entities with relationships
- Properties files (use YAML)
- Serializable unless necessary
- Return List for large datasets (use Page)

## Related Documentation
- See [checklist.md](./checklist.md) for feature implementation checklist
- See [backend/repositories.md](./backend/repositories.md) for repository patterns
- See [backend/specifications.md](./backend/specifications.md) for detailed Specifications guide
- See [setup.md](./setup.md) for complete development rules
