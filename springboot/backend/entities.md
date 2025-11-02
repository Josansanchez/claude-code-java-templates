# JPA Entities

## Overview
Entities represent database models in the Application system. They should be clean, focused on database structure, and avoid business logic.

## Lombok Annotations for Entities

**IMPORTANT**: For entities WITH JPA relationships, do NOT use `@Data`. Use the following combination:

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@ToString(exclude = {"relatedEntities"})  // Exclude relationships
@EqualsAndHashCode(of = "id")  // Only compare by ID
@Entity
@Table(name = "table_name")
public class MyEntity {
    // fields
}
```

## Standard Entity Structure

```java
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@ToString(exclude = {"orders", "users"})  // Exclude relationships to avoid lazy loading
@EqualsAndHashCode(of = "id")
@Entity
@Table(name = "user")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id", nullable = false)
    private Long id;

    @Column(name = "name", length = 20, nullable = false)
    private String name;

    @Column(name = "code", length = 10, nullable = false, unique = true)
    private String code;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
    @JsonIgnore  // Avoid serialization loops
    private List<Order> orders;
}
```

## Important Guidelines

### Serializable
**IMPORTANT**: Only implement `Serializable` if:
- Using distributed cache (Redis, Hazelcast)
- Requiring distributed sessions
- Specific use cases require it

For most REST applications with Spring Boot, **Serializable is NOT necessary**.

### Lombok Best Practices
- **DO NOT** use `@Data` on entities with JPA relationships (causes issues with equals(), hashCode(), toString())
- **DO** use `@ToString(exclude = {"relations"})` to avoid lazy loading problems
- **DO** use `@EqualsAndHashCode(of = "id")` to compare only by ID
- **DO** use `@Builder` for complex object construction

### Validations
**IMPORTANT**: Validations belong in DTOs and Request objects, NOT in entities.

**In Entities**: Only database constraints
```java
@Column(name = "email", nullable = false, unique = true, length = 100)
private String email;
```

**In DTOs/Request Objects**: Business validations (see dtos.md)

## JPA Relationships

### @ManyToOne
```java
@ManyToOne(fetch = FetchType.LAZY)
@JoinColumn(name = "user_id", nullable = false)
@JsonIgnore
private User user;
```

### @OneToMany
```java
@OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true, fetch = FetchType.LAZY)
@JsonIgnore
private List<Order> orders;
```

### @ManyToMany
```java
@ManyToMany
@JoinTable(
    name = "user_user",
    joinColumns = @JoinColumn(name = "user_id"),
    inverseJoinColumns = @JoinColumn(name = "user_id")
)
private Set<User> users;
```

### Relationship Best Practices
- Use `fetch = FetchType.LAZY` for optimization
- Use `@JsonIgnore` on bidirectional relationships to avoid serialization loops
- Use `@JsonManagedReference` / `@JsonBackReference` when appropriate
- Use `cascade = CascadeType.ALL, orphanRemoval = true` for parent-child relationships

## Separation of Responsibilities
- **Entity**: Represents domain model and database structure
- **DTO/Request**: Validates user input and business rules

## Common Pitfalls to Avoid
- ❌ Using `@Data` on entities with relationships
- ❌ Adding business validations in entities
- ❌ Implementing Serializable unnecessarily
- ❌ Not excluding relationships in `@ToString`
- ❌ Using `equals()` and `hashCode()` with all fields
- ❌ Using `FetchType.EAGER` by default

## Related Documentation
- See [dtos.md](../backend/dtos.md) for DTO patterns
- See [repositories.md](../backend/repositories.md) for data access
- See [database.md](../configuration/database.md) for JPA configuration
