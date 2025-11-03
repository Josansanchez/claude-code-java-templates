# Docker Deployment

## Overview

This guide covers containerizing Spring Boot applications with Docker, including production-ready configurations, multi-stage builds, and orchestration with docker-compose.

## Why Docker?

✅ **Consistency**: Same environment in dev, test, and prod
✅ **Portability**: Run anywhere Docker runs
✅ **Isolation**: Dependencies isolated from host
✅ **Scalability**: Easy horizontal scaling
✅ **CI/CD**: Streamlined deployment pipeline
✅ **Resource efficiency**: Lightweight compared to VMs

## Dockerfile

### Basic Dockerfile

**`Dockerfile`**:
```dockerfile
FROM eclipse-temurin:17-jre-alpine

# Set working directory
WORKDIR /app

# Copy JAR file
COPY target/*.jar app.jar

# Expose port
EXPOSE 8080

# Run application
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Multi-Stage Build (Recommended)

**`Dockerfile`**:
```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-17-alpine AS build

WORKDIR /app

# Copy pom.xml and download dependencies (cached layer)
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source code and build
COPY src ./src
RUN mvn clean package -DskipTests -B

# Stage 2: Runtime
FROM eclipse-temurin:17-jre-alpine

WORKDIR /app

# Create non-root user for security
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

# Copy JAR from build stage
COPY --from=build /app/target/*.jar app.jar

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD wget --no-verbose --tries=1 --spider http://localhost:8080/actuator/health || exit 1

# JVM options for containers
ENV JAVA_OPTS="-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"

# Run application
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar app.jar"]
```

### Production-Optimized Dockerfile

**`Dockerfile`**:
```dockerfile
# Stage 1: Build
FROM maven:3.9-eclipse-temurin-17-alpine AS build

WORKDIR /app

# Copy pom.xml first for better caching
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source and build
COPY src ./src
RUN mvn clean package -DskipTests -B

# Stage 2: Extract layers
FROM eclipse-temurin:17-jre-alpine AS layers

WORKDIR /app
COPY --from=build /app/target/*.jar app.jar

# Extract JAR layers for better caching
RUN java -Djarmode=layertools -jar app.jar extract

# Stage 3: Runtime
FROM eclipse-temurin:17-jre-alpine

LABEL maintainer="your-email@example.com"
LABEL version="1.0"
LABEL description="Spring Boot Application"

# Install curl for health checks
RUN apk add --no-cache curl

WORKDIR /app

# Create non-root user
RUN addgroup -S spring && adduser -S spring -G spring

# Copy layers from extraction stage
COPY --from=layers /app/dependencies/ ./
COPY --from=layers /app/spring-boot-loader/ ./
COPY --from=layers /app/snapshot-dependencies/ ./
COPY --from=layers /app/application/ ./

# Change ownership
RUN chown -R spring:spring /app

# Switch to non-root user
USER spring:spring

# Expose port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=60s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# JVM options optimized for containers
ENV JAVA_OPTS="\
    -XX:+UseContainerSupport \
    -XX:MaxRAMPercentage=75.0 \
    -XX:+UseG1GC \
    -XX:+UseStringDeduplication \
    -Djava.security.egd=file:/dev/./urandom"

# Run application
ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS org.springframework.boot.loader.JarLauncher"]
```

## Building Docker Image

### Build Commands

```bash
# Basic build
docker build -t myapp:latest .

# Build with tag
docker build -t myapp:1.0.0 .

# Build with build args
docker build \
  --build-arg JAR_FILE=target/myapp-1.0.0.jar \
  -t myapp:1.0.0 .

# Build with no cache (force rebuild)
docker build --no-cache -t myapp:latest .

# Multi-platform build
docker buildx build --platform linux/amd64,linux/arm64 -t myapp:latest .
```

### .dockerignore

**`.dockerignore`**:
```
target/
.git/
.gitignore
.mvn/
*.md
*.log
.env
.env.local
.DS_Store
node_modules/
*.iml
.idea/
.vscode/
```

## docker-compose.yml

### Complete Stack (Application + MySQL + Redis)

**`docker-compose.yml`**:
```yaml
version: '3.8'

services:
  # Spring Boot Application
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: myapp-backend
    ports:
      - "8080:8080"
    environment:
      SPRING_PROFILES_ACTIVE: prod
      SPRING_DATASOURCE_URL: jdbc:mysql://mysql:3306/myapp_db?useSSL=false&allowPublicKeyRetrieval=true
      SPRING_DATASOURCE_USERNAME: ${DB_USERNAME:-myapp}
      SPRING_DATASOURCE_PASSWORD: ${DB_PASSWORD:-secret}
      SPRING_DATA_REDIS_HOST: redis
      SPRING_DATA_REDIS_PORT: 6379
      SPRING_DATA_REDIS_PASSWORD: ${REDIS_PASSWORD:-}
      JWT_SECRET: ${JWT_SECRET:-your-secret-key-change-in-production}
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
    networks:
      - myapp-network
    volumes:
      - app-logs:/app/logs
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8080/actuator/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 60s
    restart: unless-stopped

  # MySQL Database
  mysql:
    image: mysql:8.0
    container_name: myapp-mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD:-root_secret}
      MYSQL_DATABASE: myapp_db
      MYSQL_USER: ${DB_USERNAME:-myapp}
      MYSQL_PASSWORD: ${DB_PASSWORD:-secret}
    volumes:
      - mysql-data:/var/lib/mysql
      - ./init-scripts:/docker-entrypoint-initdb.d  # Optional: init scripts
    networks:
      - myapp-network
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-u", "root", "-p${MYSQL_ROOT_PASSWORD:-root_secret}"]
      interval: 10s
      timeout: 3s
      retries: 5
      start_period: 30s
    restart: unless-stopped

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    ports:
      - "6379:6379"
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD:-}
    volumes:
      - redis-data:/data
    networks:
      - myapp-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 3s
      retries: 3
    restart: unless-stopped

volumes:
  mysql-data:
    driver: local
  redis-data:
    driver: local
  app-logs:
    driver: local

networks:
  myapp-network:
    driver: bridge
```

### PostgreSQL Alternative

```yaml
services:
  postgres:
    image: postgres:15-alpine
    container_name: myapp-postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_DB: myapp_db
      POSTGRES_USER: ${DB_USERNAME:-myapp}
      POSTGRES_PASSWORD: ${DB_PASSWORD:-secret}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    networks:
      - myapp-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USERNAME:-myapp}"]
      interval: 10s
      timeout: 3s
      retries: 5
    restart: unless-stopped

volumes:
  postgres-data:
```

## Environment Variables

### .env File

**`.env`** (Don't commit to git):
```bash
# Database
DB_USERNAME=myapp_user
DB_PASSWORD=strong_password_here
MYSQL_ROOT_PASSWORD=root_strong_password

# Redis
REDIS_PASSWORD=redis_strong_password

# Application
SPRING_PROFILES_ACTIVE=prod
JWT_SECRET=your-very-long-secret-key-at-least-512-bits

# Ports
APP_PORT=8080
MYSQL_PORT=3306
REDIS_PORT=6379
```

### .env.example (Commit this)

**`.env.example`**:
```bash
# Database
DB_USERNAME=myapp_user
DB_PASSWORD=changeme
MYSQL_ROOT_PASSWORD=changeme

# Redis
REDIS_PASSWORD=changeme

# Application
SPRING_PROFILES_ACTIVE=prod
JWT_SECRET=changeme

# Ports
APP_PORT=8080
MYSQL_PORT=3306
REDIS_PORT=6379
```

## Docker Commands

### Development Workflow

```bash
# Build and start all services
docker-compose up -d --build

# View logs
docker-compose logs -f app

# View logs for specific service
docker-compose logs -f mysql

# Stop all services
docker-compose down

# Stop and remove volumes (WARNING: deletes data)
docker-compose down -v

# Restart specific service
docker-compose restart app

# Execute command in running container
docker-compose exec app sh

# Run database migrations
docker-compose exec app java -jar app.jar db migrate

# Check service health
docker-compose ps
```

### Production Deployment

```bash
# Pull latest images
docker-compose pull

# Build with no cache
docker-compose build --no-cache

# Start in detached mode
docker-compose up -d

# Scale application (requires load balancer)
docker-compose up -d --scale app=3

# View resource usage
docker stats

# Backup database
docker-compose exec mysql mysqldump -u root -p myapp_db > backup.sql

# Restore database
docker-compose exec -T mysql mysql -u root -p myapp_db < backup.sql
```

## Application Configuration

### application-docker.yml

Create a Docker-specific profile:

**`src/main/resources/application-docker.yml`**:
```yaml
spring:
  datasource:
    url: jdbc:mysql://mysql:3306/myapp_db?useSSL=false
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
    hikari:
      maximum-pool-size: 10
      minimum-idle: 5

  data:
    redis:
      host: redis
      port: 6379
      password: ${REDIS_PASSWORD:}

  flyway:
    enabled: true
    locations: classpath:db/migration

logging:
  level:
    root: INFO
    com.example.myapp: DEBUG
  file:
    name: /app/logs/application.log

management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics
  endpoint:
    health:
      show-details: always
```

### Health Check Endpoint

Ensure Spring Actuator is configured:

**`pom.xml`**:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

## Advanced Configurations

### docker-compose with Nginx Reverse Proxy

**`docker-compose.yml`** (extended):
```yaml
services:
  # Nginx Reverse Proxy
  nginx:
    image: nginx:alpine
    container_name: myapp-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
      - app
    networks:
      - myapp-network
    restart: unless-stopped
```

**`nginx/nginx.conf`**:
```nginx
events {
    worker_connections 1024;
}

http {
    upstream backend {
        server app:8080;
    }

    server {
        listen 80;
        server_name localhost;

        location / {
            proxy_pass http://backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }

        location /actuator/health {
            proxy_pass http://backend/actuator/health;
            access_log off;
        }
    }
}
```

### Resource Limits

```yaml
services:
  app:
    deploy:
      resources:
        limits:
          cpus: '2.0'
          memory: 2G
        reservations:
          cpus: '0.5'
          memory: 512M
    ulimits:
      nofile:
        soft: 65536
        hard: 65536
```

### Logging with ELK Stack

```yaml
services:
  app:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
        labels: "app=myapp"

  # Elasticsearch
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: myapp-elasticsearch
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    volumes:
      - elasticsearch-data:/usr/share/elasticsearch/data
    networks:
      - myapp-network

  # Kibana
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    container_name: myapp-kibana
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
    depends_on:
      - elasticsearch
    networks:
      - myapp-network
```

## Best Practices

### 1. Use Multi-Stage Builds

✅ **Reduces final image size**
```dockerfile
FROM maven:3.9-eclipse-temurin-17-alpine AS build
# ... build steps ...

FROM eclipse-temurin:17-jre-alpine  # Only runtime, not build tools
```

### 2. Layer Caching

✅ **Copy dependencies before source code**
```dockerfile
# Dependencies change less frequently
COPY pom.xml .
RUN mvn dependency:go-offline

# Source changes frequently
COPY src ./src
RUN mvn package
```

### 3. Use Specific Tags

```yaml
# ❌ BAD: Tag can change
image: mysql:latest

# ✅ GOOD: Specific version
image: mysql:8.0.35
```

### 4. Non-Root User

```dockerfile
# Create and use non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring
```

### 5. Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:8080/actuator/health || exit 1
```

### 6. Environment-Specific Configs

```bash
# Development
docker-compose -f docker-compose.yml up

# Production
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up
```

### 7. Secrets Management

```yaml
# Use Docker secrets (Swarm mode)
services:
  app:
    secrets:
      - db_password
      - jwt_secret

secrets:
  db_password:
    external: true
  jwt_secret:
    external: true
```

### 8. Volume Backups

```bash
# Backup MySQL volume
docker run --rm \
  -v myapp_mysql-data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/mysql-backup.tar.gz /data

# Restore MySQL volume
docker run --rm \
  -v myapp_mysql-data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/mysql-backup.tar.gz -C /
```

## Production Checklist

✅ **Security**:
- [ ] Use non-root user in containers
- [ ] Use secrets for sensitive data
- [ ] Enable SSL/TLS for databases
- [ ] Use private Docker registry
- [ ] Scan images for vulnerabilities

✅ **Performance**:
- [ ] Set resource limits (CPU, memory)
- [ ] Optimize JVM options for containers
- [ ] Use connection pooling
- [ ] Enable caching (Redis)
- [ ] Use multi-stage builds

✅ **Reliability**:
- [ ] Configure health checks
- [ ] Set restart policies
- [ ] Use persistent volumes for data
- [ ] Backup strategy in place
- [ ] Monitoring and logging enabled

✅ **Networking**:
- [ ] Use custom networks
- [ ] Configure reverse proxy (Nginx)
- [ ] Use HTTPS in production
- [ ] Firewall rules configured

## Troubleshooting

### Container Won't Start

```bash
# Check logs
docker-compose logs app

# Check container status
docker-compose ps

# Inspect container
docker inspect myapp-backend

# Enter container
docker-compose exec app sh
```

### Database Connection Issues

```bash
# Check if MySQL is ready
docker-compose exec mysql mysqladmin ping -h localhost -u root -p

# Check network connectivity
docker-compose exec app ping mysql

# View MySQL logs
docker-compose logs mysql
```

### Out of Memory

```yaml
# Increase memory limit
services:
  app:
    environment:
      JAVA_OPTS: "-Xmx1024m"
    deploy:
      resources:
        limits:
          memory: 2G
```

### Port Already in Use

```bash
# Find process using port
lsof -i :8080

# Change port in docker-compose.yml
ports:
  - "8081:8080"  # Map to different host port
```

## CI/CD Integration

### GitHub Actions

**`.github/workflows/docker.yml`**:
```yaml
name: Docker Build and Push

on:
  push:
    branches: [main]
    tags: ['v*']

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: myorg/myapp
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

## Related Documentation

- See [database-migrations.md](../configuration/database-migrations.md) for database setup
- See [caching.md](../configuration/caching.md) for Redis configuration
- See [../testing/integration-tests.md](../testing/integration-tests.md) for testing with Docker
