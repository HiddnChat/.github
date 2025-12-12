# HiddnChat

Secure, end-to-end encrypted messenger built on the MLS (Messaging Layer Security) Protocol.

## Features

- **End-to-End Encrypted** - MLS Protocol (RFC 9420) with forward secrecy
- **Made in Germany** - GDPR-compliant, servers in EU
- **Privacy-First** - Zero metadata collection, optional read receipts
- **Cross-Platform** - Web, Mobile (iOS/Android), Desktop (Windows/macOS/Linux)

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

## Status

MVP development - Core messaging functionality complete.

## License

Proprietary - HiddnChat
