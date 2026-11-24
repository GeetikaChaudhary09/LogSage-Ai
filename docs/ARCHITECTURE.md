
# LogSage AI — System Architecture

## 1. Overview

LogSage AI is a developer-focused application observability platform designed to collect, process, analyze, and monitor application logs.

The platform uses a modular Java backend and a separate Python machine learning service.

The architecture prioritizes:

- Maintainability
- Security
- Asynchronous processing
- Scalability
- Fault tolerance
- Data integrity
- Clear service boundaries

**Architecture status:** Planned. Implementation and verification will happen incrementally.

## 2. High-Level Architecture

The system consists of the following components:

| Component | Technology | Responsibility |
|---|---|---|
| Frontend | React, TypeScript | User interface |
| Backend | Java 24, Spring Boot | Core business logic and APIs |
| Database | PostgreSQL | Persistent application data |
| Message Broker | RabbitMQ | Asynchronous event processing |
| Cache | Redis | Caching and rate limiting |
| ML Service | Python, FastAPI | Anomaly detection |
| Reverse Proxy | Nginx | Production routing |
| Infrastructure | Docker | Containerized deployment |

## 3. System Architecture Diagram

```text
                 External Applications
                         |
                         v
                 Log Ingestion API
                         |
                         v
                Validation and Security
                         |
                         v
                    RabbitMQ
                         |
                         v
                 Processing Workers
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
         PostgreSQL    Redis      ML Service
             |           |           |
             |           |           v
             |           |      Predictions
             |           |           |
             +-----------+-----------+
                         |
                         v
                 Incident Engine
                         |
                         v
                  Alert Evaluation
                         |
                         v
                 Backend REST APIs
                         |
                         v
                  React Dashboard
```

## 4. Frontend Architecture

Technology:
- React
- TypeScript
- Vite
- Tailwind CSS
- TanStack Query
- React Router
- React Hook Form
- Zod
- Recharts
- Lucide Icons

Responsibilities:

- Authentication and session handling
- Workspace and project management
- Log exploration and filtering
- Dashboard metrics visualization
- Anomaly investigation
- Incident management
- Alert configuration
- API key management
- Settings and onboarding

Frontend communicates with the backend through versioned REST APIs.

The frontend must never be responsible for enforcing security permissions.

## 5. Backend Architecture

Technology:
- Java 24
- Spring Boot
- Spring Security
- Spring Data JPA
- Spring Validation
- Spring AMQP
- Spring Actuator
- Flyway
- OpenAPI

The backend follows a modular architecture.

Planned modules:

- config
- security
- auth
- organization
- workspace
- project
- apikey
- ingestion
- logs
- processing
- metrics
- anomaly
- incidents
- alerts
- notifications
- audit
- common
- exception

Each module separates its controllers, services, repositories, entities, and DTOs where appropriate.

Controllers handle HTTP requests, services implement business logic, and repositories manage persistence.

JPA entities must not be exposed directly through REST APIs.

## 6. Log Ingestion Workflow

External applications send structured logs through:

POST /api/v1/ingest/logs

Authentication uses:

X-LogSage-API-Key

Planned workflow:

1. Receive the incoming request.
2. Authenticate the ingestion API key.
3. Validate project association and key status.
4. Validate payload size and log schema.
5. Normalize timestamps to UTC.
6. Assign correlation identifiers where required.
7. Publish the event to RabbitMQ.
8. Return an appropriate ingestion response.
9. Process the event asynchronously.
10. Persist validated log records in PostgreSQL.

The API key determines the authorized project. Client-provided organization and project identifiers must never override this authorization.

## 7. Asynchronous Processing

RabbitMQ is responsible for decoupling ingestion from downstream processing.

Planned queue responsibilities:

- logs.ingestion
- logs.processing
- anomaly.analysis
- incidents.events
- notifications.delivery

Reliability requirements:

- Durable queues where appropriate
- Message acknowledgements
- Bounded retries
- Dead Letter Queues
- Idempotent consumers
- Correlation IDs
- Explicit message failure handling

A message must not be acknowledged before successful processing and required persistence.

## 8. Database Architecture

PostgreSQL is the primary persistent data store.

Planned entities:

- users
- organizations
- organization_members
- projects
- api_keys
- logs
- services
- anomaly_predictions
- incidents
- incident_logs
- alert_rules
- alert_history
- audit_logs
- refresh_tokens

Database changes will be managed through Flyway migrations.

Design requirements:

- UUID identifiers where appropriate
- Foreign key constraints
- Unique constraints
- Query indexes
- Transaction management
- Pagination
- Workspace-level data isolation

## 9. Redis Architecture

Redis will support:

- Rate limiting
- Dashboard summary caching
- Frequently accessed data caching
- Temporary processing state

Cache entries will have defined expiration times.

Redis will not be used as the primary persistent store for logs or incidents.

The application must handle Redis unavailability gracefully.

## 10. Machine Learning Architecture

The ML service will be developed independently using Python and FastAPI.

Model:
- Scikit-learn Isolation Forest

Planned endpoints:

GET /health

GET /model/info

POST /predict

The model will operate on engineered numerical features rather than arbitrary raw log messages.

Initial feature candidates:

- error_rate
- request_count
- avg_duration_ms
- max_duration_ms
- p95_duration_ms
- error_count
- warn_count
- unique_services
- unique_endpoints
- 4xx_rate
- 5xx_rate

The training pipeline will include:

- Input validation
- Feature engineering
- Missing-value handling
- Train/test separation
- Model persistence
- Model versioning
- Precision, recall and F1 evaluation
- Confusion matrix

Synthetic training data will be explicitly identified as synthetic.

An inference failure must never be treated as an anomaly prediction.

## 11. Incident and Alert Architecture

The incident engine will group related abnormal events using defined fingerprints and supporting evidence.

Planned incident lifecycle:

OPEN
ACKNOWLEDGED
INVESTIGATING
RESOLVED
IGNORED

Incident records will maintain:

- Severity
- Related service
- First and last occurrence
- Occurrence count
- Supporting log references
- Assignment
- Status history

Alert rules will evaluate configured metrics and thresholds over defined time windows.

Cooldown periods will prevent repeated notifications for the same active condition.

In-app notifications are the initial notification channel.

## 12. Security Architecture

Security requirements:

- Password hashing
- JWT access and refresh token workflow
- Token revocation
- Role-based authorization
- Organization-level isolation
- Secure API key generation
- Hash-only API key storage
- Request validation
- CORS restrictions
- Rate limiting
- Request-size limits
- Environment-based secrets
- Audit events

Roles:

- OWNER
- ADMIN
- MEMBER
- VIEWER

All sensitive authorization checks must be enforced by the backend.

## 13. Failure Handling

The system must handle component failures without silently losing events.

| Component | Failure strategy |
|---|---|
| PostgreSQL | Fail safely; do not acknowledge unpersisted messages |
| RabbitMQ | Durable messaging and controlled retries |
| Redis | Graceful fallback where safe |
| ML Service | Retry and dead-letter workflow |
| Frontend API | Clear loading and error states |
| Backend | Structured errors and health monitoring |

ML unavailability must not prevent valid log ingestion from being accepted when the ingestion queue is operational.

## 14. Deployment Architecture

The planned deployment environment includes:

- Frontend container
- Backend container
- ML service container
- PostgreSQL
- Redis
- RabbitMQ
- Nginx reverse proxy

Docker Compose will be used for local multi-service development.

AWS deployment instructions will be documented separately.

No deployment will be considered complete until it has been executed and verified.

## 15. Architecture Principles

1. Prefer modular design over unnecessary microservices.
2. Keep the Java backend responsible for core business operations.
3. Keep ML inference isolated in the Python service.
4. Use asynchronous processing for expensive downstream operations.
5. Use PostgreSQL as the source of truth.
6. Never fabricate production telemetry or metrics.
7. Enforce security at the backend.
8. Make failures observable and recoverable.
9. Test critical workflows before declaring completion.
10. Keep documentation synchronized with implementation.

## 16. Implementation Status

This document describes the planned architecture.

Each component will be marked implemented only after its code, integration, and relevant tests have been verified.
