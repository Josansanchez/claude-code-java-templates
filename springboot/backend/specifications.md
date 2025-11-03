# JPA Specifications

## Overview

JPA Specifications provide a type-safe, composable way to build dynamic queries using the Criteria API. They are the **preferred** method for complex database queries in Spring Data JPA.

## Why Specifications Over @Query and Native SQL?

| Feature | Specifications | @Query / Native SQL |
|---------|---------------|---------------------|
| Type Safety | ✅ Compile-time checking | ❌ String-based, runtime errors |
| Refactoring | ✅ IDE support | ❌ Manual string updates |
| Composability | ✅ Easy to combine | ❌ Complex string concatenation |
| Reusability | ✅ Share logic across queries | ❌ Duplicate JPQL/SQL |
| Dynamic Queries | ✅ Build at runtime | ❌ Requires complex string building |
| Database Portability | ✅ JPA handles dialects | ❌ Native SQL is DB-specific |

## Directory Structure

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

## Basic Setup

### 1. Repository Interface

```java
package com.example.myapp.repositories.user;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.JpaSpecificationExecutor;
import com.example.myapp.entities.User;

public interface UserRepository extends JpaRepository<User, Long>,
                                         JpaSpecificationExecutor<User> {
    // Derived query methods for simple queries
    Optional<User> findByCode(String code);
    boolean existsByCode(String code);
}
```

### 2. Specifications Class

```java
package com.example.myapp.repositories.user;

import com.example.myapp.entities.User;
import org.springframework.data.jpa.domain.Specification;
import javax.persistence.criteria.*;
import java.time.LocalDateTime;

public class UserSpecifications {

    // Private constructor to prevent instantiation
    private UserSpecifications() {}

    public static Specification<User> hasCode(String code) {
        return (root, query, cb) ->
            code == null ? cb.conjunction() : cb.equal(root.get("code"), code);
    }

    public static Specification<User> hasName(String name) {
        return (root, query, cb) ->
            name == null ? cb.conjunction() : cb.equal(root.get("name"), name);
    }

    public static Specification<User> createdAfter(LocalDateTime date) {
        return (root, query, cb) ->
            date == null ? cb.conjunction() : cb.greaterThan(root.get("createdAt"), date);
    }
}
```

## Understanding Specification Components

Each Specification is a lambda with three parameters:

```java
(root, query, cb) -> {
    // root: The entity root (User in this case)
    // query: The CriteriaQuery being built
    // cb: CriteriaBuilder for creating predicates

    return predicate; // Your WHERE clause condition
}
```

### Root

Represents the entity being queried:

```java
root.get("fieldName")           // Access entity field
root.join("relationshipName")   // Join to related entity
root.fetch("relationshipName")  // Fetch join (eager loading)
```

### CriteriaBuilder (cb)

Creates predicates and expressions:

```java
cb.equal(x, y)              // x = y
cb.notEqual(x, y)           // x != y
cb.greaterThan(x, y)        // x > y
cb.lessThan(x, y)           // x < y
cb.like(x, pattern)         // x LIKE pattern
cb.isNull(x)                // x IS NULL
cb.isNotNull(x)             // x IS NOT NULL
cb.in(x).value(y)           // x IN (y)
cb.and(pred1, pred2)        // pred1 AND pred2
cb.or(pred1, pred2)         // pred1 OR pred2
cb.not(pred)                // NOT pred
```

## Common Specification Patterns

### Pattern 1: Equality Check

```java
public static Specification<User> hasCode(String code) {
    return (root, query, cb) ->
        code == null ? cb.conjunction() : cb.equal(root.get("code"), code);
}
```

**Note**: Return `cb.conjunction()` (always true) when parameter is null to allow combining with other specs.

### Pattern 2: Like / Contains

```java
public static Specification<User> nameContains(String keyword) {
    return (root, query, cb) ->
        keyword == null ? cb.conjunction() :
        cb.like(cb.lower(root.get("name")), "%" + keyword.toLowerCase() + "%");
}

public static Specification<User> nameStartsWith(String prefix) {
    return (root, query, cb) ->
        prefix == null ? cb.conjunction() :
        cb.like(root.get("name"), prefix + "%");
}
```

### Pattern 3: Date Range

```java
public static Specification<User> createdBetween(LocalDateTime start, LocalDateTime end) {
    return (root, query, cb) -> {
        if (start == null && end == null) {
            return cb.conjunction();
        }
        if (start != null && end != null) {
            return cb.between(root.get("createdAt"), start, end);
        }
        if (start != null) {
            return cb.greaterThanOrEqualTo(root.get("createdAt"), start);
        }
        return cb.lessThanOrEqualTo(root.get("createdAt"), end);
    };
}
```

### Pattern 4: In Collection

```java
public static Specification<User> codeIn(List<String> codes) {
    return (root, query, cb) ->
        codes == null || codes.isEmpty() ? cb.conjunction() :
        root.get("code").in(codes);
}
```

### Pattern 5: Boolean Flags

```java
public static Specification<User> isActive(Boolean active) {
    return (root, query, cb) -> {
        if (active == null) {
            return cb.conjunction();
        }
        return active ? cb.isTrue(root.get("active")) : cb.isFalse(root.get("active"));
    };
}
```

## Joins in Specifications

### One-to-Many Join

```java
public static Specification<User> hasUsername(String username) {
    return (root, query, cb) -> {
        if (username == null) {
            return cb.conjunction();
        }
        Join<User, UserEntity> userJoin = root.join("users", JoinType.INNER);
        return cb.equal(userJoin.get("username"), username);
    };
}
```

### Multiple Conditions on Join

```java
public static Specification<User> createdByUser(String username) {
    return (root, query, cb) -> {
        if (username == null) {
            return cb.conjunction();
        }
        Join<User, UserEntity> userJoin = root.join("users");
        return cb.and(
            cb.equal(userJoin.get("username"), username),
            cb.isTrue(userJoin.get("createdBy"))
        );
    };
}
```

### Left Join (Optional Relationship)

```java
public static Specification<Product> withCategoryName(String categoryName) {
    return (root, query, cb) -> {
        if (categoryName == null) {
            return cb.conjunction();
        }
        Join<Product, Category> categoryJoin = root.join("category", JoinType.LEFT);
        return cb.equal(categoryJoin.get("name"), categoryName);
    };
}
```

### Nested Joins

```java
public static Specification<Order> byCustomerCity(String city) {
    return (root, query, cb) -> {
        if (city == null) {
            return cb.conjunction();
        }
        Join<Order, Customer> customerJoin = root.join("customer");
        Join<Customer, Address> addressJoin = customerJoin.join("address");
        return cb.equal(addressJoin.get("city"), city);
    };
}
```

## Combining Specifications

### AND Combination

```java
@Service
public class UserService {

    private final UserRepository userRepository;

    @Transactional(readOnly = true)
    public List<UserDTO> searchUsers(String code, String name, LocalDateTime since) {
        Specification<User> spec = Specification.where(null);

        if (code != null) {
            spec = spec.and(UserSpecifications.hasCode(code));
        }
        if (name != null) {
            spec = spec.and(UserSpecifications.hasName(name));
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

### OR Combination

```java
@Transactional(readOnly = true)
public List<UserDTO> searchByCodeOrName(String code, String name) {
    Specification<User> spec = Specification.where(
        UserSpecifications.hasCode(code)
    ).or(UserSpecifications.hasName(name));

    return userRepository.findAll(spec).stream()
        .map(userMapper::toDTO)
        .collect(Collectors.toList());
}
```

### Complex AND/OR Logic

```java
@Transactional(readOnly = true)
public List<UserDTO> complexSearch(UserSearchCriteria criteria) {
    Specification<User> spec = Specification.where(null);

    // Must match code (if provided)
    if (criteria.getCode() != null) {
        spec = spec.and(UserSpecifications.hasCode(criteria.getCode()));
    }

    // Must match name OR username (if either provided)
    if (criteria.getName() != null || criteria.getUsername() != null) {
        Specification<User> orSpec = Specification.where(
            UserSpecifications.hasName(criteria.getName())
        ).or(UserSpecifications.hasUsername(criteria.getUsername()));

        spec = spec.and(orSpec);
    }

    // Must match date range
    if (criteria.getStartDate() != null || criteria.getEndDate() != null) {
        spec = spec.and(UserSpecifications.createdBetween(
            criteria.getStartDate(),
            criteria.getEndDate()
        ));
    }

    return userRepository.findAll(spec).stream()
        .map(userMapper::toDTO)
        .collect(Collectors.toList());
}
```

## Specifications with Pagination and Sorting

### Basic Pagination

```java
@Transactional(readOnly = true)
public Page<UserDTO> searchUsers(UserSearchCriteria criteria, Pageable pageable) {
    Specification<User> spec = buildSpecification(criteria);

    return userRepository.findAll(spec, pageable)
        .map(userMapper::toDTO);
}

// Helper method to build specification
private Specification<User> buildSpecification(UserSearchCriteria criteria) {
    Specification<User> spec = Specification.where(null);

    if (criteria.getCode() != null) {
        spec = spec.and(UserSpecifications.hasCode(criteria.getCode()));
    }
    if (criteria.getName() != null) {
        spec = spec.and(UserSpecifications.nameContains(criteria.getName()));
    }

    return spec;
}
```

### With Dynamic Sorting

```java
@GetMapping
public ResponseEntity<Page<UserDTO>> searchUsers(
        @RequestParam(required = false) String code,
        @RequestParam(required = false) String name,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "createdAt") String sortBy,
        @RequestParam(defaultValue = "DESC") String sortDir) {

    Sort sort = Sort.by(
        sortDir.equalsIgnoreCase("ASC") ? Sort.Direction.ASC : Sort.Direction.DESC,
        sortBy
    );
    Pageable pageable = PageRequest.of(page, size, sort);

    Specification<User> spec = Specification.where(null);
    if (code != null) {
        spec = spec.and(UserSpecifications.hasCode(code));
    }
    if (name != null) {
        spec = spec.and(UserSpecifications.nameContains(name));
    }

    Page<UserDTO> result = userRepository.findAll(spec, pageable)
        .map(userMapper::toDTO);

    return ResponseEntity.ok(result);
}
```

## Advanced Patterns

### Count with Specifications

```java
@Transactional(readOnly = true)
public long countActiveUsers(LocalDateTime since) {
    Specification<User> spec = UserSpecifications.isActive(true)
        .and(UserSpecifications.createdAfter(since));

    return userRepository.count(spec);
}
```

### Exists with Specifications

```java
@Transactional(readOnly = true)
public boolean hasActiveUsers(String code) {
    Specification<User> spec = UserSpecifications.hasCode(code)
        .and(UserSpecifications.isActive(true));

    return userRepository.count(spec) > 0;
}
```

### Distinct Results

```java
public static Specification<User> distinctResults() {
    return (root, query, cb) -> {
        query.distinct(true);
        return cb.conjunction();
    };
}

// Usage
Specification<User> spec = UserSpecifications.hasUsername("john")
    .and(UserSpecifications.distinctResults());
```

### Subqueries

```java
public static Specification<User> hasOrdersAbove(BigDecimal amount) {
    return (root, query, cb) -> {
        Subquery<Long> subquery = query.subquery(Long.class);
        Root<Order> orderRoot = subquery.from(Order.class);

        subquery.select(orderRoot.get("user").get("id"))
            .where(cb.greaterThan(orderRoot.get("total"), amount));

        return cb.in(root.get("id")).value(subquery);
    };
}
```

### Case-Insensitive Search

```java
public static Specification<User> nameIgnoreCase(String name) {
    return (root, query, cb) ->
        name == null ? cb.conjunction() :
        cb.equal(cb.lower(root.get("name")), name.toLowerCase());
}
```

### Fetch Joins (Avoid N+1)

```java
public static Specification<User> withUsersEager() {
    return (root, query, cb) -> {
        // Prevent duplicates when using fetch joins with pagination
        if (query.getResultType() != Long.class) {
            root.fetch("users", JoinType.LEFT);
        }
        return cb.conjunction();
    };
}
```

**Note**: Be careful with fetch joins and pagination. Consider using `@EntityGraph` in repository for better control.

## Complete Example: Product Search

### Entity

```java
@Entity
@Table(name = "products")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String code;
    private String name;
    private BigDecimal price;
    private Integer stock;
    private Boolean active;

    @ManyToOne
    @JoinColumn(name = "category_id")
    private Category category;

    private LocalDateTime createdAt;
}
```

### Specifications

```java
public class ProductSpecifications {

    private ProductSpecifications() {}

    public static Specification<Product> hasCode(String code) {
        return (root, query, cb) ->
            code == null ? cb.conjunction() : cb.equal(root.get("code"), code);
    }

    public static Specification<Product> nameContains(String keyword) {
        return (root, query, cb) ->
            keyword == null ? cb.conjunction() :
            cb.like(cb.lower(root.get("name")), "%" + keyword.toLowerCase() + "%");
    }

    public static Specification<Product> priceBetween(BigDecimal min, BigDecimal max) {
        return (root, query, cb) -> {
            if (min == null && max == null) {
                return cb.conjunction();
            }
            if (min != null && max != null) {
                return cb.between(root.get("price"), min, max);
            }
            if (min != null) {
                return cb.greaterThanOrEqualTo(root.get("price"), min);
            }
            return cb.lessThanOrEqualTo(root.get("price"), max);
        };
    }

    public static Specification<Product> stockGreaterThan(Integer minStock) {
        return (root, query, cb) ->
            minStock == null ? cb.conjunction() :
            cb.greaterThan(root.get("stock"), minStock);
    }

    public static Specification<Product> isActive(Boolean active) {
        return (root, query, cb) -> {
            if (active == null) {
                return cb.conjunction();
            }
            return active ? cb.isTrue(root.get("active")) : cb.isFalse(root.get("active"));
        };
    }

    public static Specification<Product> inCategory(String categoryName) {
        return (root, query, cb) -> {
            if (categoryName == null) {
                return cb.conjunction();
            }
            Join<Product, Category> categoryJoin = root.join("category", JoinType.INNER);
            return cb.equal(categoryJoin.get("name"), categoryName);
        };
    }

    public static Specification<Product> createdAfter(LocalDateTime date) {
        return (root, query, cb) ->
            date == null ? cb.conjunction() :
            cb.greaterThan(root.get("createdAt"), date);
    }
}
```

### Search DTO

```java
@Data
public class ProductSearchCriteria {
    private String code;
    private String keyword;
    private BigDecimal minPrice;
    private BigDecimal maxPrice;
    private Integer minStock;
    private Boolean active;
    private String category;
    private LocalDateTime createdAfter;
}
```

### Service

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;
    private final ProductMapper productMapper;

    @Transactional(readOnly = true)
    public Page<ProductDTO> searchProducts(ProductSearchCriteria criteria, Pageable pageable) {
        Specification<Product> spec = buildProductSpecification(criteria);

        return productRepository.findAll(spec, pageable)
            .map(productMapper::toDTO);
    }

    private Specification<Product> buildProductSpecification(ProductSearchCriteria criteria) {
        Specification<Product> spec = Specification.where(null);

        if (criteria.getCode() != null) {
            spec = spec.and(ProductSpecifications.hasCode(criteria.getCode()));
        }
        if (criteria.getKeyword() != null) {
            spec = spec.and(ProductSpecifications.nameContains(criteria.getKeyword()));
        }
        if (criteria.getMinPrice() != null || criteria.getMaxPrice() != null) {
            spec = spec.and(ProductSpecifications.priceBetween(
                criteria.getMinPrice(),
                criteria.getMaxPrice()
            ));
        }
        if (criteria.getMinStock() != null) {
            spec = spec.and(ProductSpecifications.stockGreaterThan(criteria.getMinStock()));
        }
        if (criteria.getActive() != null) {
            spec = spec.and(ProductSpecifications.isActive(criteria.getActive()));
        }
        if (criteria.getCategory() != null) {
            spec = spec.and(ProductSpecifications.inCategory(criteria.getCategory()));
        }
        if (criteria.getCreatedAfter() != null) {
            spec = spec.and(ProductSpecifications.createdAfter(criteria.getCreatedAfter()));
        }

        return spec;
    }
}
```

### Controller

```java
@RestController
@RequestMapping("/api/products")
@RequiredArgsConstructor
public class ProductController {

    private final ProductService productService;

    @GetMapping("/search")
    public ResponseEntity<Page<ProductDTO>> searchProducts(
            @ModelAttribute ProductSearchCriteria criteria,
            @PageableDefault(size = 20, sort = "createdAt", direction = Sort.Direction.DESC)
            Pageable pageable) {

        Page<ProductDTO> result = productService.searchProducts(criteria, pageable);
        return ResponseEntity.ok(result);
    }
}
```

## Testing Specifications

### Unit Test

```java
@ExtendWith(MockitoExtension.class)
class ProductSpecificationsTest {

    @Test
    void hasCode_shouldReturnCorrectPredicate() {
        // Given
        String code = "PROD-001";
        Specification<Product> spec = ProductSpecifications.hasCode(code);

        // Test that specification is created correctly
        assertNotNull(spec);
    }

    @Test
    void hasCode_withNullCode_shouldReturnConjunction() {
        // Given
        Specification<Product> spec = ProductSpecifications.hasCode(null);

        // Should not throw exception and allow combining with other specs
        assertNotNull(spec);
    }
}
```

### Integration Test

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
class ProductRepositoryTest {

    @Autowired
    private ProductRepository productRepository;

    @Test
    void findAll_withNameContainsSpecification_shouldReturnMatchingProducts() {
        // Given
        Product product1 = new Product();
        product1.setName("Laptop Computer");
        product1.setCode("PROD-001");
        productRepository.save(product1);

        Product product2 = new Product();
        product2.setName("Desktop Computer");
        product2.setCode("PROD-002");
        productRepository.save(product2);

        Product product3 = new Product();
        product3.setName("Mouse");
        product3.setCode("PROD-003");
        productRepository.save(product3);

        // When
        Specification<Product> spec = ProductSpecifications.nameContains("computer");
        List<Product> results = productRepository.findAll(spec);

        // Then
        assertThat(results).hasSize(2);
        assertThat(results).extracting(Product::getCode)
            .containsExactlyInAnyOrder("PROD-001", "PROD-002");
    }
}
```

## Best Practices

1. ✅ **Always handle null parameters** - Return `cb.conjunction()` when parameter is null
2. ✅ **One Specifications class per entity** - Keep them organized
3. ✅ **Use static methods** - Make them easy to use and combine
4. ✅ **Private constructor** - Prevent instantiation of utility class
5. ✅ **Descriptive method names** - `hasCode()`, `nameContains()`, `createdAfter()`
6. ✅ **Make specifications composable** - Each should work alone or combined
7. ✅ **Use joins carefully** - Consider N+1 query problems
8. ✅ **Distinct when needed** - Use with multiple joins to avoid duplicates
9. ✅ **Test specifications** - Write integration tests for complex specs
10. ❌ **NO business logic** - Keep specifications pure query logic
11. ❌ **NO database calls in specifications** - Only query building
12. ❌ **NO side effects** - Specifications should be pure functions

## Migration from @Query to Specifications

### Before (Native Query)

```java
@Query(value = "SELECT * FROM users WHERE code = :code AND active = true", nativeQuery = true)
List<User> findActiveUsersByCode(@Param("code") String code);
```

### After (Specifications)

```java
// Repository - no method declaration needed
public interface UserRepository extends JpaRepository<User, Long>,
                                         JpaSpecificationExecutor<User> {
}

// Specifications
public class UserSpecifications {
    public static Specification<User> hasCode(String code) {
        return (root, query, cb) ->
            code == null ? cb.conjunction() : cb.equal(root.get("code"), code);
    }

    public static Specification<User> isActive(Boolean active) {
        return (root, query, cb) ->
            active == null ? cb.conjunction() : cb.isTrue(root.get("active"));
    }
}

// Service
@Transactional(readOnly = true)
public List<UserDTO> findActiveUsersByCode(String code) {
    Specification<User> spec = UserSpecifications.hasCode(code)
        .and(UserSpecifications.isActive(true));

    return userRepository.findAll(spec).stream()
        .map(userMapper::toDTO)
        .collect(Collectors.toList());
}
```

## Common Pitfalls

### Pitfall 1: Not Handling Null Parameters

❌ **Wrong**:
```java
public static Specification<User> hasCode(String code) {
    return (root, query, cb) -> cb.equal(root.get("code"), code);
}
```

✅ **Correct**:
```java
public static Specification<User> hasCode(String code) {
    return (root, query, cb) ->
        code == null ? cb.conjunction() : cb.equal(root.get("code"), code);
}
```

### Pitfall 2: Fetch Join with Pagination

❌ **Wrong** - Can cause memory issues:
```java
public static Specification<User> withUsers() {
    return (root, query, cb) -> {
        root.fetch("users");
        return cb.conjunction();
    };
}
```

✅ **Correct** - Check query result type:
```java
public static Specification<User> withUsers() {
    return (root, query, cb) -> {
        if (query.getResultType() != Long.class) {
            root.fetch("users", JoinType.LEFT);
        }
        return cb.conjunction();
    };
}
```

### Pitfall 3: Cartesian Product with Multiple Joins

❌ **Wrong**:
```java
// Multiple fetch joins can cause Cartesian product
root.fetch("orders");
root.fetch("addresses");
```

✅ **Correct** - Use separate queries or `@EntityGraph`:
```java
@EntityGraph(attributePaths = {"orders", "addresses"})
List<User> findAll(Specification<User> spec);
```

## Related Documentation

- See [repositories.md](./repositories.md) for repository basics
- See [entities.md](./entities.md) for entity structure
- See [services.md](./services.md) for service layer usage
- See [database.md](../configuration/database.md) for JPA configuration
