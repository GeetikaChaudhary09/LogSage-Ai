
# ADR-0012: Docker, Infrastructure and Deployment Architecture

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI consists of multiple application and infrastructure components that must work together reliably.

The platform includes:

- React frontend.
- Java Spring Boot backend.
- Python FastAPI ML service.
- PostgreSQL database.
- Redis cache.
- RabbitMQ message broker.
- Nginx reverse proxy.

Managing these components independently across development, testing and production environments can introduce configuration differences and deployment complexity.

The platform requires a reproducible containerized environment, persistent data storage, health monitoring, secure configuration and a documented deployment strategy.

## Decision

LogSage AI will use Docker and Docker Compose for containerization and local multi-service orchestration.

Each application component will have an independently maintained Docker image.

Docker Compose will coordinate the services required for local development and integration testing.

Nginx will act as the reverse proxy and frontend entry point.

Production deployment will use an environment-appropriate container hosting strategy, with AWS as the initial deployment target.

## Container Architecture

The initial containerized services will include:

### Frontend

- React application.
- TypeScript.
- Vite production build.
- Nginx-based static asset serving where appropriate.

### Backend

- Java 24.
- Spring Boot.
- Spring Security.
- REST API.
- RabbitMQ consumers and publishers.

### ML Service

- Python.
- FastAPI.
- Scikit-learn.
- Model inference endpoints.

### PostgreSQL

- Persistent relational database.
- Flyway schema migrations.
- Organization and project data.
- Logs, incidents, alert rules and related records.

### Redis

- Cache.
- Rate limiting.
- Temporary application state.

### RabbitMQ

- Asynchronous messaging.
- Durable queues.
- Retry and dead-letter handling.

### Nginx

- Reverse proxy.
- Frontend delivery.
- Backend API routing.
- Appropriate request-size and timeout configuration.

## Docker Compose

Docker Compose will define the initial local development environment.

The Compose configuration must support:

- Service definitions.
- Build configuration.
- Container networking.
- Environment configuration.
- Persistent volumes.
- Health checks.
- Service dependencies.
- Restart policies where appropriate.

The expected service topology is:

```text
                       Browser
                          |
                          v
                    Nginx Gateway
                          |
              +-----------+-----------+
              |                       |
              v                       v
         React Frontend          Spring Boot API
                                      |
                    +-----------------+----------------+
                    |                 |                |
                    v                 v                v
                PostgreSQL          Redis           RabbitMQ
                                                        |
                                                        v
                                                 Log Consumers
                                                        |
                                                        v
                                                   ML Service
```

This diagram represents the logical topology. Actual network routing and service dependencies will be finalized during implementation.

## Internal Networking

Application services will communicate through Docker's internal network.

The initial network design must:

- Allow frontend gateway access to required application services.
- Allow backend access to PostgreSQL, Redis and RabbitMQ.
- Allow backend access to the ML service.
- Avoid unnecessary public exposure of internal infrastructure.
- Keep service communication names and ports consistent.

PostgreSQL, Redis and RabbitMQ management interfaces must not be publicly exposed by default.

Only required external entry points should be published to the host.

## Persistent Storage

PostgreSQL data must use a persistent Docker volume.

Database data must survive container restarts and recreation.

RabbitMQ persistence should be configured where required for durable message recovery.

Redis persistence requirements will depend on the type of state stored.

Redis must not become the source of truth for persistent application records.

Model artifacts and other required ML files must use an explicitly defined storage strategy.

## Health Checks

Each critical service should expose or configure an appropriate health check.

Examples:

- Backend: Spring Boot Actuator health endpoint.
- ML service: FastAPI health endpoint.
- PostgreSQL: Database readiness check.
- Redis: Redis health command.
- RabbitMQ: Broker health check.
- Frontend: HTTP availability check.
- Nginx: HTTP health endpoint where configured.

Service startup order must not be treated as proof that a dependency is ready.

Applications must handle temporary dependency unavailability and recover appropriately.

## Service Dependencies

The backend depends on PostgreSQL for persistent application data.

Redis and RabbitMQ are required for their respective caching and asynchronous processing responsibilities.

The backend must not assume that every dependent service is immediately available at startup.

Health checks, retry policies and connection recovery must be implemented.

ML service unavailability must not prevent the core ingestion workflow from operating.

## Environment Configuration

Configuration must be separated from container images.

Environment variables will provide environment-specific settings such as:

- Database connection URL.
- Database username and password.
- Redis connection settings.
- RabbitMQ connection settings.
- JWT signing configuration.
- CORS configuration.
- ML service URL.
- Frontend API base URL.
- Application environment.
- Logging configuration.

A root `.env.example` file and service-specific examples may document required configuration.

Actual `.env` files must not be committed to Git.

## Secrets Management

Production secrets must be supplied through an appropriate secure secret-management mechanism.

Secrets must never be:

- Hardcoded in source code.
- Included in Docker images.
- Committed to Git.
- Exposed in frontend bundles.
- Printed in application logs.

Local development secrets must remain separate from production credentials.

JWT signing keys, database passwords and other sensitive credentials must be rotated according to the security policy.

## Image Building

Each application component will have an appropriate Dockerfile.

Images should:

- Use maintained base images.
- Pin or control dependency versions.
- Prefer multi-stage builds where beneficial.
- Avoid unnecessary build tools in runtime images.
- Run as a non-root user where practical.
- Exclude development files and secrets.
- Minimize image size.
- Support reproducible builds.

Build failures must be visible in the CI pipeline.

## Database Migrations

Flyway will manage PostgreSQL schema migrations.

Migrations must:

- Be version controlled.
- Have unique version identifiers.
- Be applied in a controlled order.
- Avoid destructive changes without an explicit migration plan.
- Be tested against supported database versions.

Application startup and deployment procedures must account for migration failures.

## Local Development Workflow

Developers should be able to:

1. Clone the repository.
2. Configure environment variables.
3. Build required application images.
4. Start infrastructure and application services.
5. Check service health.
6. Run database migrations.
7. Access the frontend.
8. Test backend APIs.
9. Inspect logs and container status.
10. Stop the environment safely.

The exact commands will be documented in the development guide.

## Production Deployment

The initial production deployment target will be AWS.

The deployment architecture may use suitable AWS-managed or container-hosting services based on operational requirements and project scale.

Potential infrastructure components include:

- Container hosting for frontend, backend and ML service.
- Managed PostgreSQL.
- Managed Redis where justified.
- Managed RabbitMQ-compatible messaging or an appropriately hosted broker.
- HTTPS termination.
- Domain and DNS configuration.
- Secure secret management.
- Centralized logs and monitoring.
- Persistent storage and backup services.

The final AWS service selection will be documented in the deployment guide after cost, reliability and operational requirements are evaluated.

## Reverse Proxy and Routing

Nginx will provide the initial reverse-proxy configuration.

It must support:

- Frontend application delivery.
- Backend API routing.
- Appropriate request-size limits.
- Connection and read timeouts.
- Secure headers.
- HTTPS integration in production.
- Correct forwarding of request metadata.

Internal services must not be unnecessarily accessible through public routes.

## Logging and Monitoring

Containerized services must produce structured and useful operational logs.

The deployment environment must support monitoring of:

- Container health.
- Application availability.
- CPU and memory usage.
- Database connectivity.
- Queue health.
- Cache availability.
- API errors.
- Processing failures.
- Restart counts.

Infrastructure monitoring must complement LogSage AI's application-level observability features.

## Backup and Recovery

The production deployment must define:

- PostgreSQL backup frequency.
- Backup retention.
- Restore procedures.
- Recovery objectives.
- Persistent volume management.
- Recovery testing.

Backups must be protected from unauthorized access.

A successful backup operation must not be assumed to guarantee a successful restore.

Restore procedures must be tested.

## CI/CD Integration

GitHub Actions will support automated validation and build workflows.

The pipeline should eventually include:

- Frontend dependency installation and build.
- Backend tests and build.
- ML service tests.
- Static checks.
- Docker image builds.
- Security and dependency checks.
- Deployment workflow for approved environments.

Production deployment must use controlled credentials and an explicit release process.

## Security Requirements

- Avoid running containers as root where practical.
- Keep internal infrastructure private.
- Do not commit secrets.
- Use secure production transport.
- Restrict exposed ports.
- Apply database access controls.
- Use maintained base images.
- Validate container health.
- Protect persistent data.
- Monitor container and application failures.

## Testing Requirements

Planned infrastructure tests:

- Docker image builds.
- Docker Compose configuration validation.
- Service startup.
- Service health checks.
- Internal service connectivity.
- PostgreSQL persistence across restarts.
- Redis connectivity.
- RabbitMQ connectivity.
- Backend-to-ML communication.
- Nginx routing.
- Environment configuration validation.
- Database migration execution.
- Container restart recovery.
- Backup and restore testing.
- CI pipeline validation.

## Alternatives Considered

### Manual Local Service Installation

Not selected as the primary setup because it introduces environment differences and increases onboarding effort.

### Kubernetes from the Beginning

Not selected for the initial project because it introduces additional operational complexity and is unnecessary for the first implementation stage.

Kubernetes may be evaluated later if scale and operational requirements justify it.

### Single Container for Every Component

Rejected because independent services require different runtimes, dependencies and operational lifecycles.

### Docker Compose

Selected for reproducible local development and multi-service orchestration.

## Consequences

### Positive

- Reproducible development environments.
- Independent service images.
- Easier onboarding.
- Better deployment consistency.
- Persistent database storage.
- Health monitoring.
- Clear separation of configuration and application code.
- Foundation for automated CI/CD.

### Negative

- Docker introduces additional learning and setup requirements.
- Container networking requires configuration.
- Persistent volumes and backups need careful management.
- Production deployment requires secure infrastructure configuration.
- Multiple services increase operational monitoring requirements.

## Implementation Notes

- Use Dockerfiles for individual application services.
- Use Docker Compose for local orchestration.
- Configure persistent PostgreSQL storage.
- Add service health checks.
- Keep internal services private.
- Use environment-based configuration.
- Use Flyway for schema migrations.
- Configure Nginx as the reverse proxy.
- Add GitHub Actions validation.
- Document AWS deployment and recovery procedures.
- Verify all services before declaring deployment readiness.

## Review Conditions

Revisit this decision if:

- Deployment scale changes significantly.
- Kubernetes becomes operationally justified.
- Infrastructure cost requires a different architecture.
- Managed AWS services become preferable.
- Availability and recovery requirements change.
- Security review identifies infrastructure weaknesses.
