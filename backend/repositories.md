# JPA Repositories

## Overview
Repositories provide data access layer using Spring Data JPA. Each entity has **one repository** that can contain multiple query methods.

## Basic Repository Structure

```java
public interface UserRepository extends JpaRepository<User, Long> {

    Optional<User> findByCode(String code);

    boolean existsByCode(String code);

    void deleteByCodeAndUsers_UserEntity_UsernameAndUsers_CreatedByIsTrue(String code, String username);

    @Query("SELECT h.code FROM User h")
    List<String> getUserCodes();
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

## @Query Annotation

For complex queries that can't be expressed with method names:

### JPQL Queries

```java
@Query("SELECT h FROM User h WHERE h.code = :code")
Optional<User> findByUserCode(@Param("code") String code);

@Query("SELECT h.code FROM User h")
List<String> getUserCodes();

@Query("SELECT h FROM User h JOIN h.users u WHERE u.username = :username")
List<User> findUsersByUsername(@Param("username") String username);
```

### Native SQL Queries

```java
@Query(value = "SELECT * FROM user WHERE code = :code", nativeQuery = true)
Optional<User> findByCodeNative(@Param("code") String code);

@Query(value = "SELECT h.* FROM user h " +
               "INNER JOIN user_user uh ON h.id = uh.user_id " +
               "WHERE uh.user_id = :userId",
       nativeQuery = true)
List<User> findUsersByUserIdNative(@Param("userId") Long userId);
```

### Modifying Queries

For UPDATE/DELETE operations:

```java
@Modifying
@Query("UPDATE User h SET h.name = :name WHERE h.code = :code")
int updateUserName(@Param("code") String code, @Param("name") String name);

@Modifying
@Query("DELETE FROM User h WHERE h.createdAt < :date")
int deleteOldUsers(@Param("date") LocalDateTime date);
```

**Note**: Use `@Modifying` with `@Transactional` in service methods.

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

## Custom Repository Implementation

For complex logic that can't be expressed in queries:

### 1. Create Custom Interface
```java
public interface UserRepositoryCustom {
    List<User> findUsersWithComplexLogic(String criteria);
}
```

### 2. Implement Custom Interface
```java
public class UserRepositoryImpl implements UserRepositoryCustom {

    @PersistenceContext
    private EntityManager entityManager;

    @Override
    public List<User> findUsersWithComplexLogic(String criteria) {
        CriteriaBuilder cb = entityManager.getCriteriaBuilder();
        CriteriaQuery<User> query = cb.createQuery(User.class);
        // Complex criteria logic
        return entityManager.createQuery(query).getResultList();
    }
}
```

### 3. Extend Both Interfaces
```java
public interface UserRepository extends JpaRepository<User, Long>, UserRepositoryCustom {
    // Standard queries
}
```

## Best Practices

1. ✅ One repository per entity
2. ✅ Use derived query methods when possible
3. ✅ Use `@Query` for complex queries
4. ✅ Return `Optional<T>` for single results
5. ✅ Use `Page<T>` for paginated results
6. ✅ Use `@Param` for named parameters in `@Query`
7. ✅ Keep repository interfaces clean
8. ✅ Use projections for specific data needs
9. ❌ NO business logic in repositories
10. ❌ NO `@Transactional` in repositories (use in services)
11. ❌ NO complex logic in query methods (use custom implementation)

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

### Pattern 4: Complex Query
```java
// Repository
@Query("SELECT h FROM User h " +
       "JOIN h.users u " +
       "WHERE u.username = :username " +
       "AND u.createdBy = true")
List<User> findUsersCreatedByUser(@Param("username") String username);
```

## Related Documentation
- See [entities.md](./entities.md) for entity structure
- See [services.md](./services.md) for repository usage
- See [database.md](../configuration/database.md) for JPA configuration
