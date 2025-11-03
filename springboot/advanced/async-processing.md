# Async Processing and Events

## Overview

Asynchronous processing improves application responsiveness and scalability by executing long-running tasks in the background. This guide covers @Async methods, Spring Events, and message queues.

## @Async Methods

### 1. Enable Async Processing

```java
@SpringBootApplication
@EnableAsync  // ✅ Enable async support
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 2. Configure Thread Pool

```java
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(5);
        executor.setMaxPoolSize(10);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (ex, method, params) ->
            log.error("Async method {} threw exception: {}", method.getName(), ex.getMessage(), ex);
    }
}
```

### 3. Async Methods

**Fire and Forget**:
```java
@Service
@Slf4j
public class EmailService {

    @Async
    public void sendWelcomeEmail(User user) {
        log.info("Sending welcome email to: {}", user.getEmail());

        try {
            Thread.sleep(2000);  // Simulate email sending
            // Send email logic
            log.info("Welcome email sent to: {}", user.getEmail());
        } catch (Exception e) {
            log.error("Failed to send email", e);
        }
    }
}
```

**With CompletableFuture**:
```java
@Service
public class ReportService {

    @Async
    public CompletableFuture<ReportDTO> generateReport(String reportType) {
        try {
            // Long-running operation
            Thread.sleep(5000);
            ReportDTO report = createReport(reportType);
            return CompletableFuture.completedFuture(report);
        } catch (Exception e) {
            return CompletableFuture.failedFuture(e);
        }
    }
}

// Usage in controller
@GetMapping("/reports/{type}")
public CompletableFuture<ResponseEntity<ReportDTO>> getReport(@PathVariable String type) {
    return reportService.generateReport(type)
        .thenApply(ResponseEntity::ok)
        .exceptionally(ex -> ResponseEntity.internalServerError().build());
}
```

### 4. Multiple Async Calls

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final EmailService emailService;
    private final NotificationService notificationService;
    private final InventoryService inventoryService;

    @Transactional
    public OrderDTO createOrder(CreateOrderRequest request) {
        // Save order (synchronous)
        Order order = orderMapper.toEntity(request);
        Order saved = orderRepository.save(order);

        // Async operations (don't wait)
        emailService.sendOrderConfirmation(saved);
        notificationService.notifyWarehouse(saved);
        inventoryService.updateStock(saved);

        return orderMapper.toDTO(saved);
    }

    // Wait for multiple async operations
    public DashboardDTO getDashboard() {
        CompletableFuture<List<OrderDTO>> orders = orderService.getRecentOrders();
        CompletableFuture<StatisticsDTO> stats = statisticsService.getStatistics();
        CompletableFuture<List<ProductDTO>> products = productService.getTopProducts();

        // Wait for all to complete
        CompletableFuture.allOf(orders, stats, products).join();

        return new DashboardDTO(
            orders.join(),
            stats.join(),
            products.join()
        );
    }
}
```

## Spring Application Events

### 1. Define Custom Event

```java
@Getter
public class OrderCreatedEvent extends ApplicationEvent {

    private final Order order;
    private final LocalDateTime timestamp;

    public OrderCreatedEvent(Object source, Order order) {
        super(source);
        this.order = order;
        this.timestamp = LocalDateTime.now();
    }
}
```

### 2. Publish Event

```java
@Service
@RequiredArgsConstructor
public class OrderService {

    private final ApplicationEventPublisher eventPublisher;

    @Transactional
    public OrderDTO createOrder(CreateOrderRequest request) {
        Order order = orderMapper.toEntity(request);
        Order saved = orderRepository.save(order);

        // ✅ Publish event
        eventPublisher.publishEvent(new OrderCreatedEvent(this, saved));

        return orderMapper.toDTO(saved);
    }
}
```

### 3. Listen to Events

**Synchronous Listener**:
```java
@Component
@Slf4j
public class OrderEventListener {

    @EventListener
    public void handleOrderCreated(OrderCreatedEvent event) {
        log.info("Order created: {}", event.getOrder().getId());
        // This runs in the same thread as the publisher
    }
}
```

**Asynchronous Listener**:
```java
@Component
@Slf4j
public class NotificationListener {

    @Async
    @EventListener
    public void sendOrderNotification(OrderCreatedEvent event) {
        log.info("Sending notification for order: {}", event.getOrder().getId());
        // This runs in a separate thread
    }
}
```

**Conditional Listener**:
```java
@EventListener(condition = "#event.order.total > 1000")
public void handleLargeOrder(OrderCreatedEvent event) {
    // Only triggered for orders > 1000
    log.info("Large order detected: {}", event.getOrder().getId());
}
```

### 4. @TransactionalEventListener

**Execute After Transaction Commit**:
```java
@Component
@Slf4j
public class EmailListener {

    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    public void sendEmailAfterCommit(OrderCreatedEvent event) {
        // ✅ Only sends email if transaction commits successfully
        emailService.sendOrderConfirmation(event.getOrder());
    }

    @TransactionalEventListener(phase = TransactionPhase.AFTER_ROLLBACK)
    public void handleRollback(OrderCreatedEvent event) {
        // Triggered if transaction rolls back
        log.error("Order creation rolled back: {}", event.getOrder().getId());
    }
}
```

## Message Queues

### RabbitMQ Setup

**1. Add Dependency**:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

**2. Configure RabbitMQ**:
```yaml
spring:
  rabbitmq:
    host: ${RABBITMQ_HOST:localhost}
    port: ${RABBITMQ_PORT:5672}
    username: ${RABBITMQ_USER:guest}
    password: ${RABBITMQ_PASSWORD:guest}
    virtual-host: /
    listener:
      simple:
        acknowledge-mode: auto
        concurrency: 5
        max-concurrency: 10
```

**3. Queue Configuration**:
```java
@Configuration
public class RabbitMQConfig {

    public static final String ORDER_QUEUE = "order.created";
    public static final String ORDER_EXCHANGE = "order.exchange";
    public static final String ORDER_ROUTING_KEY = "order.created";

    @Bean
    public Queue orderQueue() {
        return new Queue(ORDER_QUEUE, true);  // Durable queue
    }

    @Bean
    public TopicExchange orderExchange() {
        return new TopicExchange(ORDER_EXCHANGE);
    }

    @Bean
    public Binding orderBinding(Queue orderQueue, TopicExchange orderExchange) {
        return BindingBuilder.bind(orderQueue)
            .to(orderExchange)
            .with(ORDER_ROUTING_KEY);
    }

    @Bean
    public MessageConverter jsonMessageConverter() {
        return new Jackson2JsonMessageConverter();
    }
}
```

**4. Publish Message**:
```java
@Service
@RequiredArgsConstructor
public class OrderPublisher {

    private final RabbitTemplate rabbitTemplate;

    public void publishOrderCreated(Order order) {
        OrderMessage message = new OrderMessage(
            order.getId(),
            order.getUser().getId(),
            order.getTotal()
        );

        rabbitTemplate.convertAndSend(
            RabbitMQConfig.ORDER_EXCHANGE,
            RabbitMQConfig.ORDER_ROUTING_KEY,
            message
        );
    }
}
```

**5. Consume Message**:
```java
@Component
@Slf4j
public class OrderConsumer {

    @RabbitListener(queues = RabbitMQConfig.ORDER_QUEUE)
    public void handleOrderCreated(OrderMessage message) {
        log.info("Received order message: {}", message.getOrderId());

        try {
            processOrder(message);
        } catch (Exception e) {
            log.error("Failed to process order: {}", message.getOrderId(), e);
            throw new AmqpRejectAndDontRequeueException("Processing failed", e);
        }
    }
}
```

### Kafka Setup (Alternative)

**1. Add Dependency**:
```xml
<dependency>
    <groupId>org.springframework.kafka</groupId>
    <artifactId>spring-kafka</artifactId>
</dependency>
```

**2. Configure Kafka**:
```yaml
spring:
  kafka:
    bootstrap-servers: ${KAFKA_BOOTSTRAP_SERVERS:localhost:9092}
    consumer:
      group-id: myapp-group
      auto-offset-reset: earliest
      key-deserializer: org.apache.kafka.common.serialization.StringDeserializer
      value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
      properties:
        spring.json.trusted.packages: com.example.myapp
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

**3. Publish to Kafka**:
```java
@Service
@RequiredArgsConstructor
public class KafkaProducer {

    private final KafkaTemplate<String, OrderMessage> kafkaTemplate;

    public void sendOrderEvent(OrderMessage message) {
        kafkaTemplate.send("order-events", message.getOrderId().toString(), message);
    }
}
```

**4. Consume from Kafka**:
```java
@Component
@Slf4j
public class KafkaConsumer {

    @KafkaListener(topics = "order-events", groupId = "myapp-group")
    public void consumeOrderEvent(OrderMessage message) {
        log.info("Consumed order event: {}", message.getOrderId());
        processOrder(message);
    }
}
```

## Scheduled Tasks

### 1. Enable Scheduling

```java
@SpringBootApplication
@EnableScheduling  // ✅ Enable scheduling
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 2. Scheduled Methods

```java
@Component
@Slf4j
public class ScheduledTasks {

    // Fixed rate: Every 5 seconds
    @Scheduled(fixedRate = 5000)
    public void reportCurrentTime() {
        log.info("Current time: {}", LocalDateTime.now());
    }

    // Fixed delay: 5 seconds after previous completion
    @Scheduled(fixedDelay = 5000)
    public void processQueue() {
        log.info("Processing queue");
        // Process items
    }

    // Cron expression: Every day at 3 AM
    @Scheduled(cron = "0 0 3 * * *")
    public void dailyMaintenance() {
        log.info("Running daily maintenance");
        // Cleanup, backups, etc.
    }

    // Cron: Every Monday at 9 AM
    @Scheduled(cron = "0 0 9 * * MON")
    public void weeklyReport() {
        log.info("Generating weekly report");
    }
}
```

## Best Practices

### 1. Use Appropriate Thread Pool Size

```java
@Configuration
@EnableAsync
public class AsyncConfig {

    @Bean(name = "emailExecutor")
    public Executor emailExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(2);  // Based on load
        executor.setMaxPoolSize(5);
        executor.setQueueCapacity(50);
        executor.setThreadNamePrefix("email-");
        executor.initialize();
        return executor;
    }

    @Bean(name = "reportExecutor")
    public Executor reportExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(3);  // CPU-intensive
        executor.setMaxPoolSize(6);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("report-");
        executor.initialize();
        return executor;
    }
}

// Usage
@Async("emailExecutor")
public void sendEmail() { }

@Async("reportExecutor")
public CompletableFuture<Report> generateReport() { }
```

### 2. Handle Exceptions

```java
@Async
public CompletableFuture<Result> processAsync() {
    try {
        Result result = doWork();
        return CompletableFuture.completedFuture(result);
    } catch (Exception e) {
        log.error("Async processing failed", e);
        return CompletableFuture.failedFuture(e);
    }
}
```

### 3. Avoid Self-Invocation

```java
// ❌ WRONG: @Async won't work (same class call)
@Service
public class ProductService {

    public void updateProduct(Long id) {
        // ...
        this.sendNotification();  // Won't be async!
    }

    @Async
    public void sendNotification() {
        // ...
    }
}

// ✅ CORRECT: Inject another service
@Service
@RequiredArgsConstructor
public class ProductService {

    private final NotificationService notificationService;

    public void updateProduct(Long id) {
        // ...
        notificationService.sendNotification();  // Will be async!
    }
}

@Service
public class NotificationService {

    @Async
    public void sendNotification() {
        // ...
    }
}
```

### 4. Use @TransactionalEventListener

```java
// ✅ GOOD: Wait for transaction to commit
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
@Async
public void sendEmail(OrderCreatedEvent event) {
    // Email only sent if order successfully saved
    emailService.send(event.getOrder());
}
```

### 5. Idempotent Message Processing

```java
@Component
public class OrderConsumer {

    private final Set<String> processedMessages = ConcurrentHashMap.newKeySet();

    @RabbitListener(queues = "order.queue")
    public void processOrder(OrderMessage message) {
        String messageId = message.getId();

        // ✅ Prevent duplicate processing
        if (!processedMessages.add(messageId)) {
            log.warn("Duplicate message ignored: {}", messageId);
            return;
        }

        try {
            // Process message
            processOrder(message);
        } finally {
            // Optional: Remove after some time to prevent memory leak
            scheduleMessageIdCleanup(messageId);
        }
    }
}
```

## Performance Considerations

### 1. Monitor Thread Pools

```java
@Component
@Slf4j
public class ThreadPoolMonitor {

    @Scheduled(fixedRate = 60000)  // Every minute
    public void logThreadPoolStats() {
        ThreadPoolTaskExecutor executor = (ThreadPoolTaskExecutor) asyncExecutor;

        log.info("Thread Pool Stats - Active: {}, Queue: {}, Completed: {}",
            executor.getActiveCount(),
            executor.getThreadPoolExecutor().getQueue().size(),
            executor.getThreadPoolExecutor().getCompletedTaskCount());
    }
}
```

### 2. Circuit Breaker Pattern

```java
@Service
public class ExternalApiService {

    private final CircuitBreaker circuitBreaker = CircuitBreaker.ofDefaults("externalApi");

    @Async
    public CompletableFuture<Result> callExternalApi() {
        return CompletableFuture.supplyAsync(() ->
            circuitBreaker.executeSupplier(() -> {
                // Call external API
                return externalApi.getData();
            })
        );
    }
}
```

## Checklist

✅ **Async Configuration**:
- [ ] @EnableAsync configured
- [ ] Custom thread pools defined
- [ ] Exception handler configured
- [ ] Thread pool monitoring in place

✅ **Events**:
- [ ] Custom events created
- [ ] Event publishers implemented
- [ ] Event listeners created
- [ ] @TransactionalEventListener used correctly

✅ **Message Queues**:
- [ ] Message broker configured (RabbitMQ/Kafka)
- [ ] Queues/topics defined
- [ ] Publishers implemented
- [ ] Consumers with error handling
- [ ] Idempotent processing

✅ **Best Practices**:
- [ ] Avoid self-invocation
- [ ] Handle exceptions properly
- [ ] Monitor performance
- [ ] Use appropriate pool sizes
- [ ] Test async behavior

## Related Documentation

- See [../testing/integration-tests.md](../testing/integration-tests.md) for testing async methods
- See [../deployment/monitoring.md](../deployment/monitoring.md) for monitoring async tasks
- See [../configuration/caching.md](../configuration/caching.md) for async caching patterns
