# HiddnChat

Secure, end-to-end encrypted messaging platform using the MLS protocol.

## Overview

HiddnChat is a privacy-focused messaging platform that implements the MLS (Messaging Layer Security) protocol (RFC 9420) for end-to-end encryption with forward secrecy and post-compromise security.

## Architecture

```
┌──────────────────────────────────────────────────────────┐
│                        Clients                            │
│   ┌──────────┐   ┌──────────┐   ┌──────────┐             │
│   │   Web    │   │  Mobile  │   │ Desktop  │             │
│   │ (Vue.js) │   │ (React   │   │(Electron)│             │
│   │          │   │  Native) │   │          │             │
│   └────┬─────┘   └────┬─────┘   └────┬─────┘             │
│        │              │              │                    │
│        └──────────────┼──────────────┘                    │
│                       │                                   │
│                       ▼                                   │
│              ┌────────────────┐                          │
│              │   REST API     │                          │
│              │  (Symfony)     │                          │
│              │   :8000        │                          │
│              └───────┬────────┘                          │
│                      │                                    │
│         ┌────────────┼────────────┐                      │
│         │            │            │                       │
│         ▼            ▼            ▼                       │
│   ┌──────────┐ ┌──────────┐ ┌──────────┐                │
│   │PostgreSQL│ │  Redis   │ │ MLS API  │                │
│   │          │ │          │ │ (Node.js)│                │
│   │  :5432   │ │  :6379   │ │  :50051  │                │
│   └──────────┘ └──────────┘ └────┬─────┘                │
│                                   │                       │
│                                   ▼                       │
│                            ┌──────────┐                  │
│                            │ MLS Core │                  │
│                            │  (Rust)  │                  │
│                            │  :50052  │                  │
│                            └──────────┘                  │
└──────────────────────────────────────────────────────────┘
```

## Repositories

| Repository | Description | Tech Stack |
|------------|-------------|------------|
| [api](https://github.com/HiddnChat/api) | REST API backend | Symfony 7, PHP 8.4 |
| [mls-api](https://github.com/HiddnChat/mls-api) | MLS middleware service | Node.js, gRPC |
| [mls-core](https://github.com/HiddnChat/mls-core) | MLS protocol implementation | Rust, OpenMLS |
| [web](https://github.com/HiddnChat/web) | Web application | Vue.js 3, TypeScript |
| [mobile](https://github.com/HiddnChat/mobile) | Mobile application | React Native |
| [desktop](https://github.com/HiddnChat/desktop) | Desktop application | Electron, Vue.js |

## Features

### Security
- End-to-end encryption (MLS protocol)
- Forward secrecy
- Post-compromise security
- Zero-knowledge message storage

### Messaging
- 1:1 direct messages
- Group conversations
- Typing indicators
- Read receipts (optional)
- Delivery confirmations
- File sharing

### Platform
- Web, mobile, and desktop clients
- Real-time updates via WebSocket
- User presence
- Premium subscriptions

## Documentation

| Document | Description |
|----------|-------------|
| [Architecture](ARCHITECTURE.md) | System design and diagrams |
| [API Reference](../api/docs/openapi.yaml) | OpenAPI specification |
| [Database Schema](DATABASE.md) | Database documentation |
| [Deployment](DEPLOYMENT.md) | Production deployment guide |
| [Development](DEVELOPMENT.md) | Developer setup guide |
| [Security](SECURITY.md) | Security architecture |
| [Contributing](../CONTRIBUTING.md) | Contribution guidelines |

## Quick Start

### Prerequisites

- Docker 24+
- Docker Compose v2

### Start Development Environment

```bash
# Clone all repositories
git clone https://github.com/HiddnChat/api.git
git clone https://github.com/HiddnChat/mls-api.git
git clone https://github.com/HiddnChat/mls-core.git
git clone https://github.com/HiddnChat/web.git

# Start infrastructure
cd api && docker-compose up -d

# Start API
composer install
php bin/console lexik:jwt:generate-keypair
php bin/console doctrine:migrations:migrate
symfony serve

# Start MLS Core
cd ../mls-core
cargo run

# Start MLS API
cd ../mls-api
npm install && npm run dev

# Start Web Client
cd ../web
npm install && npm run dev
```

Access the application at http://localhost:5173

## Technology Stack

### Backend
- **API**: Symfony 7.2, PHP 8.4
- **MLS Middleware**: Node.js 20, TypeScript, gRPC
- **MLS Core**: Rust 1.83, OpenMLS
- **Database**: PostgreSQL 15
- **Cache**: Redis 7
- **Realtime**: Pusher

### Frontend
- **Web**: Vue.js 3, TypeScript, Vite
- **Mobile**: React Native, Expo
- **Desktop**: Electron, Vue.js

### Protocols
- **Encryption**: MLS (RFC 9420)
- **Transport**: REST, gRPC, WebSocket
- **Authentication**: JWT

## Security

HiddnChat implements a multi-layered security approach:

1. **Transport Security**: TLS 1.3 for all connections
2. **Authentication**: JWT tokens with RS256
3. **Message Encryption**: MLS protocol (RFC 9420)
   - Cipher Suite: X25519, Ed25519, AES-128-GCM, SHA-256
   - Forward secrecy via epoch-based key derivation
   - Post-compromise security via TreeKEM

See [Security Documentation](SECURITY.md) for details.

## License

Proprietary - HiddnChat

## Contact

- Website: https://hiddn.chat
- GitHub: https://github.com/HiddnChat
- Security: security@hiddn.chat
