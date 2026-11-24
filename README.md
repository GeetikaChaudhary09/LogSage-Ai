# LogSage AI

**Intelligent Application Observability & Anomaly Detection Platform**

LogSage AI is a developer-focused observability platform designed to collect, process, search, and analyze application logs, detect unusual behaviour using machine learning, and support incident investigation.

## Project Status

IN PROGRESS Under active development.

Features will be marked complete only after implementation and verification.

## Core Capabilities

- Secure user authentication and workspace isolation
- Project and API key management
- Structured log ingestion
- Log search, filtering, and pagination
- Metrics derived from actual persisted logs
- Machine-learning-based anomaly detection
- Incident management and investigation
- Configurable alert rules and notification history
- Developer onboarding and integration examples

## Technology Stack

- **Frontend:** React, TypeScript, Vite
- **Backend:** Java 24, Spring Boot, Spring Security
- **Database:** PostgreSQL, Flyway
- **Messaging:** RabbitMQ
- **Caching:** Redis
- **ML Service:** Python, FastAPI, Scikit-learn
- **Infrastructure:** Docker, Docker Compose, Nginx
- **CI/CD:** GitHub Actions

## Repository Structure

```text
logsage-ai/
|-- frontend/
|-- backend/
|-- ml-service/
|-- sample-apps/
|-- infrastructure/
|-- docs/
|-- scripts/
\-- .github/
    \-- workflows/
```

## Documentation

- Architecture: `docs/ARCHITECTURE.md`
- Development: `docs/DEVELOPMENT.md`
- Database: `docs/DATABASE.md`
- API: `docs/API.md`
- ML Pipeline: `docs/ML_PIPELINE.md`
- Security: `docs/SECURITY.md`
- Deployment: `docs/DEPLOYMENT.md`
- Troubleshooting: `docs/TROUBLESHOOTING.md`
- Contributing: `docs/CONTRIBUTING.md`

## Quality and Security

- No fabricated production telemetry or metrics.
- Secrets must never be committed.
- Authorization must be enforced on the backend.
- Critical workflows will have automated tests.
- Known limitations will be documented.

## License

License selection is pending.
