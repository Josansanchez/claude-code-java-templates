# Caching Strategies

## Overview

Caching is a technique to store frequently accessed data in memory to improve application performance and reduce database load. This guide covers Spring Cache abstraction and Redis as a distributed cache provider.

## Why Use Caching?

### Benefits

✅ **Performance**: Reduce response time from seconds to milliseconds
✅ **Scalability**: Handle more requests with same infrastructure
✅ **Database Load**: Reduce expensive database queries
✅ **Cost**: Lower infrastructure costs by reducing database usage
✅ **User Experience**: Faster page loads and API responses

### Use Cases

- **Database query results** - Expensive joins, aggregations
- **API responses** - External API calls
- **Session data** - User sessions, shopping carts
- **Configuration** - Application settings, feature flags
- **Computed values** - Reports, statistics, calculations
- **Static content** - Product catalogs, categories

## Spring Cache vs Redis

| Feature | Spring Cache (Simple) | Redis |
|---------|----------------------|-------|
| **Storage** | In-memory (JVM heap) | External server |
| **Distribution** | Single instance | Distributed across instances |
| **Persistence** | Lost on restart | Can persist to disk |
| **Size Limit** | JVM heap size | GB to TB |
| **Eviction** | LRU, LFU, TTL | Advanced eviction policies |
| **Use Case** | Small apps, development | Production, microservices |
| **Setup** | Simple | Requires Redis server |
| **Recommendation** | Development/testing | ✅ **Production** |

## Spring Cache Abstraction

### 1. Enable Caching

```java
@SpringBootApplication
@EnableCaching  // ✅ Enable caching
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 2. Simple Cache Configuration (Development)

**`application.yml`**:
```yaml
spring:
  cache:
    type: simple  # In-memory cache (development only)
```

### 3. Cache Annotations

#### @Cacheable - Cache Method Result

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;

    @Cacheable(value = "products", key = "#code")
    public ProductDTO getProduct(String code) {
        // This method will only execute if result is not in cache
        return productRepository.findByCode(code)
            .map(productMapper::toDTO)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }
}
```

**How it works**:
1. Check if cache "products" has entry with key = code
2. If **found** → return cached value (method not executed)
3. If **not found** → execute method, store result in cache, return result

#### @CachePut - Update Cache

```java
@CachePut(value = "products", key = "#result.code")
public ProductDTO updateProduct(String code, UpdateProductRequest request) {
    Product product = productRepository.findByCode(code)
        .orElseThrow(() -> new ResourceNotFoundException("Product not found"));

    productMapper.updateEntity(request, product);
    Product updated = productRepository.save(product);

    return productMapper.toDTO(updated);  // This result updates cache
}
```

**How it works**:
1. Method **always executes**
2. Result is stored in cache with key = result.code
3. Useful for updates to keep cache in sync

#### @CacheEvict - Remove from Cache

```java
@CacheEvict(value = "products", key = "#code")
public void deleteProduct(String code) {
    productRepository.deleteByCode(code);
    // Cache entry for this code is removed
}

// Evict all entries in cache
@CacheEvict(value = "products", allEntries = true)
public void deleteAllProducts() {
    productRepository.deleteAll();
}
```

#### @Caching - Multiple Cache Operations

```java
@Caching(
    cacheable = @Cacheable(value = "products", key = "#code"),
    evict = @CacheEvict(value = "productSummaries", allEntries = true)
)
public ProductDTO getProductWithEviction(String code) {
    return productRepository.findByCode(code)
        .map(productMapper::toDTO)
        .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
}
```

### 4. Cache Key Expressions (SpEL)

```java
// Simple parameter
@Cacheable(value = "users", key = "#username")
public UserDTO getUser(String username) { ... }

// Multiple parameters
@Cacheable(value = "orders", key = "#userId + '-' + #orderId")
public OrderDTO getOrder(Long userId, Long orderId) { ... }

// Object field
@Cacheable(value = "products", key = "#request.code")
public ProductDTO createProduct(CreateProductRequest request) { ... }

// Result field
@CachePut(value = "products", key = "#result.code")
public ProductDTO updateProduct(UpdateProductRequest request) { ... }

// Method name
@Cacheable(value = "products", key = "#root.methodName + #code")
public ProductDTO getProductByCode(String code) { ... }

// Composite key
@Cacheable(value = "search", key = "{#criteria.category, #criteria.minPrice, #criteria.maxPrice}")
public List<ProductDTO> searchProducts(SearchCriteria criteria) { ... }
```

### 5. Conditional Caching

```java
// Cache only if condition is true
@Cacheable(value = "products", key = "#code", condition = "#code.length() > 3")
public ProductDTO getProduct(String code) { ... }

// Cache unless result is null
@Cacheable(value = "users", key = "#username", unless = "#result == null")
public UserDTO findUser(String username) { ... }

// Cache unless exception thrown
@Cacheable(value = "products", key = "#code", unless = "#result == null")
public ProductDTO getProductSafe(String code) {
    try {
        return getProduct(code);
    } catch (Exception e) {
        return null;  // Won't be cached
    }
}
```

## Redis Setup

### 1. Add Dependencies

**Maven (`pom.xml`)**:
```xml
<dependencies>
    <!-- Spring Data Redis -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-redis</artifactId>
    </dependency>

    <!-- Lettuce (Redis client, recommended) -->
    <dependency>
        <groupId>io.lettuce</groupId>
        <artifactId>lettuce-core</artifactId>
    </dependency>

    <!-- Jackson for JSON serialization -->
    <dependency>
        <groupId>com.fasterxml.jackson.core</groupId>
        <artifactId>jackson-databind</artifactId>
    </dependency>

    <!-- Optional: For Redis sessions -->
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session-data-redis</artifactId>
    </dependency>
</dependencies>
```

**Gradle (`build.gradle`)**:
```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-redis'
    implementation 'io.lettuce:lettuce-core'
    implementation 'com.fasterxml.jackson.core:jackson-databind'
    // Optional
    implementation 'org.springframework.session:spring-session-data-redis'
}
```

### 2. Configure Redis

**`application.yml`**:
```yaml
spring:
  cache:
    type: redis
    redis:
      time-to-live: 600000  # 10 minutes in milliseconds
      cache-null-values: false  # Don't cache null values
      use-key-prefix: true

  data:
    redis:
      host: ${REDIS_HOST:localhost}
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD:}
      database: 0
      timeout: 2000ms
      lettuce:
        pool:
          max-active: 8  # Max connections
          max-idle: 8
          min-idle: 0
          max-wait: -1ms
```

**`application-dev.yml`** (Development):
```yaml
spring:
  data:
    redis:
      host: localhost
      port: 6379
      password:  # No password in dev

  cache:
    redis:
      time-to-live: 300000  # 5 minutes in dev
```

**`application-prod.yml`** (Production):
```yaml
spring:
  data:
    redis:
      host: ${REDIS_HOST}  # Required from environment
      port: ${REDIS_PORT:6379}
      password: ${REDIS_PASSWORD}  # Required from environment
      ssl: true  # Use SSL in production

  cache:
    redis:
      time-to-live: 3600000  # 1 hour in production
```

### 3. Redis Configuration Class

```java
@Configuration
@EnableCaching
public class RedisConfig {

    @Bean
    public RedisCacheConfiguration cacheConfiguration() {
        return RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(10))  // Default TTL
            .disableCachingNullValues()
            .serializeKeysWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new StringRedisSerializer()
                )
            )
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair.fromSerializer(
                    new GenericJackson2JsonRedisSerializer()
                )
            );
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration defaultConfig = cacheConfiguration();

        // Custom TTL per cache
        Map<String, RedisCacheConfiguration> cacheConfigurations = new HashMap<>();

        // Products cache: 1 hour
        cacheConfigurations.put("products",
            defaultConfig.entryTtl(Duration.ofHours(1)));

        // Users cache: 30 minutes
        cacheConfigurations.put("users",
            defaultConfig.entryTtl(Duration.ofMinutes(30)));

        // Session cache: 24 hours
        cacheConfigurations.put("sessions",
            defaultConfig.entryTtl(Duration.ofHours(24)));

        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(defaultConfig)
            .withInitialCacheConfigurations(cacheConfigurations)
            .build();
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(
            RedisConnectionFactory connectionFactory) {

        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setConnectionFactory(connectionFactory);

        // Use String serializer for keys
        template.setKeySerializer(new StringRedisSerializer());
        template.setHashKeySerializer(new StringRedisSerializer());

        // Use JSON serializer for values
        GenericJackson2JsonRedisSerializer serializer =
            new GenericJackson2JsonRedisSerializer();
        template.setValueSerializer(serializer);
        template.setHashValueSerializer(serializer);

        template.afterPropertiesSet();
        return template;
    }
}
```

## Cache Patterns

### 1. Cache-Aside (Lazy Loading)

**Most common pattern** - Application manages cache

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;
    private final ProductMapper productMapper;

    @Cacheable(value = "products", key = "#code")
    public ProductDTO getProduct(String code) {
        // 1. Check cache (Spring does this automatically)
        // 2. If not in cache, query database
        // 3. Store result in cache
        // 4. Return result

        return productRepository.findByCode(code)
            .map(productMapper::toDTO)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    }
}
```

**Pros**: Simple, only caches what's requested
**Cons**: Cache miss causes latency

### 2. Write-Through

**Updates cache when writing to database**

```java
@CachePut(value = "products", key = "#result.code")
@Transactional
public ProductDTO updateProduct(String code, UpdateProductRequest request) {
    Product product = productRepository.findByCode(code)
        .orElseThrow(() -> new ResourceNotFoundException("Product not found"));

    productMapper.updateEntity(request, product);
    Product updated = productRepository.save(product);

    // Result is automatically written to cache
    return productMapper.toDTO(updated);
}
```

**Pros**: Cache always consistent with database
**Cons**: Write latency, may cache data that's never read

### 3. Write-Behind (Write-Back)

**Updates cache first, database later (async)**

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final RedisTemplate<String, ProductDTO> redisTemplate;
    private final ProductRepository productRepository;

    public ProductDTO updateProductAsync(String code, UpdateProductRequest request) {
        // 1. Update cache immediately
        ProductDTO updated = /* create updated DTO */;
        redisTemplate.opsForValue().set("product:" + code, updated);

        // 2. Async update to database
        CompletableFuture.runAsync(() -> {
            Product product = productRepository.findByCode(code).orElseThrow();
            productMapper.updateEntity(request, product);
            productRepository.save(product);
        });

        return updated;
    }
}
```

**Pros**: Fast writes, better throughput
**Cons**: Risk of data loss if cache fails, complexity

### 4. Refresh-Ahead

**Proactively refresh cache before expiration**

```java
@Scheduled(fixedRate = 3000000)  // 50 minutes
public void refreshPopularProducts() {
    List<String> popularCodes = getPopularProductCodes();

    for (String code : popularCodes) {
        try {
            // Force cache refresh
            cacheManager.getCache("products").evict(code);
            getProduct(code);  // Reload into cache
        } catch (Exception e) {
            log.error("Failed to refresh product: {}", code, e);
        }
    }
}
```

**Pros**: Reduces cache misses for popular data
**Cons**: Complexity, may refresh unused data

## Advanced Caching

### 1. Custom Cache Key Generator

```java
@Configuration
public class CacheConfig {

    @Bean
    public KeyGenerator customKeyGenerator() {
        return (target, method, params) -> {
            StringBuilder key = new StringBuilder();
            key.append(target.getClass().getSimpleName()).append(":");
            key.append(method.getName()).append(":");

            for (Object param : params) {
                if (param != null) {
                    key.append(param.toString()).append(":");
                }
            }

            return key.toString();
        };
    }
}

// Usage
@Cacheable(value = "products", keyGenerator = "customKeyGenerator")
public ProductDTO getProduct(String code) { ... }
```

### 2. Multiple Cache Names

```java
@Cacheable(value = {"products", "catalogItems"}, key = "#code")
public ProductDTO getProduct(String code) {
    // Cached in both "products" and "catalogItems" caches
    return productRepository.findByCode(code)
        .map(productMapper::toDTO)
        .orElseThrow();
}
```

### 3. Cache with Collection Results

```java
@Cacheable(value = "productLists", key = "#category")
public List<ProductDTO> getProductsByCategory(String category) {
    return productRepository.findAll(
        ProductSpecifications.inCategory(category)
    ).stream()
        .map(productMapper::toDTO)
        .collect(Collectors.toList());
}

// Evict when any product in category changes
@CacheEvict(value = "productLists", key = "#product.category")
public ProductDTO updateProduct(Product product) { ... }
```

### 4. Programmatic Cache Access

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final CacheManager cacheManager;

    public void clearProductCache(String code) {
        Cache cache = cacheManager.getCache("products");
        if (cache != null) {
            cache.evict(code);
        }
    }

    public ProductDTO getFromCacheOrNull(String code) {
        Cache cache = cacheManager.getCache("products");
        if (cache != null) {
            Cache.ValueWrapper wrapper = cache.get(code);
            if (wrapper != null) {
                return (ProductDTO) wrapper.get();
            }
        }
        return null;
    }

    public void putInCache(String code, ProductDTO product) {
        Cache cache = cacheManager.getCache("products");
        if (cache != null) {
            cache.put(code, product);
        }
    }
}
```

### 5. Using RedisTemplate Directly

```java
@Service
@RequiredArgsConstructor
public class SessionService {

    private final RedisTemplate<String, UserSession> redisTemplate;

    public void saveSession(String sessionId, UserSession session) {
        redisTemplate.opsForValue().set(
            "session:" + sessionId,
            session,
            Duration.ofHours(24)  // TTL
        );
    }

    public UserSession getSession(String sessionId) {
        return redisTemplate.opsForValue().get("session:" + sessionId);
    }

    public void deleteSession(String sessionId) {
        redisTemplate.delete("session:" + sessionId);
    }

    // Hash operations
    public void saveUserPreference(String userId, String key, String value) {
        redisTemplate.opsForHash().put(
            "user:preferences:" + userId,
            key,
            value
        );
    }

    // List operations
    public void addToRecentSearches(String userId, String searchTerm) {
        String key = "user:searches:" + userId;
        redisTemplate.opsForList().leftPush(key, searchTerm);
        redisTemplate.opsForList().trim(key, 0, 9);  // Keep last 10
        redisTemplate.expire(key, Duration.ofDays(30));
    }

    // Set operations (unique items)
    public void addFavorite(String userId, String productCode) {
        redisTemplate.opsForSet().add(
            "user:favorites:" + userId,
            productCode
        );
    }

    // Sorted set (with scores)
    public void incrementProductViews(String productCode) {
        redisTemplate.opsForZSet().incrementScore(
            "products:popular",
            productCode,
            1.0
        );
    }
}
```

## Cache Eviction Strategies

### 1. Time-Based Eviction (TTL)

```java
// Global TTL
@Bean
public RedisCacheConfiguration cacheConfiguration() {
    return RedisCacheConfiguration.defaultCacheConfig()
        .entryTtl(Duration.ofMinutes(30));
}

// Per-cache TTL
Map<String, RedisCacheConfiguration> configs = new HashMap<>();
configs.put("products", defaultConfig.entryTtl(Duration.ofHours(1)));
configs.put("users", defaultConfig.entryTtl(Duration.ofMinutes(30)));
```

### 2. Size-Based Eviction

```java
// With Caffeine cache (in-memory)
@Bean
public CacheManager cacheManager() {
    CaffeineCacheManager cacheManager = new CaffeineCacheManager();
    cacheManager.setCaffeine(Caffeine.newBuilder()
        .maximumSize(1000)  // Max 1000 entries
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .recordStats());
    return cacheManager;
}
```

### 3. Event-Based Eviction

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final CacheManager cacheManager;
    private final ApplicationEventPublisher eventPublisher;

    @CacheEvict(value = "products", key = "#code")
    public void deleteProduct(String code) {
        productRepository.deleteByCode(code);

        // Publish event to evict related caches
        eventPublisher.publishEvent(new ProductDeletedEvent(code));
    }
}

@Component
@RequiredArgsConstructor
public class CacheEvictionListener {

    private final CacheManager cacheManager;

    @EventListener
    public void handleProductDeleted(ProductDeletedEvent event) {
        // Evict all product list caches
        Cache cache = cacheManager.getCache("productLists");
        if (cache != null) {
            cache.clear();
        }
    }
}
```

### 4. Scheduled Cache Clearing

```java
@Component
@EnableScheduling
public class CacheMaintenanceScheduler {

    private final CacheManager cacheManager;

    // Clear all caches daily at 3 AM
    @Scheduled(cron = "0 0 3 * * *")
    public void clearAllCaches() {
        cacheManager.getCacheNames().forEach(cacheName -> {
            Cache cache = cacheManager.getCache(cacheName);
            if (cache != null) {
                cache.clear();
                log.info("Cleared cache: {}", cacheName);
            }
        });
    }

    // Refresh popular products every hour
    @Scheduled(fixedRate = 3600000)
    public void refreshPopularProducts() {
        // Implementation
    }
}
```

## Best Practices

### 1. Cache Naming Conventions

```java
// ✅ GOOD: Descriptive, hierarchical names
@Cacheable("products")
@Cacheable("products:summary")
@Cacheable("products:details")
@Cacheable("users:profile")
@Cacheable("orders:user")

// ❌ BAD: Generic, unclear names
@Cacheable("cache1")
@Cacheable("data")
@Cacheable("myCache")
```

### 2. Appropriate TTL

```java
// Static data: Long TTL
products -> 24 hours
categories -> 12 hours

// Dynamic data: Short TTL
user sessions -> 30 minutes
search results -> 5 minutes
real-time data -> 1 minute

// Rarely changing: Very long TTL
configuration -> 7 days
feature flags -> 1 day
```

### 3. Cache Size Limits

```yaml
spring:
  data:
    redis:
      lettuce:
        pool:
          max-active: 8  # Don't overload Redis
          max-wait: 1000ms  # Fail fast if pool exhausted
```

### 4. Handle Cache Failures Gracefully

```java
@Cacheable(value = "products", key = "#code")
public ProductDTO getProduct(String code) {
    try {
        return productRepository.findByCode(code)
            .map(productMapper::toDTO)
            .orElseThrow(() -> new ResourceNotFoundException("Product not found"));
    } catch (Exception e) {
        // Log but don't fail - degrade gracefully
        log.error("Cache failure, querying database directly", e);
        return queryDatabaseDirectly(code);
    }
}
```

### 5. Don't Cache Everything

**Cache**:
✅ Expensive database queries
✅ External API calls
✅ Computed/aggregated data
✅ Frequently accessed data

**Don't Cache**:
❌ User-specific data (unless partitioned)
❌ Rapidly changing data
❌ Large objects (>1MB)
❌ Security-sensitive data

### 6. Monitor Cache Performance

```java
@Component
public class CacheMetrics {

    @Scheduled(fixedRate = 60000)
    public void logCacheStats() {
        CacheManager cacheManager = // get cache manager

        for (String cacheName : cacheManager.getCacheNames()) {
            Cache cache = cacheManager.getCache(cacheName);
            // Log cache hit/miss ratio
            // Log cache size
            // Log eviction count
        }
    }
}
```

## Testing with Cache

### 1. Disable Cache in Tests

```yaml
# application-test.yml
spring:
  cache:
    type: none  # Disable caching in tests
```

### 2. Test with Cache Enabled

```java
@SpringBootTest
@EnableCaching
class ProductServiceCacheTest {

    @Autowired
    private ProductService productService;

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private CacheManager cacheManager;

    @Test
    void shouldCacheProductOnSecondCall() {
        String code = "PROD-001";

        // First call - cache miss
        ProductDTO first = productService.getProduct(code);
        assertNotNull(first);

        // Verify it's in cache
        Cache cache = cacheManager.getCache("products");
        assertNotNull(cache.get(code));

        // Second call - cache hit (shouldn't query DB)
        ProductDTO second = productService.getProduct(code);
        assertEquals(first.getCode(), second.getCode());

        // Verify only one DB call was made
        verify(productRepository, times(1)).findByCode(code);
    }

    @Test
    void shouldEvictCacheOnUpdate() {
        String code = "PROD-001";

        // Cache the product
        productService.getProduct(code);

        // Update (should evict cache)
        productService.updateProduct(code, updateRequest);

        // Verify cache was evicted
        Cache cache = cacheManager.getCache("products");
        assertNull(cache.get(code));
    }
}
```

### 3. Integration Test with Redis Testcontainers

```java
@SpringBootTest
@Testcontainers
class ProductServiceRedisTest {

    @Container
    static GenericContainer<?> redis = new GenericContainer<>("redis:7-alpine")
        .withExposedPorts(6379);

    @DynamicPropertySource
    static void redisProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.data.redis.host", redis::getHost);
        registry.add("spring.data.redis.port", redis::getFirstMappedPort);
    }

    @Autowired
    private ProductService productService;

    @Test
    void shouldUseRedisCache() {
        // Test with real Redis
        ProductDTO product = productService.getProduct("PROD-001");
        assertNotNull(product);
    }
}
```

## Docker Setup for Redis

### docker-compose.yml

```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    networks:
      - myapp-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3

volumes:
  redis-data:

networks:
  myapp-network:
```

## Troubleshooting

### Cache Not Working

```java
// Check if caching is enabled
@SpringBootApplication
@EnableCaching  // ✅ Must be present
public class Application { }

// Check if method is public
public class ProductService {
    @Cacheable("products")
    public ProductDTO getProduct(String code) { }  // ✅ Must be public
}

// Check if calling from same class (won't work)
public class ProductService {
    public void caller() {
        this.getProduct("code");  // ❌ Won't use cache (same class call)
    }

    @Cacheable("products")
    public ProductDTO getProduct(String code) { }
}
```

### Redis Connection Failures

```java
@Configuration
public class RedisConfig {

    @Bean
    public LettuceConnectionFactory redisConnectionFactory() {
        LettuceConnectionFactory factory = new LettuceConnectionFactory();
        factory.setValidateConnection(true);  // Validate before use
        return factory;
    }
}
```

### Serialization Issues

```java
// Ensure DTOs are serializable
@Data
public class ProductDTO implements Serializable {  // ✅ For Java serialization
    private static final long serialVersionUID = 1L;

    private String code;
    private String name;
    // ...
}

// Or use JSON serialization (better)
@Bean
public RedisCacheConfiguration cacheConfiguration() {
    return RedisCacheConfiguration.defaultCacheConfig()
        .serializeValuesWith(
            RedisSerializationContext.SerializationPair.fromSerializer(
                new GenericJackson2JsonRedisSerializer()  // ✅ JSON
            )
        );
}
```

## Summary Checklist

✅ **Setup**:
- [ ] Add Redis dependencies
- [ ] Configure Redis connection
- [ ] Add @EnableCaching
- [ ] Configure cache manager

✅ **Usage**:
- [ ] Use @Cacheable for reads
- [ ] Use @CachePut for updates
- [ ] Use @CacheEvict for deletes
- [ ] Set appropriate TTLs

✅ **Best Practices**:
- [ ] Don't cache user-specific data
- [ ] Handle cache failures gracefully
- [ ] Monitor cache hit/miss ratios
- [ ] Use meaningful cache names
- [ ] Set size limits

✅ **Testing**:
- [ ] Test with cache disabled
- [ ] Test cache hit/miss scenarios
- [ ] Test cache eviction
- [ ] Use Testcontainers for integration tests

## Related Documentation

- See [database.md](./database.md) for database configuration
- See [../backend/services.md](../backend/services.md) for service layer patterns
- See [../testing/integration-tests.md](../testing/integration-tests.md) for testing strategies
