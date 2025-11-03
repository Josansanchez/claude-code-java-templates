# Monitoring and Observability

## Overview

Monitoring and observability are critical for understanding application behavior, detecting issues, and maintaining reliability in production. This guide covers Spring Boot Actuator, Prometheus metrics, and Grafana dashboards.

## Spring Boot Actuator

### 1. Add Dependency

**Maven (`pom.xml`)**:
```xml
<dependencies>
    <!-- Spring Boot Actuator -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>

    <!-- Micrometer Prometheus Registry -->
    <dependency>
        <groupId>io.micrometer</groupId>
        <artifactId>micrometer-registry-prometheus</artifactId>
    </dependency>
</dependencies>
```

### 2. Configure Actuator

**`application.yml`**:
```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
      base-path: /actuator
  endpoint:
    health:
      show-details: always
      show-components: always
    metrics:
      enabled: true
    prometheus:
      enabled: true
  metrics:
    tags:
      application: ${spring.application.name}
      environment: ${spring.profiles.active}
    export:
      prometheus:
        enabled: true

info:
  app:
    name: @project.name@
    description: @project.description@
    version: @project.version@
```

### 3. Health Checks

**Custom Health Indicator**:
```java
@Component
public class DatabaseHealthIndicator implements HealthIndicator {

    private final DataSource dataSource;

    @Override
    public Health health() {
        try (Connection conn = dataSource.getConnection()) {
            return Health.up()
                .withDetail("database", "MySQL")
                .withDetail("validConnection", true)
                .build();
        } catch (SQLException e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}
```

**Redis Health Indicator**:
```java
@Component
@RequiredArgsConstructor
public class RedisHealthIndicator implements HealthIndicator {

    private final RedisTemplate<String, String> redisTemplate;

    @Override
    public Health health() {
        try {
            String pong = redisTemplate.getConnectionFactory()
                .getConnection()
                .ping();

            return Health.up()
                .withDetail("redis", "connected")
                .withDetail("response", pong)
                .build();
        } catch (Exception e) {
            return Health.down()
                .withDetail("error", e.getMessage())
                .build();
        }
    }
}
```

### 4. Custom Metrics

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductRepository productRepository;
    private final MeterRegistry meterRegistry;

    public ProductDTO createProduct(CreateProductRequest request) {
        // Increment counter
        meterRegistry.counter("products.created",
            "category", request.getCategory()).increment();

        // Record execution time
        return Timer.record(
            meterRegistry,
            () -> {
                Product product = productMapper.toEntity(request);
                Product saved = productRepository.save(product);
                return productMapper.toDTO(saved);
            }
        );
    }

    public List<ProductDTO> searchProducts(SearchCriteria criteria) {
        // Track search distribution
        meterRegistry.gauge("products.search.active",
            productRepository.count());

        return productRepository.findAll(spec).stream()
            .map(productMapper::toDTO)
            .collect(Collectors.toList());
    }
}
```

## Prometheus Setup

### 1. Prometheus Configuration

**`prometheus.yml`**:
```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'spring-boot-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['app:8080']
        labels:
          application: 'myapp'
          environment: 'production'
```

### 2. Docker Compose with Prometheus

**`docker-compose.yml`** (extended):
```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: myapp-prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus/prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/etc/prometheus/console_libraries'
      - '--web.console.templates=/etc/prometheus/consoles'
    networks:
      - myapp-network
    restart: unless-stopped

volumes:
  prometheus-data:
```

## Grafana Setup

### 1. Docker Compose with Grafana

**`docker-compose.yml`** (extended):
```yaml
services:
  grafana:
    image: grafana/grafana:latest
    container_name: myapp-grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=admin
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD:-admin}
      - GF_SERVER_ROOT_URL=http://localhost:3000
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/dashboards:/etc/grafana/provisioning/dashboards
      - ./grafana/datasources:/etc/grafana/provisioning/datasources
    depends_on:
      - prometheus
    networks:
      - myapp-network
    restart: unless-stopped

volumes:
  grafana-data:
```

### 2. Datasource Configuration

**`grafana/datasources/prometheus.yml`**:
```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    access: proxy
    url: http://prometheus:9090
    isDefault: true
    editable: true
```

### 3. Dashboard Provisioning

**`grafana/dashboards/dashboard.yml`**:
```yaml
apiVersion: 1

providers:
  - name: 'SpringBoot'
    folder: 'Spring Boot'
    type: file
    options:
      path: /etc/grafana/provisioning/dashboards
```

## Key Metrics to Monitor

### Application Metrics

```promql
# Request Rate
rate(http_server_requests_seconds_count[5m])

# Error Rate
rate(http_server_requests_seconds_count{status=~"5.."}[5m])

# Response Time (p95)
histogram_quantile(0.95, rate(http_server_requests_seconds_bucket[5m]))

# Active Requests
http_server_requests_active

# JVM Memory
jvm_memory_used_bytes
jvm_memory_max_bytes

# CPU Usage
process_cpu_usage
system_cpu_usage

# Thread Count
jvm_threads_live

# GC Time
rate(jvm_gc_pause_seconds_sum[5m])
```

### Database Metrics

```promql
# Connection Pool
hikaricp_connections_active
hikaricp_connections_idle
hikaricp_connections_max

# Query Time
rate(jdbc_connections_active[5m])
```

### Cache Metrics

```promql
# Cache Hit Rate
rate(cache_gets{result="hit"}[5m]) /
rate(cache_gets[5m])

# Cache Size
cache_size
```

## Logging Integration

### 1. Logback Configuration

**`src/main/resources/logback-spring.xml`**:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- Console Appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- File Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>/app/logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>/app/logs/application-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- JSON Appender (for ELK) -->
    <appender name="JSON" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>/app/logs/application.json</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>/app/logs/application-%d{yyyy-MM-dd}.json</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>

    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="JSON"/>
    </root>
</configuration>
```

### 2. Structured Logging

```java
@Slf4j
@Service
public class ProductService {

    public ProductDTO createProduct(CreateProductRequest request) {
        log.info("Creating product: code={}, category={}",
            request.getCode(), request.getCategory());

        try {
            Product saved = productRepository.save(product);

            log.info("Product created successfully: id={}, code={}",
                saved.getId(), saved.getCode());

            return productMapper.toDTO(saved);
        } catch (Exception e) {
            log.error("Failed to create product: code={}, error={}",
                request.getCode(), e.getMessage(), e);
            throw e;
        }
    }
}
```

## Alerting

### Prometheus Alerts

**`prometheus/alerts.yml`**:
```yaml
groups:
  - name: application_alerts
    rules:
      # High Error Rate
      - alert: HighErrorRate
        expr: rate(http_server_requests_seconds_count{status=~"5.."}[5m]) > 0.05
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} errors/sec"

      # High Response Time
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, rate(http_server_requests_seconds_bucket[5m])) > 1
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High response time detected"
          description: "P95 response time is {{ $value }}s"

      # Database Connection Pool Exhausted
      - alert: DatabasePoolExhausted
        expr: hikaricp_connections_active / hikaricp_connections_max > 0.9
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "Database connection pool nearly exhausted"
          description: "{{ $value }} of connections in use"

      # High JVM Memory Usage
      - alert: HighMemoryUsage
        expr: (jvm_memory_used_bytes / jvm_memory_max_bytes) > 0.9
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High JVM memory usage"
          description: "Memory usage is {{ $value }}%"
```

## Best Practices

### 1. Meaningful Metric Names

```java
// ✅ GOOD
meterRegistry.counter("products.created", "category", category);
meterRegistry.timer("products.search.duration");

// ❌ BAD
meterRegistry.counter("counter1");
meterRegistry.timer("time");
```

### 2. Use Tags for Dimensions

```java
// ✅ GOOD: Tags allow filtering/aggregation
meterRegistry.counter("api.requests",
    "endpoint", "/products",
    "method", "GET",
    "status", "200");

// ❌ BAD: Too specific, creates cardinality explosion
meterRegistry.counter("api.requests.products.get.200");
```

### 3. Security

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
        exclude: shutdown,env  # Don't expose sensitive endpoints
  endpoint:
    health:
      show-details: when-authorized  # Restrict details
```

### 4. Performance

```yaml
management:
  metrics:
    enable:
      jvm: true
      system: true
      process: true
    export:
      prometheus:
        step: 15s  # Match Prometheus scrape interval
```

## Monitoring Checklist

✅ **Metrics**:
- [ ] Spring Boot Actuator enabled
- [ ] Prometheus metrics exposed
- [ ] Custom business metrics added
- [ ] JVM metrics enabled
- [ ] Database pool metrics tracked

✅ **Health Checks**:
- [ ] Application health endpoint
- [ ] Database health check
- [ ] Redis health check
- [ ] Custom health indicators

✅ **Logging**:
- [ ] Structured logging configured
- [ ] Log levels appropriate
- [ ] Log rotation enabled
- [ ] Sensitive data not logged

✅ **Dashboards**:
- [ ] Grafana configured
- [ ] Key metrics visualized
- [ ] Alerts configured
- [ ] Dashboard templates provisioned

✅ **Production**:
- [ ] Metrics secured
- [ ] Retention policy configured
- [ ] Backup strategy for metrics
- [ ] On-call alerts configured

## Related Documentation

- See [docker.md](./docker.md) for Docker deployment
- See [../configuration/caching.md](../configuration/caching.md) for cache metrics
- See [../documentation/logging.md](../documentation/logging.md) for logging configuration
