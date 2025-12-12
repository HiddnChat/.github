# HiddnChat Developer Setup

Development environment setup guide for HiddnChat.

## Prerequisites

### Required Software

| Software | Version | Purpose |
|----------|---------|---------|
| Docker | 24+ | Container runtime |
| Docker Compose | v2+ | Multi-container orchestration |
| PHP | 8.4+ | API backend |
| Composer | 2+ | PHP dependencies |
| Node.js | 20+ | MLS API & Web frontend |
| npm | 10+ | Node package manager |
| Rust | 1.83+ | MLS Core |
| protoc | 3.20+ | Protocol Buffers |
| Git | 2.40+ | Version control |

### Optional Tools

| Tool | Purpose |
|------|---------|
| grpcurl | Test gRPC endpoints |
| Postman | API testing |
| DBeaver | Database GUI |
| Redis Commander | Redis GUI |

---

## Quick Setup

### 1. Clone Repositories

```bash
mkdir hiddn && cd hiddn

git clone https://github.com/HiddnChat/api.git
git clone https://github.com/HiddnChat/mls-api.git
git clone https://github.com/HiddnChat/mls-core.git
git clone https://github.com/HiddnChat/web.git
```

### 2. Start Infrastructure

```bash
cd api
docker-compose up -d postgres redis
```

### 3. Setup API

```bash
cd api

# Install dependencies
composer install

# Copy environment
cp .env.example .env

# Generate JWT keys
php bin/console lexik:jwt:generate-keypair

# Run migrations
php bin/console doctrine:migrations:migrate

# Start server
symfony serve
# or: php -S localhost:8000 -t public
```

### 4. Setup MLS Core

```bash
cd mls-core

# Build
cargo build

# Run
GRPC_PORT=50052 cargo run
```

### 5. Setup MLS API

```bash
cd mls-api

# Install dependencies
npm install

# Copy environment
cp .env.example .env

# Run migrations
npx mikro-orm migration:up

# Start in development mode
npm run dev
```

### 6. Setup Web Client

```bash
cd web

# Install dependencies
npm install

# Copy environment
cp .env.example .env.local

# Start dev server
npm run dev
```

---

## Development Ports

| Service | Port | URL |
|---------|------|-----|
| Web | 5173 | http://localhost:5173 |
| API | 8000 | http://localhost:8000 |
| MLS API | 50051 | grpc://localhost:50051 |
| MLS Core | 50052 | grpc://localhost:50052 |
| PostgreSQL | 5432 | localhost:5432 |
| Redis | 6379 | localhost:6379 |

---

## Service Details

### API (Symfony)

```bash
# Development server (with hot reload)
symfony serve

# Run tests
php bin/phpunit

# Code style check
vendor/bin/php-cs-fixer fix --dry-run

# Static analysis
vendor/bin/phpstan analyse

# Create migration
php bin/console doctrine:migrations:diff

# Clear cache
php bin/console cache:clear
```

**Project Structure:**

```
api/
├── config/           # Symfony config
├── migrations/       # Doctrine migrations
├── public/           # Web root
├── src/
│   ├── Controller/   # REST endpoints
│   ├── Entity/       # Doctrine entities
│   ├── Repository/   # Data access
│   ├── Service/      # Business logic
│   └── Enum/         # Type enums
└── tests/            # PHPUnit tests
```

---

### MLS API (Node.js)

```bash
# Development (watch mode)
npm run dev

# Build TypeScript
npm run build

# Run tests
npm test

# Lint
npm run lint

# Create migration
npx mikro-orm migration:create

# Generate proto types
npm run proto:generate
```

**Project Structure:**

```
mls-api/
├── proto/            # gRPC definitions
├── src/
│   ├── cache/        # Redis client
│   ├── database/     # MikroORM setup
│   ├── entities/     # Database entities
│   ├── repositories/ # Data access
│   ├── services/     # MLS operations
│   ├── utils/        # Helpers
│   └── server.ts     # gRPC server
└── tests/            # Jest tests
```

---

### MLS Core (Rust)

```bash
# Build (debug)
cargo build

# Build (release)
cargo build --release

# Run tests
cargo test

# Format code
cargo fmt

# Lint
cargo clippy

# Generate proto
cargo build  # build.rs handles this
```

**Project Structure:**

```
mls-core/
├── proto/            # gRPC definitions
├── src/
│   ├── crypto/       # OpenMLS wrappers
│   ├── grpc/         # gRPC service impl
│   ├── storage/      # State storage
│   └── main.rs       # Entry point
└── tests/            # Integration tests
```

---

### Web (Vue.js)

```bash
# Development server
npm run dev

# Build production
npm run build

# Preview production build
npm run preview

# Run tests
npm run test

# Lint
npm run lint

# Type check
npm run type-check
```

**Project Structure:**

```
web/
├── public/           # Static assets
├── src/
│   ├── assets/       # Images, styles
│   ├── components/   # Vue components
│   ├── composables/  # Composition API
│   ├── router/       # Vue Router
│   ├── services/     # API clients
│   ├── stores/       # Pinia stores
│   └── views/        # Page components
└── tests/            # Vitest tests
```

---

## IDE Setup

### VS Code

**Recommended extensions:**

```json
{
  "recommendations": [
    "bmewburn.vscode-intelephense-client",
    "Vue.volar",
    "rust-lang.rust-analyzer",
    "esbenp.prettier-vscode",
    "dbaeumer.vscode-eslint"
  ]
}
```

### PhpStorm / WebStorm

- Enable Symfony plugin (PhpStorm)
- Enable Vue.js plugin
- Configure PHP interpreter from Docker

---

## Testing

### API Tests

```bash
cd api

# All tests
php bin/phpunit

# Specific test
php bin/phpunit tests/Controller/AuthControllerTest.php

# With coverage
php bin/phpunit --coverage-html coverage
```

### MLS API Tests

```bash
cd mls-api

# All tests
npm test

# Watch mode
npm run test:watch

# Coverage
npm run test:coverage
```

### Web Tests

```bash
cd web

# Unit tests
npm run test:unit

# E2E tests
npm run test:e2e
```

---

## Database Access

### PostgreSQL

```bash
# Connect via Docker
docker exec -it hiddn-postgres psql -U hiddn hiddn_api

# Or with local client
psql -h localhost -U hiddn -d hiddn_api
```

### Redis

```bash
# Connect via Docker
docker exec -it hiddn-redis redis-cli

# Or with local client
redis-cli -h localhost
```

---

## Debugging

### API (Xdebug)

Add to `.env`:

```env
XDEBUG_MODE=debug
XDEBUG_CONFIG="client_host=host.docker.internal"
```

### MLS API (Node.js)

```bash
# Debug with Node inspector
node --inspect dist/server.js

# Or with VS Code launch config
```

### Web (Vue DevTools)

Install Vue DevTools browser extension.

---

## Common Issues

### Port already in use

```bash
# Find process
lsof -i :8000

# Kill process
kill -9 <PID>
```

### Database connection refused

```bash
# Check if PostgreSQL is running
docker ps | grep postgres

# Restart
docker-compose restart postgres
```

### gRPC connection failed

```bash
# Verify MLS Core is running
grpcurl -plaintext localhost:50052 mls.MlsCoreService/HealthCheck

# Check logs
docker-compose logs mls-core
```

### JWT key pair issues

```bash
# Regenerate keys
php bin/console lexik:jwt:generate-keypair --overwrite
```

---

## Git Workflow

### Branch Naming

```
feature/issue-number-description
fix/issue-number-description
docs/description
```

### Commit Messages

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add user presence endpoint
fix: resolve message encryption issue
docs: update API documentation
refactor: simplify auth middleware
test: add conversation tests
```

### Pull Request Process

1. Create feature branch from `main`
2. Make changes and commit
3. Push branch and create PR
4. Wait for CI checks
5. Request review
6. Merge after approval
