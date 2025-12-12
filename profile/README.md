# HiddnChat

Secure, end-to-end encrypted messenger built on the MLS (Messaging Layer Security) Protocol.

## Features

- **End-to-End Encrypted** - MLS Protocol (RFC 9420) with forward secrecy
- **Made in Germany** - GDPR-compliant, servers in EU
- **Privacy-First** - Zero metadata collection, optional read receipts
- **Cross-Platform** - Web, Mobile (iOS/Android), Desktop (Windows/macOS/Linux)

## Repositories

| Repository | Description | Stack |
|------------|-------------|-------|
| [api](https://github.com/HiddnChat/api) | REST API backend | Symfony 7, PHP 8.4 |
| [mls-api](https://github.com/HiddnChat/mls-api) | MLS middleware service | Node.js, gRPC |
| [mls-core](https://github.com/HiddnChat/mls-core) | MLS protocol implementation | Rust, OpenMLS |
| [web](https://github.com/HiddnChat/web) | Web application | Vue.js 3, TypeScript |
| [mobile](https://github.com/HiddnChat/mobile) | Mobile application | React Native |
| [desktop](https://github.com/HiddnChat/desktop) | Desktop application | Electron |
| [.github](https://github.com/HiddnChat/.github) | Documentation | - |

## Architecture

```
Clients (Web/Mobile/Desktop)
         │
         ▼
    REST API (Symfony)
         │
    ┌────┴────┐
    ▼         ▼
PostgreSQL  MLS API (Node.js)
    │              │
    │              ▼
    │       MLS Core (Rust/OpenMLS)
    │
Redis
```

## Documentation

- [Architecture](https://github.com/HiddnChat/.github/blob/main/docs/ARCHITECTURE.md)
- [API Reference](https://github.com/HiddnChat/api/blob/main/docs/openapi.yaml)
- [Development Guide](https://github.com/HiddnChat/.github/blob/main/docs/DEVELOPMENT.md)
- [Security](https://github.com/HiddnChat/.github/blob/main/docs/SECURITY.md)

## Status

MVP development - Core messaging functionality complete.

## License

Proprietary - HiddnChat
