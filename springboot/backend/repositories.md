# JPA Repositories

## Overview
Repositories provide data access layer using Spring Data JPA. Each entity has **one repository** that can contain multiple query methods.

## Basic Repository Structure

```java
public interface UserRepository extends JpaRepository<User, Long>,
                                         JpaSpecificationExecutor<User> {

    // Simple derived query methods
    Optional<User> findByCode(String code);

    boolean existsByCode(String code);

    // For complex queries, use Specifications instead
    // See UserSpecifications.java for complex query logic
}
```

## Directory Structure

Repositories are organized by resource in lowercase directories:
```
repositories/
├── user/
│   └── UserRepository.java
├── product/
│   └── ProductRepository.java
├── order/
│   └── OrderRepository.java
└── user/
    └── UserRepository.java
```

## Naming Conventions
- **Directory**: `repositories/<resource>/` (lowercase)
- **Interface**: `<Resource>Repository.java` (e.g., `UserRepository.java`)
- **One repository per entity**

## Extending JpaRepository

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // User = Entity type
    // Long = ID type

    // Inherited methods (no need to implement):
    // - save(entity)
    // - findById(id)
    // - findAll()
    // - deleteById(id)
    // - count()
    // - existsById(id)
}
```

## Spring Data JPA Query Methods

Spring Data JPA automatically implements methods based on naming conventions.

### Find Methods

```java
// Find by single field
Optional<User> findByCode(String code);
Optional<User> findByUsername(String username);

// Find by multiple fields
Optional<User> findByCodeAndName(String code, String name);

// Find all by field
List<User> findByUserId(Long userId);

// Find with case-insensitive
Optional<User> findByUsernameIgnoreCase(String username);

// Find with ordering
List<User> findByUserIdOrderByNameAsc(Long userId);
```

### Exists Methods

```java
// Check existence
boolean existsByCode(String code);
boolean existsByName(String name);
boolean existsByCodeAndUserId(String code, Long userId);
```

### Count Methods

```java
// Count entities
long countByUserId(Long userId);
long countByCreatedAtAfter(LocalDateTime date);
```

### Delete Methods

```java
// Delete by field
void deleteByCode(String code);

// Complex delete
void deleteByCodeAndUsers_UserEntity_UsernameAndUsers_CreatedByIsTrue(
    String code,
    String username
);
```

## JPA Specifications

For complex queries that can't be expressed with method names, use **JPA Specifications** instead of native queries or JPQL.

### Why Specifications?

✅ **Type-safe**: Compile-time checking prevents errors
✅ **Composable**: Combine multiple specifications
✅ **Reusable**: Share specification logic across queries
✅ **Maintainable**: Easier to refactor than string queries
✅ **Dynamic**: Build queries based on runtime conditions

### Extend JpaSpecificationExecutor

```java
public interface UserRepository extends JpaRepository<User, Long>,
                                         JpaSpecificationExecutor<User> {
    // Derived query methods
    Optional<User> findByCode(String code);
    boolean existsByCode(String code);
}
```

### Basic Specification Example

```java
public class UserSpecifications {

    public static Specification<User> hasCode(String code) {
        return (root, query, cb) -> cb.equal(root.get("code"), code);
    }

    public static Specification<User> hasName(String name) {
        return (root, query, cb) -> cb.equal(root.get("name"), name);
    }

    public static Specification<User> createdAfter(LocalDateTime date) {
        return (root, query, cb) -> cb.greaterThan(root.get("createdAt"), date);
    }
}
```

### Using Specifications in Service

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    @Transactional(readOnly = true)
    public List<UserDTO> findUsers(String code, LocalDateTime since) {
        Specification<User> spec = Specification.where(null);

        if (code != null) {
            spec = spec.and(UserSpecifications.hasCode(code));
        }
        if (since != null) {
            spec = spec.and(UserSpecifications.createdAfter(since));
        }

        return userRepository.findAll(spec).stream()
            .map(userMapper::toDTO)
            .collect(Collectors.toList());
    }
}
```

### Complex Specifications with Joins

```java
public class UserSpecifications {

    public static Specification<User> hasUsername(String username) {
        return (root, query, cb) -> {
            Join<User, UserEntity> userJoin = root.join("users");
            return cb.equal(userJoin.get("username"), username);
        };
    }

    public static Specification<User> createdByUser(String username) {
        return (root, query, cb) -> {
            Join<User, UserEntity> userJoin = root.join("users");
            return cb.and(
                cb.equal(userJoin.get("username"), username),
                cb.isTrue(userJoin.get("createdBy"))
            );
        };
    }
}
```

### Combining Specifications

```java
// In service
public List<UserDTO> searchUsers(UserSearchCriteria criteria) {
    Specification<User> spec = Specification.where(null);

    // Combine with AND
    if (criteria.getCode() != null) {
        spec = spec.and(UserSpecifications.hasCode(criteria.getCode()));
    }

    // Combine with OR
    if (criteria.getName() != null || criteria.getUsername() != null) {
        Specification<User> orSpec = Specification.where(null);
        if (criteria.getName() != null) {
            orSpec = orSpec.or(UserSpecifications.hasName(criteria.getName()));
        }
        if (criteria.getUsername() != null) {
            orSpec = orSpec.or(UserSpecifications.hasUsername(criteria.getUsername()));
        }
        spec = spec.and(orSpec);
    }

    return userRepository.findAll(spec).stream()
        .map(userMapper::toDTO)
        .collect(Collectors.toList());
}
```

### Specifications with Pagination

```java
@Transactional(readOnly = true)
public Page<UserDTO> searchUsers(UserSearchCriteria criteria, Pageable pageable) {
    Specification<User> spec = buildSpecification(criteria);

    return userRepository.findAll(spec, pageable)
        .map(userMapper::toDTO);
}
```

**Note**: For detailed examples and advanced patterns, see [specifications.md](./specifications.md)

## Query Method Keywords

Common keywords for building method names:

| Keyword | Example | JPQL snippet |
|---------|---------|--------------|
| And | findByNameAndCode | ... where x.name = ?1 and x.code = ?2 |
| Or | findByNameOrCode | ... where x.name = ?1 or x.code = ?2 |
| Is, Equals | findByCode | ... where x.code = ?1 |
| Between | findByCreatedAtBetween | ... where x.createdAt between ?1 and ?2 |
| LessThan | findByIdLessThan | ... where x.id < ?1 |
| GreaterThan | findByIdGreaterThan | ... where x.id > ?1 |
| Like | findByNameLike | ... where x.name like ?1 |
| StartingWith | findByNameStartingWith | ... where x.name like ?1% |
| EndingWith | findByNameEndingWith | ... where x.name like %?1 |
| Containing | findByNameContaining | ... where x.name like %?1% |
| OrderBy | findByNameOrderByCodeAsc | ... order by x.code asc |
| Not | findByNameNot | ... where x.name <> ?1 |
| In | findByCodeIn | ... where x.code in ?1 |
| NotIn | findByCodeNotIn | ... where x.code not in ?1 |
| True | findByActiveTrue | ... where x.active = true |
| False | findByActiveFalse | ... where x.active = false |
| IgnoreCase | findByNameIgnoreCase | ... where UPPER(x.name) = UPPER(?1) |

## Pagination and Sorting

```java
// With Pageable
Page<User> findByUserId(Long userId, Pageable pageable);

// With Sort
List<User> findByUserId(Long userId, Sort sort);

// Usage in service:
@Transactional(readOnly = true)
public Page<UserDTO> getAllUsers(Pageable pageable) {
    Page<User> page = userRepository.findAll(pageable);
    return userMapper.toPageDTO(page);
}
```

## Projections

Return only specific fields:

### Interface Projection

```java
public interface UserCodeProjection {
    String getCode();
    String getName();
}

// In repository
List<UserCodeProjection> findAllProjectedBy();
```

### Class-based Projection (DTO)

```java
@Query("SELECT new com.example.myapp.dto.UserNavBarDTO(h.code, h.name) " +
       "FROM User h")
List<UserNavBarDTO> findAllForNavBar();
```

## Directory Structure for Specifications

Organize specifications in the same directory as repositories:

```
repositories/
├── user/
│   ├── UserRepository.java
│   └── UserSpecifications.java
├── product/
│   ├── ProductRepository.java
│   └── ProductSpecifications.java
└── order/
    ├── OrderRepository.java
    └── OrderSpecifications.java
```

## Custom Repository Implementation (Advanced)

Only use custom implementations when Specifications are not sufficient (rare cases):

### 1. Create Custom Interface
```java
public interface UserRepositoryCustom {
    List<User> complexBatchOperation(List<String> codes);
}
```

### 2. Implement Custom Interface
```java
public class UserRepositoryImpl implements UserRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<User> complexBatchOperation(List<String> codes) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        Root<User> root = query.from(User.class);

        // Complex batch logic that can't be done with Specifications
        query.select(root).where(root.get("code").in(codes));

        return entityManager.createQuery(query).getResultList();
    }
}
```

### 3. Extend Both Interfaces
```java
public interface UserRepository extends JpaRepository<User, Long>,
                                         JpaSpecificationExecutor<User>,
                                         UserRepositoryCustom {
    // Standard query methods
}
```

**Note**: 99% of complex queries should use Specifications. Only use custom implementations for truly complex scenarios like batch operations or database-specific features.

## Best Practices

1. ✅ One repository per entity
2. ✅ Use derived query methods for simple queries
3. ✅ Use **Specifications** for complex queries (NOT @Query or native SQL)
4. ✅ Extend `JpaSpecificationExecutor` for dynamic queries
5. ✅ Create separate `*Specifications.java` classes for each entity
6. ✅ Return `Optional<T>` for single results
7. ✅ Use `Page<T>` for paginated results with Specifications
8. ✅ Keep repository interfaces clean
9. ✅ Use projections for specific data needs
10. ✅ Make Specifications reusable and composable
11. ❌ NO business logic in repositories
12. ❌ NO `@Transactional` in repositories (use in services)
13. ❌ NO native queries or @Query annotations (use Specifications)
14. ❌ NO JPQL strings (they're not type-safe)

## Common Patterns

### Pattern 1: Find with Optional
```java
// Repository
Optional<User> findByCode(String code);

// Service
public UserDTO getUser(String code) {
    return userRepository.findByCode(code)
        .map(userMapper::toDTO)
        .orElseThrow(() -> new ResourceNotFoundException("User not found"));
}
```

### Pattern 2: Exists Check
```java
// Repository
boolean existsByCode(String code);

// Service
if (userRepository.existsByCode(code)) {
    throw new ResourceAlreadyExistsException("User already exists");
}
```

### Pattern 3: Paginated Query
```java
// Repository
Page<User> findAll(Pageable pageable);

// Service
@Transactional(readOnly = true)
public Page<UserDTO> getAllUsers(Pageable pageable) {
    return userRepository.findAll(pageable)
        .map(userMapper::toDTO);
}
```

### Pattern 4: Complex Query with Specifications
```java
// Specifications class
public class UserSpecifications {
    public static Specification<User> createdByUser(String username) {
        return (root, query, cb) -> {
            Join<User, UserEntity> userJoin = root.join("users");
            return cb.and(
                cb.equal(userJoin.get("username"), username),
                cb.isTrue(userJoin.get("createdBy"))
            );
        };
    }
}

// Repository
public interface UserRepository extends JpaRepository<User, Long>,
                                         JpaSpecificationExecutor<User> {
    // No need to declare the method
}

// Service
@Transactional(readOnly = true)
public List<UserDTO> getUsersCreatedBy(String username) {
    Specification<User> spec = UserSpecifications.createdByUser(username);
    return userRepository.findAll(spec).stream()
        .map(userMapper::toDTO)
        .collect(Collectors.toList());
}
```

## Related Documentation
- See [entities.md](./entities.md) for entity structure
- See [services.md](./services.md) for repository usage
- See [database.md](../configuration/database.md) for JPA configuration
