# Docker Deployment - Angular 17+

## Overview

This guide covers Docker deployment for Angular applications using multi-stage builds and nginx for production.

## Dockerfile (Multi-Stage Build)

```dockerfile
# Stage 1: Build Angular application
FROM node:20-alpine AS build

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Build for production
RUN npm run build -- --configuration production

# Stage 2: Serve with nginx
FROM nginx:alpine

# Copy custom nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Copy built application from previous stage
COPY --from=build /app/dist/my-angular-app/browser /usr/share/nginx/html

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
```

---

## Nginx Configuration

```nginx
# nginx.conf
events {
  worker_connections 1024;
}

http {
  include /etc/nginx/mime.types;
  default_type application/octet-stream;

  # Logging
  access_log /var/log/nginx/access.log;
  error_log /var/log/nginx/error.log;

  # Performance
  sendfile on;
  tcp_nopush on;
  tcp_nodelay on;
  keepalive_timeout 65;
  types_hash_max_size 2048;

  # Gzip compression
  gzip on;
  gzip_vary on;
  gzip_proxied any;
  gzip_comp_level 6;
  gzip_types
    text/plain
    text/css
    text/xml
    text/javascript
    application/json
    application/javascript
    application/xml+rss
    application/rss+xml
    font/truetype
    font/opentype
    application/vnd.ms-fontobject
    image/svg+xml;

  server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
      expires 1y;
      add_header Cache-Control "public, immutable";
    }

    # Serve index.html for all routes (Angular routing)
    location / {
      try_files $uri $uri/ /index.html;
    }

    # Health check endpoint
    location /health {
      access_log off;
      return 200 "OK";
      add_header Content-Type text/plain;
    }
  }
}
```

---

## Docker Compose

### Basic Configuration

```yaml
# docker-compose.yml
version: '3.8'

services:
  angular-app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
    restart: unless-stopped
    environment:
      - NODE_ENV=production
```

### With Backend Service

```yaml
# docker-compose.yml
version: '3.8'

services:
  angular-app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8080:80"
    depends_on:
      - backend
    environment:
      - API_URL=http://backend:8080
    restart: unless-stopped

  backend:
    image: my-backend:latest
    ports:
      - "8081:8080"
    environment:
      - DATABASE_URL=postgresql://db:5432/mydb
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - postgres-data:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  postgres-data:
```

---

## Environment Variables

### Runtime Environment Configuration

Angular applications are built at compile time, so environment variables need special handling.

**Option 1: Build-time Variables**

```dockerfile
# Dockerfile with build args
FROM node:20-alpine AS build

ARG API_URL
ARG APP_VERSION

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .

# Build with environment variables
RUN API_URL=$API_URL APP_VERSION=$APP_VERSION npm run build -- --configuration production

FROM nginx:alpine
COPY --from=build /app/dist/my-angular-app/browser /usr/share/nginx/html
```

**Build with arguments:**

```bash
docker build --build-arg API_URL=https://api.example.com \
             --build-arg APP_VERSION=1.0.0 \
             -t my-angular-app .
```

**Option 2: Runtime Configuration with env.js**

```nginx
# nginx.conf - Generate env.js at runtime
server {
  location / {
    # Generate env.js from environment variables
    if ($request_uri = /env.js) {
      return 200 'window.ENV = { API_URL: "$API_URL" };';
      add_header Content-Type application/javascript;
    }

    try_files $uri $uri/ /index.html;
  }
}
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html>
<head>
  <script src="/env.js"></script>
</head>
<body>
  <app-root></app-root>
</body>
</html>
```

---

## Build Commands

### Build Docker Image

```bash
# Build image
docker build -t my-angular-app:latest .

# Build with specific tag
docker build -t my-angular-app:1.0.0 .

# Build with build args
docker build \
  --build-arg API_URL=https://api.example.com \
  -t my-angular-app:latest .
```

### Run Container

```bash
# Run container
docker run -d -p 8080:80 my-angular-app:latest

# Run with environment variables
docker run -d -p 8080:80 \
  -e API_URL=https://api.example.com \
  my-angular-app:latest

# Run with custom name
docker run -d -p 8080:80 --name angular-app my-angular-app:latest
```

### Docker Compose

```bash
# Start services
docker-compose up -d

# Build and start
docker-compose up -d --build

# Stop services
docker-compose down

# View logs
docker-compose logs -f angular-app
```

---

## Production Optimization

### .dockerignore

```
# .dockerignore
node_modules
npm-debug.log
dist
.git
.gitignore
README.md
.angular
.vscode
*.spec.ts
e2e
coverage
.env
.env.local
```

### Optimized Dockerfile

```dockerfile
# Multi-stage build with optimizations
FROM node:20-alpine AS build

WORKDIR /app

# Copy only package files first (better caching)
COPY package*.json ./

# Install dependencies with frozen lockfile
RUN npm ci --prefer-offline --no-audit

# Copy source code
COPY . .

# Build with production optimizations
RUN npm run build -- \
  --configuration production \
  --output-hashing=all \
  --optimization=true \
  --build-optimizer=true

# Production stage
FROM nginx:alpine

# Install dumb-init for proper signal handling
RUN apk add --no-cache dumb-init

# Copy nginx configuration
COPY nginx.conf /etc/nginx/nginx.conf

# Copy built app
COPY --from=build /app/dist/my-angular-app/browser /usr/share/nginx/html

# Create non-root user
RUN addgroup -g 1001 -S nginx-user && \
    adduser -S -D -H -u 1001 -h /var/cache/nginx -s /sbin/nologin -G nginx-user -g nginx-user nginx-user

# Change ownership
RUN chown -R nginx-user:nginx-user /usr/share/nginx/html && \
    chown -R nginx-user:nginx-user /var/cache/nginx && \
    chown -R nginx-user:nginx-user /var/log/nginx && \
    chown -R nginx-user:nginx-user /etc/nginx/conf.d

RUN touch /var/run/nginx.pid && \
    chown -R nginx-user:nginx-user /var/run/nginx.pid

# Switch to non-root user
USER nginx-user

EXPOSE 8080

ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["nginx", "-g", "daemon off;"]
```

---

## Health Checks

### Dockerfile Health Check

```dockerfile
FROM nginx:alpine

COPY --from=build /app/dist/my-angular-app/browser /usr/share/nginx/html

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost/health || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose Health Check

```yaml
services:
  angular-app:
    build: .
    healthcheck:
      test: ["CMD", "wget", "--quiet", "--tries=1", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 3s
      retries: 3
      start_period: 5s
```

---

## CI/CD Integration

### GitHub Actions

```yaml
# .github/workflows/docker-build.yml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Login to Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Build and push
        uses: docker/build-push-action@v4
        with:
          context: .
          push: true
          tags: myusername/my-angular-app:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## Best Practices

### ✅ DO

1. **Use multi-stage builds** to reduce image size
2. **Use alpine images** for smaller footprint
3. **Add .dockerignore** to exclude unnecessary files
4. **Use nginx** for serving static files
5. **Enable gzip compression** in nginx
6. **Add security headers** in nginx
7. **Use health checks** for container monitoring
8. **Run as non-root user** in production
9. **Cache dependencies** layer separately
10. **Use specific version tags** (not `latest`)

### ❌ DON'T

1. **Don't include source code** in final image
2. **Don't run as root** in production
3. **Don't hardcode secrets** in Dockerfile
4. **Don't use development server** in production
5. **Don't forget** .dockerignore file

---

## Summary

| Component | Purpose |
|-----------|---------|
| **Multi-stage Build** | Build app + serve with nginx |
| **Nginx** | Serve static files efficiently |
| **Docker Compose** | Orchestrate multiple services |
| **Health Checks** | Monitor container health |
| **CI/CD** | Automate build and deployment |

---

## Related Documentation

- See [../configuration/environments.md](../configuration/environments.md) for environment setup
- See [../best-practices.md](../best-practices.md) for production best practices
