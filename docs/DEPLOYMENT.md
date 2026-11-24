
# LogSage AI - Deployment Guide

## 1. Overview

This document describes the planned deployment strategy for LogSage AI.

The platform will support local development using Docker Compose and a production deployment on AWS.

Deployment instructions will be updated as infrastructure and application components are implemented and tested.

**Status:** Planned. No production deployment is currently verified.

## 2. Deployment Architecture

The planned deployment includes:

- React frontend
- Java Spring Boot backend
- Python FastAPI ML service
- PostgreSQL database
- Redis cache
- RabbitMQ message broker
- Nginx reverse proxy

Production services should run in a controlled private network wherever possible.

## 3. Local Development Deployment

Local development will use Docker Compose for infrastructure services.

Planned infrastructure containers:

- PostgreSQL
- Redis
- RabbitMQ

The frontend, backend and ML service may initially run directly on the development machine.

### Prerequisites

- Git
- Java 24
- Node.js 24
- Python version supported by the selected ML dependencies
- Docker Desktop
- Docker Compose

### Clone the Repository

```bash
git clone https://github.com/GeetikaChaudhary09/LogSage-Ai.git
cd LogSage-Ai
```

### Environment Configuration

Create local environment files using the documented examples.

Never commit real credentials.

Example configuration categories:

- Database connection
- Redis connection
- RabbitMQ connection
- JWT signing configuration
- ML service URL
- Frontend API URL

### Start Infrastructure

Once Docker Compose configuration is implemented:

```bash
docker compose up -d
```

Check running containers:

```bash
docker compose ps
```

View service logs:

```bash
docker compose logs
```

Stop services:

```bash
docker compose down
```

Persistent database data must be stored in Docker volumes.

## 4. Backend Deployment

The backend will be built using the Maven Wrapper.

Planned build command:

```bash
./mvnw clean package
```

On Windows PowerShell:

```powershell
.\mvnw.cmd clean package
```

The generated application artifact will be packaged as a JAR.

Run the application using the configured environment:

```bash
java -jar target/application.jar
```

The actual artifact name and production run configuration will be finalized during implementation.

## 5. Frontend Deployment

The frontend will use Vite for building production assets.

Install dependencies:

```bash
npm install
```

Create a production build:

```bash
npm run build
```

The generated static assets will be served through Nginx or an appropriate static hosting service.

Frontend environment variables must use public configuration only.

Never expose backend secrets or signing keys in frontend bundles.

## 6. ML Service Deployment

The ML service will use FastAPI.

Create and activate a Python virtual environment.

Install dependencies from the finalized dependency file.

Planned local startup command:

```bash
uvicorn app.main:app --host 0.0.0.0 --port 8000
```

Production configuration must include:

- Model artifact location
- Service port
- Logging configuration
- Health endpoint
- Resource limits
- Model version information

The ML service must not block log ingestion when inference is unavailable.

## 7. Nginx Configuration

Nginx will act as the reverse proxy and frontend static-file server.

Planned responsibilities:

- Serve frontend assets
- Route API requests to the backend
- Forward requests to configured services
- Configure request-size limits
- Support HTTPS termination in production
- Apply appropriate security headers

Internal service ports should not be exposed publicly unless required.

## 8. Database Deployment

PostgreSQL is the primary persistent data store.

Deployment requirements:

- Persistent storage
- Restricted database access
- Strong credentials
- Automated schema migrations through Flyway
- Backup and recovery procedures
- Connection pooling
- Health checks

Redis and RabbitMQ are supporting infrastructure and must not replace PostgreSQL as the source of truth.

## 9. AWS Production Deployment

The initial production deployment target is AWS.

The final AWS architecture will be selected after evaluating cost, operational complexity and expected workload.

Potential AWS services:

| Requirement | Possible AWS service |
|---|---|
| Application containers | ECS |
| Container registry | ECR |
| Frontend hosting | S3 and CloudFront |
| PostgreSQL | Amazon RDS |
| Secrets | AWS Secrets Manager |
| DNS | Route 53 |
| HTTPS certificates | AWS Certificate Manager |
| Monitoring | CloudWatch |

Redis and RabbitMQ hosting options will be selected according to the final infrastructure design.

These are candidate services, not confirmed deployed resources.

## 10. Environment and Secrets

Production configuration must be supplied securely.

Required configuration categories include:

- Database URL and credentials
- Redis connection
- RabbitMQ connection
- JWT signing keys
- ML service endpoint
- CORS allowed origins
- Frontend public API URL

Do not store production secrets in:

- Git repositories
- Docker images
- Frontend bundles
- Application logs
- Public documentation

Use an appropriate secrets manager for production credentials.

## 11. CI/CD Pipeline

GitHub Actions will be used for continuous integration.

Planned pipeline stages:

1. Checkout source code.
2. Set up required runtimes.
3. Install dependencies.
4. Run backend tests.
5. Run ML service tests.
6. Run frontend tests.
7. Run build checks.
8. Perform dependency security checks.
9. Build container images.
10. Publish images when deployment configuration is ready.

Production deployment must require appropriate credentials and controlled approval.

## 12. Health Checks and Monitoring

Every deployable service must expose or provide a reliable health check.

Planned checks:

- Backend: Spring Boot Actuator health endpoint
- ML service: `/health`
- PostgreSQL: database readiness
- Redis: connection health
- RabbitMQ: broker health
- Frontend: HTTP availability

Health checks must distinguish between service availability and full application readiness.

## 13. Backup and Recovery

Production data protection must include:

- Scheduled database backups
- Backup retention policy
- Recovery procedure
- Restore testing
- Access restrictions
- Documented recovery responsibilities

Backup success must be verified rather than assumed.

## 14. Deployment Security

Production deployment must enforce:

- HTTPS
- Secure secret management
- Restricted network access
- Least-privilege IAM permissions
- Database access restrictions
- Container image review
- Dependency vulnerability checks
- Secure logging
- Regular backup verification

## 15. Rollback Strategy

A deployment rollback procedure should include:

1. Identify the failed release.
2. Stop or redirect traffic from the unhealthy release.
3. Restore the last known healthy application version.
4. Verify service health.
5. Check database migration compatibility.
6. Review logs and incidents.
7. Document the cause and resolution.

Database migrations must be designed carefully because application rollback does not automatically reverse schema changes.

## 16. Production Readiness Checklist

- [ ] All required services build successfully.
- [ ] Environment configuration is validated.
- [ ] No production secrets are committed.
- [ ] Database persistence is verified.
- [ ] Database migrations run successfully.
- [ ] Authentication and authorization are tested.
- [ ] Health checks are working.
- [ ] Logging and error handling are configured.
- [ ] Backup and restore procedures are tested.
- [ ] HTTPS and network restrictions are configured.
- [ ] CI pipeline passes.
- [ ] Rollback procedure is documented.
- [ ] End-to-end workflows are verified.

## 17. Implementation Status

This document describes the intended deployment architecture.

Deployment will be marked complete only after:

1. Local Docker Compose startup is verified.
2. All required services pass health checks.
3. Application builds succeed.
4. Persistent storage is tested.
5. CI workflows pass.
6. Production deployment is performed and verified.
7. Deployment and rollback procedures are tested.
