# HiddnChat Deployment Guide

Deployment documentation for HiddnChat services.

## Prerequisites

- Docker 24+ & Docker Compose v2
- PostgreSQL 15+
- Redis 7+
- Rust 1.83+ (for mls-core)
- Node.js 20+ (for mls-api)
- PHP 8.4+ (for API)
- Pusher account (for WebSocket)

## Quick Start (Development)

### Clone Repositories

```bash
git clone https://github.com/HiddnChat/api.git
git clone https://github.com/HiddnChat/mls-api.git
git clone https://github.com/HiddnChat/mls-core.git
git clone https://github.com/HiddnChat/web.git
```

### Start with Docker Compose

```bash
# From the api directory
cd api
docker-compose up -d

# Start web client
cd ../web
npm install
npm run dev
```

This starts:
- API on http://localhost:8000
- PostgreSQL on localhost:5432
- Redis on localhost:6379
- Web on http://localhost:5173

## Service Configuration

### API (Symfony)

**Environment variables (`.env`):**

```env
# App
APP_ENV=prod
APP_SECRET=your-secret-here

# Database
DATABASE_URL="postgresql://hiddn:password@localhost:5432/hiddn_api"

# JWT
JWT_SECRET_KEY=%kernel.project_dir%/config/jwt/private.pem
JWT_PUBLIC_KEY=%kernel.project_dir%/config/jwt/public.pem
JWT_PASSPHRASE=your-jwt-passphrase

# Pusher
PUSHER_APP_ID=your-app-id
PUSHER_KEY=your-key
PUSHER_SECRET=your-secret
PUSHER_CLUSTER=eu

# MLS
MLS_API_HOST=mls-api
MLS_API_PORT=50051

# CORS
CORS_ALLOW_ORIGIN='^https?://(localhost|127\.0\.0\.1)(:[0-9]+)?$'
```

**Build & Deploy:**

```bash
# Install dependencies
composer install --no-dev --optimize-autoloader

# Generate JWT keys
php bin/console lexik:jwt:generate-keypair

# Run migrations
php bin/console doctrine:migrations:migrate --no-interaction

# Clear cache
php bin/console cache:clear --env=prod

# Start with PHP-FPM
php-fpm -D
```

---

### MLS API (Node.js)

**Environment variables (`.env`):**

```env
# gRPC Server
GRPC_HOST=0.0.0.0
GRPC_PORT=50051

# MLS Core
MLS_CORE_HOST=mls-core
MLS_CORE_PORT=50052

# PostgreSQL
POSTGRES_HOST=postgres
POSTGRES_PORT=5432
POSTGRES_DB=mls_db
POSTGRES_USER=mls
POSTGRES_PASSWORD=password

# Redis
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=

# Logging
LOG_LEVEL=info
NODE_ENV=production
```

**Build & Deploy:**

```bash
# Install dependencies
npm ci --production

# Build TypeScript
npm run build

# Run migrations
npx mikro-orm migration:up

# Start
node dist/server.js
```

---

### MLS Core (Rust)

**Environment variables:**

```env
GRPC_HOST=0.0.0.0
GRPC_PORT=50052
RUST_LOG=info
```

**Build & Deploy:**

```bash
# Build release
cargo build --release

# Run
./target/release/mls-core
```

---

### Web Client (Vue.js)

**Environment variables (`.env.production`):**

```env
VITE_API_URL=https://api.hiddn.chat
VITE_PUSHER_KEY=your-key
VITE_PUSHER_CLUSTER=eu
```

**Build & Deploy:**

```bash
# Install dependencies
npm ci

# Build for production
npm run build

# Serve with Nginx
# Output is in dist/
```

---

## Docker Compose (Full Stack)

```yaml
version: '3.8'

services:
  # PostgreSQL
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: hiddn
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: hiddn_api
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U hiddn"]
      interval: 5s
      timeout: 5s
      retries: 5

  # Redis
  redis:
    image: redis:7-alpine
    command: redis-server --appendonly yes
    volumes:
      - redis_data:/data
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 5s
      retries: 5

  # MLS Core (Rust)
  mls-core:
    build: ./mls-core
    environment:
      GRPC_PORT: 50052
      RUST_LOG: info
    healthcheck:
      test: ["CMD", "grpcurl", "-plaintext", "localhost:50052", "mls.MlsCoreService/HealthCheck"]
      interval: 10s
      timeout: 5s
      retries: 3

  # MLS API (Node.js)
  mls-api:
    build: ./mls-api
    environment:
      GRPC_PORT: 50051
      MLS_CORE_HOST: mls-core
      MLS_CORE_PORT: 50052
      POSTGRES_HOST: postgres
      POSTGRES_PORT: 5432
      POSTGRES_DB: mls_db
      POSTGRES_USER: hiddn
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      REDIS_HOST: redis
      REDIS_PORT: 6379
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_healthy
      mls-core:
        condition: service_healthy

  # API (Symfony)
  api:
    build: ./api
    environment:
      APP_ENV: prod
      DATABASE_URL: postgresql://hiddn:${POSTGRES_PASSWORD}@postgres:5432/hiddn_api
      MLS_API_HOST: mls-api
      MLS_API_PORT: 50051
      PUSHER_APP_ID: ${PUSHER_APP_ID}
      PUSHER_KEY: ${PUSHER_KEY}
      PUSHER_SECRET: ${PUSHER_SECRET}
      PUSHER_CLUSTER: ${PUSHER_CLUSTER}
    ports:
      - "8000:8000"
    depends_on:
      postgres:
        condition: service_healthy
      mls-api:
        condition: service_started

  # Web Client (Nginx)
  web:
    build: ./web
    ports:
      - "80:80"
    depends_on:
      - api

volumes:
  postgres_data:
  redis_data:
```

---

## Production Deployment

### Nginx Configuration

```nginx
# API
upstream api {
    server 127.0.0.1:8000;
}

server {
    listen 443 ssl http2;
    server_name api.hiddn.chat;

    ssl_certificate /etc/letsencrypt/live/api.hiddn.chat/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.hiddn.chat/privkey.pem;

    location / {
        proxy_pass http://api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

# Web
server {
    listen 443 ssl http2;
    server_name app.hiddn.chat;

    ssl_certificate /etc/letsencrypt/live/app.hiddn.chat/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/app.hiddn.chat/privkey.pem;

    root /var/www/hiddn/web/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
}
```

### SSL/TLS

```bash
# Install certbot
apt install certbot python3-certbot-nginx

# Generate certificates
certbot --nginx -d api.hiddn.chat -d app.hiddn.chat

# Auto-renewal (cron)
0 0 * * * certbot renew --quiet
```

### Systemd Services

**API Service (`/etc/systemd/system/hiddn-api.service`):**

```ini
[Unit]
Description=HiddnChat API
After=network.target postgresql.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/var/www/hiddn/api
ExecStart=/usr/bin/php -S 0.0.0.0:8000 -t public
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**MLS API Service (`/etc/systemd/system/hiddn-mls-api.service`):**

```ini
[Unit]
Description=HiddnChat MLS API
After=network.target

[Service]
Type=simple
User=hiddn
WorkingDirectory=/var/www/hiddn/mls-api
ExecStart=/usr/bin/node dist/server.js
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

**MLS Core Service (`/etc/systemd/system/hiddn-mls-core.service`):**

```ini
[Unit]
Description=HiddnChat MLS Core
After=network.target

[Service]
Type=simple
User=hiddn
WorkingDirectory=/var/www/hiddn/mls-core
ExecStart=/var/www/hiddn/mls-core/target/release/mls-core
Restart=always
RestartSec=5
Environment=RUST_LOG=info
Environment=GRPC_PORT=50052

[Install]
WantedBy=multi-user.target
```

---

## Health Checks

### API

```bash
curl http://localhost:8000/api/status
```

### MLS API

```bash
grpcurl -plaintext localhost:50051 mls.MLSService/HealthCheck
```

### MLS Core

```bash
grpcurl -plaintext localhost:50052 mls.MlsCoreService/HealthCheck
```

---

## Monitoring

### Recommended Stack

- **Metrics**: Prometheus + Grafana
- **Logs**: Loki or ELK Stack
- **Uptime**: UptimeRobot or Pingdom

### Prometheus Endpoints

| Service | Endpoint |
|---------|----------|
| API | `/metrics` (with Prometheus bundle) |
| MLS API | Custom metrics via Pino |
| PostgreSQL | postgres_exporter |
| Redis | redis_exporter |

---

## Backup & Recovery

### Database Backup

```bash
# Backup API database
pg_dump -U hiddn hiddn_api > backup_api_$(date +%Y%m%d).sql

# Backup MLS database
pg_dump -U mls mls_db > backup_mls_$(date +%Y%m%d).sql
```

### Redis Backup

```bash
# Create RDB snapshot
redis-cli BGSAVE

# Copy dump file
cp /var/lib/redis/dump.rdb /backup/redis_$(date +%Y%m%d).rdb
```

### Recovery

```bash
# Restore PostgreSQL
psql -U hiddn hiddn_api < backup_api.sql

# Restore Redis
cp backup_redis.rdb /var/lib/redis/dump.rdb
systemctl restart redis
```

---

## Scaling

### Horizontal Scaling

| Service | Scaling Strategy |
|---------|------------------|
| API | Load balancer + multiple instances |
| MLS API | Single instance (stateful) |
| MLS Core | Single instance (stateful) |
| PostgreSQL | Primary + read replicas |
| Redis | Redis Cluster |

### Load Balancer Config

```nginx
upstream api_cluster {
    least_conn;
    server api1:8000;
    server api2:8000;
    server api3:8000;
}
```
