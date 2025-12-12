# HiddnChat Architecture

Technical architecture documentation for HiddnChat - a secure, end-to-end encrypted messaging platform.

## System Overview

```mermaid
flowchart TB
    subgraph Clients
        WEB[Web App<br/>Vue.js]
        MOBILE[Mobile App<br/>React Native]
        DESKTOP[Desktop App<br/>Electron]
    end

    subgraph Backend
        API[API Server<br/>Symfony :8000]
        MLS_API[MLS API<br/>Node.js :50051]
        MLS_CORE[MLS Core<br/>Rust :50052]
    end

    subgraph Data
        PG[(PostgreSQL)]
        REDIS[(Redis)]
    end

    subgraph Realtime
        PUSHER[Pusher<br/>WebSocket]
    end

    WEB --> API
    MOBILE --> API
    DESKTOP --> API

    WEB -.-> PUSHER
    MOBILE -.-> PUSHER
    DESKTOP -.-> PUSHER

    API --> PG
    API --> REDIS
    API --> PUSHER
    API --> MLS_API

    MLS_API --> MLS_CORE
    MLS_API --> PG
    MLS_API --> REDIS
```

## Service Architecture

### API Server (Symfony)

Primary REST API handling authentication, conversations, and message routing.

```mermaid
flowchart LR
    subgraph API [Symfony API :8000]
        AUTH[Auth<br/>JWT]
        CONV[Conversations]
        MSG[Messages]
        USER[Users]
    end

    subgraph Services
        MLS[MLS API]
        PUSHER[Pusher]
        DB[(PostgreSQL)]
    end

    AUTH --> DB
    CONV --> DB
    MSG --> DB
    MSG --> MLS
    MSG --> PUSHER
    USER --> DB
```

**Responsibilities:**
- User authentication (JWT)
- Conversation management
- Message storage and routing
- User presence
- Push notifications

### MLS API (Node.js)

Middleware service for MLS protocol operations.

```mermaid
flowchart LR
    subgraph MLS_API [MLS API :50051]
        GRP[Group Ops]
        ENC[Encryption]
        KEY[Key Mgmt]
        AUDIT[Audit Log]
    end

    API[Symfony API] --> MLS_API
    MLS_API --> CORE[MLS Core :50052]
    MLS_API --> PG[(PostgreSQL)]
    MLS_API --> REDIS[(Redis)]
```

**Responsibilities:**
- Session management
- Audit logging
- Key package storage
- Rate limiting
- Caching

### MLS Core (Rust)

Core MLS protocol implementation using OpenMLS.

```mermaid
flowchart LR
    subgraph MLS_CORE [MLS Core :50052]
        OPENMLS[OpenMLS]
        CRYPTO[Crypto]
        STATE[Group State]
    end

    MLS_API[MLS API] --> MLS_CORE
```

**Responsibilities:**
- RFC 9420 MLS protocol
- Key package generation
- Group creation/management
- Message encryption/decryption
- Epoch management

## Data Flow

### Message Send Flow

```mermaid
sequenceDiagram
    participant Client
    participant API
    participant MLS_API
    participant MLS_Core
    participant Pusher
    participant DB

    Client->>API: POST /messages/send
    API->>MLS_API: gRPC EncryptMessage
    MLS_API->>MLS_Core: gRPC EncryptMessage
    MLS_Core-->>MLS_API: ciphertext
    MLS_API-->>API: ciphertext
    API->>DB: Store message
    API->>Pusher: Broadcast
    API-->>Client: 201 Created
    Pusher-->>Client: WebSocket event
```

### Group Creation Flow

```mermaid
sequenceDiagram
    participant Client
    participant API
    participant MLS_API
    participant MLS_Core
    participant DB

    Client->>API: POST /conversations
    API->>MLS_API: gRPC CreateGroup
    MLS_API->>MLS_Core: gRPC CreateGroup
    MLS_Core-->>MLS_API: GroupInfo + Welcome
    MLS_API->>DB: Store group state
    MLS_API-->>API: Group created
    API->>DB: Store conversation
    API-->>Client: 201 Conversation
```

### Key Package Flow

```mermaid
sequenceDiagram
    participant Client
    participant API
    participant MLS_API
    participant MLS_Core

    Note over Client: User registration/login

    Client->>API: Generate keys
    API->>MLS_API: gRPC GenerateKeyPackage
    MLS_API->>MLS_Core: gRPC GenerateKeyPackage
    MLS_Core-->>MLS_API: KeyPackage
    MLS_API-->>API: KeyPackage
    API-->>Client: KeyPackage stored
```

## Database Schema

### API Database (PostgreSQL)

```mermaid
erDiagram
    USER ||--o{ CONVERSATION_PARTICIPANT : participates
    USER ||--o{ MESSAGE : sends
    CONVERSATION ||--o{ CONVERSATION_PARTICIPANT : has
    CONVERSATION ||--o{ MESSAGE : contains

    USER {
        uuid id PK
        string username UK
        string password
        boolean is_online
        datetime last_seen_at
        datetime created_at
    }

    CONVERSATION {
        uuid id PK
        string type
        string name
        datetime created_at
        datetime updated_at
    }

    CONVERSATION_PARTICIPANT {
        uuid id PK
        uuid user_id FK
        uuid conversation_id FK
        datetime joined_at
    }

    MESSAGE {
        uuid id PK
        uuid conversation_id FK
        uuid sender_id FK
        text content
        datetime created_at
        datetime read_at
        datetime delivered_at
    }
```

### MLS Database (PostgreSQL)

```mermaid
erDiagram
    MLS_GROUP ||--o{ GROUP_MEMBER : has
    MLS_GROUP ||--o{ AUDIT_LOG : logs
    USER ||--o{ KEY_PACKAGE : owns

    MLS_GROUP {
        uuid id PK
        string group_id UK
        bigint epoch
        string creator_id
        bytes group_state
        datetime created_at
    }

    GROUP_MEMBER {
        uuid id PK
        uuid group_id FK
        string user_id
        datetime joined_at
    }

    KEY_PACKAGE {
        uuid id PK
        string user_id
        bytes key_package
        boolean claimed
        datetime created_at
        datetime expires_at
    }

    AUDIT_LOG {
        uuid id PK
        uuid group_id FK
        string action
        string user_id
        json metadata
        datetime created_at
    }
```

## Security Architecture

### Encryption Layers

```mermaid
flowchart TB
    subgraph Transport
        TLS[TLS 1.3]
    end

    subgraph Application
        JWT[JWT Auth]
        MLS[MLS Protocol]
    end

    subgraph MLS_Features [MLS Security]
        FS[Forward Secrecy]
        PCS[Post-Compromise Security]
        AUTH[Member Authentication]
    end

    TLS --> JWT
    JWT --> MLS
    MLS --> FS
    MLS --> PCS
    MLS --> AUTH
```

### Authentication Flow

```mermaid
sequenceDiagram
    participant Client
    participant API
    participant DB

    Client->>API: POST /auth/login
    API->>DB: Verify credentials
    DB-->>API: User data
    API->>API: Generate JWT
    API-->>Client: JWT token

    Note over Client,API: Subsequent requests

    Client->>API: Request + Bearer token
    API->>API: Verify JWT
    API-->>Client: Protected resource
```

## Deployment Architecture

### Docker Compose (Development)

```mermaid
flowchart TB
    subgraph Docker [Docker Compose]
        API[api :8000]
        MLS_API[mls-api :50051]
        MLS_CORE[mls-core :50052]
        PG[(postgres :5432)]
        REDIS[(redis :6379)]
        WEB[web :5173]
    end

    WEB --> API
    API --> PG
    API --> REDIS
    API --> MLS_API
    MLS_API --> MLS_CORE
    MLS_API --> PG
    MLS_API --> REDIS
```

### Production

```mermaid
flowchart TB
    subgraph CDN
        CF[Cloudflare]
    end

    subgraph LB [Load Balancer]
        NGINX[Nginx]
    end

    subgraph App [Application Servers]
        API1[API #1]
        API2[API #2]
    end

    subgraph MLS [MLS Cluster]
        MLS_API1[MLS API #1]
        MLS_CORE1[MLS Core #1]
    end

    subgraph Data [Data Layer]
        PG[(PostgreSQL<br/>Primary)]
        PG_R[(PostgreSQL<br/>Replica)]
        REDIS[(Redis Cluster)]
    end

    CF --> NGINX
    NGINX --> API1
    NGINX --> API2
    API1 --> MLS_API1
    API2 --> MLS_API1
    MLS_API1 --> MLS_CORE1
    API1 --> PG
    API2 --> PG
    MLS_API1 --> PG
    PG --> PG_R
    API1 --> REDIS
    MLS_API1 --> REDIS
```

## Technology Stack

| Layer | Technology |
|-------|------------|
| Web Frontend | Vue.js 3, TypeScript, Vite |
| Mobile | React Native, Expo |
| Desktop | Electron, Vue.js |
| API | Symfony 7, PHP 8.4 |
| MLS Middleware | Node.js, gRPC |
| MLS Core | Rust, OpenMLS |
| Database | PostgreSQL 15 |
| Cache | Redis 7 |
| Realtime | Pusher |
| Auth | JWT (lexik/jwt-authentication) |

## Port Reference

| Service | Port | Protocol |
|---------|------|----------|
| API | 8000 | HTTP/REST |
| Web | 5173 | HTTP |
| MLS API | 50051 | gRPC |
| MLS Core | 50052 | gRPC |
| PostgreSQL | 5432 | PostgreSQL |
| Redis | 6379 | Redis |
