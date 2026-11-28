
# LogSage AI

**Intelligent Application Observability & Anomaly Detection Platform**

LogSage AI is a developer-focused observability platform that helps engineering teams understand application behaviour through centralized log ingestion, real-time processing, machine-learning-based anomaly detection, and evidence-driven incident investigation.

The platform is designed to collect structured logs from external applications, identify unusual patterns, surface potential issues, and help developers investigate application failures from a unified dashboard.

> **Project Status:** IN PROGRESS — Active Development
> Features will be marked complete only after implementation and verification.


## The Problem

Modern applications generate large volumes of logs across different services and environments. Investigating failures often requires developers to switch between tools, manually search logs, identify unusual behaviour, and correlate events.

LogSage AI aims to simplify this workflow through a unified observability platform that connects log collection, analysis, anomaly detection, and incident management.

## Core Capabilities

The following capabilities define the planned platform:

- **Centralized Log Ingestion:** Collect structured logs from external applications through secure APIs.
- **Project Management:** Organize applications, environments, and API credentials.
- **Log Explorer:** Search, filter, and inspect logs using severity, service, time range, and other attributes.
- **Application Metrics:** Generate metrics from actual persisted telemetry.
- **Anomaly Detection:** Identify unusual application behaviour using machine learning.
- **Incident Management:** Group relevant evidence into incidents and support investigation workflows.
- **Alert Management:** Configure threshold-based alert rules and track notification history.
- **Developer Integration:** Provide sample applications and integration documentation.

## System Architecture

LogSage AI follows a modular monolith architecture for the primary backend, with a separately deployed Python ML service.

```text
 External Applications
   (Java / Python)
          |
          v
    Log Ingestion API
          |
          v
    Spring Boot Backend
          |
          v
       RabbitMQ
          |
          v
   Async Processing
          |
     +----+----+
     |         |
     v         v
PostgreSQL   ML Service
(Source       (FastAPI +
of Truth)   Isolation Forest)
     |         |
     +----+----+
          |
          v
   Incident & Alert Engine
          |
          v
    React Dashboard
```

Redis supports caching, rate limiting, and temporary state. PostgreSQL remains the persistent source of truth.

See [Architecture Documentation](docs/ARCHITECTURE.md) for the detailed design.

## Technology Stack

| Layer | Technologies |
|---|---|
| Frontend | React, TypeScript, Vite |
| UI | Tailwind CSS, shadcn/ui-style components |
| Backend | Java 24, Spring Boot, Spring Security |
| Database | PostgreSQL, Spring Data JPA, Flyway |
| Messaging | RabbitMQ, Spring AMQP |
| Caching | Redis |
| ML Service | Python, FastAPI, Scikit-learn |
| Infrastructure | Docker, Docker Compose, Nginx |
| CI/CD | GitHub Actions |
| Testing | JUnit 5, Mockito, Pytest, frontend testing tools |

## Repository Structure

```text
logsage-ai/
|
|-- backend/                  # Spring Boot backend
|
|-- frontend/                 # React + TypeScript frontend
|
|-- ml-service/               # Python ML service
|   |-- app/
|   |-- datasets/
|   |-- training/
|   |-- tests/
|
|-- sample-apps/              # External application examples
|   |-- spring-boot-sample/
|   |-- python-sample/
|
|-- infrastructure/
|   |-- docker/
|   |-- nginx/
|   |-- aws/
|
|-- docs/
|   |-- adr/
|   |-- diagrams/
|
|-- scripts/
|
|-- .github/
|   |-- workflows/
|
|-- .gitignore
|-- README.md
```

## Planned Log Processing Workflow

1. An application sends structured logs using a project-scoped API key.
2. The backend authenticates the key and validates the incoming payload.
3. Valid log events enter RabbitMQ for asynchronous processing.
4. Processing workers persist accepted logs in PostgreSQL.
5. The ML service analyzes numerical time-window features to identify unusual behaviour.
6. The incident engine groups relevant evidence and updates incident states.
7. Alert rules evaluate configured conditions and generate notifications.
8. The frontend displays persisted logs, metrics, anomalies, incidents, and alerts.

The exact processing and failure-handling contracts are documented in the architecture and API specifications.

## Machine Learning

The planned anomaly detection pipeline uses **Isolation Forest** to identify unusual patterns in application telemetry.

Planned numerical features include:

- Request count
- Error and warning counts
- Error rate
- Average and maximum response duration
- P95 response duration
- HTTP 4xx and 5xx rates
- Service and endpoint activity

Model evaluation will include precision, recall, F1-score, and confusion matrices.

Synthetic datasets, where used for development and testing, will be clearly distinguished from real production telemetry.

## Engineering Principles

- PostgreSQL is the source of truth.
- Log ingestion and analysis are designed for asynchronous processing.
- Organization and project data must be isolated.
- API keys and authentication credentials must be securely stored.
- ML failures must not block log ingestion.
- Metrics must come from actual persisted data.
- No fabricated production telemetry or incidents.
- Critical workflows must have automated tests.
- Secrets must never be committed to the repository.

## Documentation

| Document | Description |
|---|---|
| [Architecture](docs/ARCHITECTURE.md) | System architecture and component responsibilities |
| [Development](docs/DEVELOPMENT.md) | Development workflow and conventions |
| [Database](docs/DATABASE.md) | Database design and migration strategy |
| [API](docs/API.md) | API design and endpoint contracts |
| [ML Pipeline](docs/ML_PIPELINE.md) | Anomaly detection and model lifecycle |
| [Security](docs/SECURITY.md) | Authentication, authorization, and security |
| [Deployment](docs/DEPLOYMENT.md) | Infrastructure and deployment strategy |
| [Troubleshooting](docs/TROUBLESHOOTING.md) | Common development issues |
| [Contributing](docs/CONTRIBUTING.md) | Contribution guidelines |
| [Architecture Decisions](docs/adr/) | Architecture Decision Records |

## Development Roadmap

| Phase | Focus | Status |
|---|---|---|
| 1 | Project foundation and architecture documentation | In Progress |
| 2 | Infrastructure, Docker, database, and messaging | Planned |
| 3 | Backend APIs, authentication, and log ingestion | Planned |
| 4 | ML service and anomaly detection | Planned |
| 5 | Frontend dashboard and user workflows | Planned |
| 6 | End-to-end service integration | Planned |
| 7 | Testing, validation, and optimization | Planned |
| 8 | CI/CD, deployment, and final documentation | Planned |

## Security

Security is a core design requirement:

- Password hashing and secure session management.
- Project-scoped API key authentication.
- Backend-enforced role-based access control.
- Organization-level data isolation.
- Input validation and request limits.
- Environment-based secret management.
- Audit logging for important operations.

## License

License selection is pending.
