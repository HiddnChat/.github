# HiddnChat Security Documentation

Security architecture and best practices for HiddnChat.

## Overview

HiddnChat is built with security-first principles, implementing end-to-end encryption using the MLS (Messaging Layer Security) protocol as defined in RFC 9420.

## Encryption Architecture

### Layers of Security

```
┌─────────────────────────────────────────────────┐
│                 Transport Layer                  │
│                   TLS 1.3                        │
├─────────────────────────────────────────────────┤
│              Authentication Layer                │
│                JWT Tokens                        │
├─────────────────────────────────────────────────┤
│               Message Layer                      │
│           MLS Protocol (RFC 9420)               │
│  ┌─────────────────────────────────────────┐    │
│  │ Forward Secrecy + Post-Compromise Sec.  │    │
│  └─────────────────────────────────────────┘    │
└─────────────────────────────────────────────────┘
```

### MLS Protocol

HiddnChat uses the Messaging Layer Security protocol for end-to-end encryption:

| Feature | Implementation |
|---------|----------------|
| Protocol | RFC 9420 MLS |
| Library | OpenMLS (Rust) |
| Cipher Suite | MLS_128_DHKEMX25519_AES128GCM_SHA256_Ed25519 |
| Key Exchange | X25519 |
| Signature | Ed25519 |
| AEAD | AES-128-GCM |
| Hash | SHA-256 |

### Security Properties

#### Forward Secrecy

- Each message is encrypted with a unique key derived from the current epoch
- Compromising current keys does not reveal past messages
- Achieved through epoch-based key derivation

#### Post-Compromise Security

- Automatic key rotation on group membership changes
- TreeKEM algorithm for efficient group key updates
- Self-healing after device compromise

#### Message Authentication

- All messages signed by sender's identity key
- Recipients verify sender authenticity
- Prevents message forgery

---

## Authentication

### JWT Tokens

| Property | Value |
|----------|-------|
| Algorithm | RS256 |
| Expiration | 1 hour |
| Refresh | Token rotation |

### Token Structure

```json
{
  "iat": 1702400000,
  "exp": 1702403600,
  "sub": "user-uuid",
  "roles": ["ROLE_USER"]
}
```

### Password Storage

- Algorithm: bcrypt
- Cost factor: 13
- Salted automatically

---

## Data Protection

### Data at Rest

| Data | Protection |
|------|------------|
| Messages | E2E encrypted (MLS) |
| Passwords | bcrypt hashed |
| Private Keys | Encrypted at rest |
| Database | Encrypted volumes |

### Data in Transit

| Protocol | Version |
|----------|---------|
| HTTPS | TLS 1.3 |
| gRPC | TLS 1.3 |
| WebSocket | WSS (TLS 1.3) |

### Key Storage

```
┌─────────────────────────────────────┐
│          Client Device              │
│  ┌─────────────────────────────┐    │
│  │   Identity Private Key      │    │
│  │   (encrypted with passphrase)│   │
│  └─────────────────────────────┘    │
│  ┌─────────────────────────────┐    │
│  │   Key Packages (ephemeral)  │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘

┌─────────────────────────────────────┐
│            Server                   │
│  ┌─────────────────────────────┐    │
│  │   Identity Public Keys      │    │
│  │   (for verification)        │    │
│  └─────────────────────────────┘    │
│  ┌─────────────────────────────┐    │
│  │   Key Packages (public)     │    │
│  │   (for group joins)         │    │
│  └─────────────────────────────┘    │
└─────────────────────────────────────┘
```

---

## API Security

### Rate Limiting

| Endpoint | Limit |
|----------|-------|
| `/auth/register` | 5/hour per IP |
| `/auth/login` | 10/minute per IP |
| `/messages/send` | 60/minute per user |
| General | 100/minute per user |

### Input Validation

- All inputs validated via Symfony Validator
- SQL injection prevented by Doctrine ORM
- XSS prevented by output escaping
- CSRF tokens for web forms

### CORS Configuration

```php
// Production CORS
$corsOrigins = [
    'https://app.hiddn.chat',
    'https://hiddn.chat',
];
```

---

## Privacy Features

### Read Receipts

- Optional per-user setting
- Controlled by `allowReadReceipts` flag
- Users can disable to hide read status

### Online Status

- Optional visibility
- Users can appear offline
- Last seen timestamp configurable

### Message Deletion

- Soft delete (reversible by admin)
- Hard delete after retention period
- E2E content not stored on server

---

## Audit Logging

### Logged Events

| Event | Data Captured |
|-------|---------------|
| Login | User, IP, timestamp |
| Failed login | IP, timestamp, username |
| Registration | User, IP |
| Message send | User, conversation, timestamp |
| Group changes | User, action, members |
| Key operations | User, action type |

### GDPR Compliance

| Requirement | Implementation |
|-------------|----------------|
| Right to access | Export endpoint |
| Right to erasure | Account deletion |
| Data portability | JSON export |
| Consent | Explicit opt-in |

---

## Threat Model

### Protected Against

| Threat | Mitigation |
|--------|------------|
| Eavesdropping | E2E encryption (MLS) |
| Server compromise | Zero-knowledge of content |
| Man-in-the-middle | TLS + certificate pinning |
| Replay attacks | Epoch-based encryption |
| Key compromise | Forward secrecy |
| Account takeover | MFA (planned) |

### Not Protected Against

| Threat | Reason |
|--------|--------|
| Device compromise | Client security responsibility |
| Screenshot/copy | User behavior |
| Social engineering | User awareness |
| Metadata analysis | Partially visible to server |

---

## Security Headers

```nginx
# API Server
add_header X-Frame-Options "DENY";
add_header X-Content-Type-Options "nosniff";
add_header X-XSS-Protection "1; mode=block";
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains";
add_header Content-Security-Policy "default-src 'self'";
add_header Referrer-Policy "strict-origin-when-cross-origin";
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()";
```

---

## Vulnerability Reporting

### Responsible Disclosure

If you discover a security vulnerability:

1. **Do not** disclose publicly
2. Email: security@hiddn.chat
3. Include detailed reproduction steps
4. Allow 90 days for fix before disclosure

### Bug Bounty

Currently no formal bug bounty program. Researchers will be credited.

---

## Security Checklist

### Development

- [ ] No secrets in code
- [ ] Dependencies up to date
- [ ] Input validation on all endpoints
- [ ] Output encoding
- [ ] Parameterized queries
- [ ] Secure password hashing

### Deployment

- [ ] TLS 1.3 only
- [ ] Security headers configured
- [ ] Rate limiting enabled
- [ ] Logging enabled
- [ ] Backups encrypted
- [ ] Access controls in place

### Operations

- [ ] Regular security updates
- [ ] Log monitoring
- [ ] Incident response plan
- [ ] Key rotation schedule
- [ ] Access review quarterly

---

## Cryptographic Details

### Key Derivation

```
Identity Key (Ed25519)
    │
    └─► Sign Key Packages
         │
         └─► Pre-Key (X25519)
              │
              └─► Group Join
                   │
                   └─► Epoch Secret
                        │
                        └─► Message Keys
```

### Message Encryption

```
Plaintext
    │
    ├─► AEAD Encrypt (AES-128-GCM)
    │       │
    │       ├─► Key: derived from epoch secret
    │       └─► Nonce: unique per message
    │
    └─► MLS PrivateMessage
         │
         └─► Authenticated by sender signature
```

---

## Compliance

| Standard | Status |
|----------|--------|
| GDPR | Compliant |
| SOC 2 | Planned |
| ISO 27001 | Planned |
| HIPAA | Not applicable |
