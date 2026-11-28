
# LogSage AI — System Architecture

## 1. High-Level Architecture

```mermaid
flowchart TB
    EXT["External Applications<br/>Java / Python"]
    FE["Frontend<br/>React + TypeScript"]
    API["Backend API<br/>Spring Boot"]
    AUTH["Authentication & RBAC"]
    INGEST["Log Ingestion API"]
    MQ["RabbitMQ<br/>Message Broker"]
    WORKER["Async Processing Workers"]
    PG[("PostgreSQL<br/>Source of Truth")]
    REDIS[("Redis<br/>Cache & Rate Limiting")]
    ML["ML Service<br/>FastAPI + Isolation Forest"]
    INCIDENT["Incident Engine"]
    ALERT["Alert Evaluation Engine"]
    DASH["Observability Dashboard"]

    EXT -->|"Structured Logs + API Key"| INGEST
    FE --> API
    API --> AUTH
    API --> INGEST
    INGEST -->|"Validate & Publish"| MQ
    MQ --> WORKER
    WORKER -->|"Persist Logs"| PG
    WORKER -->|"Feature Windows"| ML
    API <--> REDIS
    ML -->|"Anomaly Results"| INCIDENT
    PG --> INCIDENT
    INCIDENT --> ALERT
    PG --> API
    ALERT --> PG
    API --> DASH
    DASH --> FE
```

## 2. Log Ingestion Workflow

```mermaid
sequenceDiagram
    participant APP as External Application
    participant API as Spring Boot API
    participant MQ as RabbitMQ
    participant WORKER as Processing Worker
    participant DB as PostgreSQL
    participant ML as ML Service

    APP->>API: POST /api/v1/ingest/logs
    API->>API: Validate API Key
    API->>API: Validate Payload
    API->>MQ: Publish Log Event
    MQ-->>API: Publish Confirmation
    API-->>APP: Ingestion Accepted
    MQ->>WORKER: Deliver Message
    WORKER->>DB: Persist Log
    WORKER->>ML: Submit Feature Window
    ML-->>WORKER: Anomaly Prediction
    WORKER->>DB: Persist Processing Results
```

## 3. Core Architecture Principles

- Spring Boot backend follows a modular monolith architecture.
- PostgreSQL is the persistent source of truth.
- RabbitMQ handles asynchronous event processing.
- Redis is used for caching and rate limiting.
- Python ML service operates independently from the main backend.
- Log ingestion must not depend on successful ML inference.
- Frontend displays data retrieved from backend APIs.
- Organization and project isolation must be enforced by the backend.
```
