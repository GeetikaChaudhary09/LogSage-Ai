
# ADR-0001: Modular Monolith with a Separate ML Service

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI requires several business capabilities, including:

- Authentication and authorization
- Organization and workspace management
- Project and API key management
- Log ingestion and processing
- Metrics and log search
- Anomaly detection
- Incident management
- Alert management
- Notifications and audit logging

The platform also requires Python-based machine learning capabilities.

Building every capability as an independent microservice would introduce additional deployment, networking, monitoring and operational complexity.

Combining Python ML functionality directly into the Java backend would also create unnecessary runtime and dependency coupling.

## Decision

Use a modular monolith for the primary Java Spring Boot backend and maintain a separate Python FastAPI ML service.

The Java backend will be organized into clearly defined business modules.

The Python service will independently handle model training, model information and anomaly inference.

RabbitMQ will support asynchronous processing between application components.

## Backend Modules

The Java backend will contain modules for:

- Authentication
- Organizations
- Workspaces
- Projects
- API keys
- Log ingestion
- Log processing
- Metrics
- Anomaly coordination
- Incidents
- Alerts
- Notifications
- Audit logging

Modules should maintain clear responsibilities and avoid unnecessary coupling.

## ML Service Responsibilities

The Python ML service will handle:

- Feature processing
- Model training
- Model evaluation
- Model version information
- Anomaly inference

The Java backend remains responsible for application authorization, business workflows and incident management.

## Consequences

### Positive

- Simpler initial deployment.
- Clear separation of business responsibilities.
- Easier local development.
- Independent Python ML runtime.
- Reduced operational complexity compared with many microservices.
- Ability to extract modules into separate services later if justified.

### Negative

- The Java backend remains a shared deployment unit.
- Module boundaries require discipline.
- ML service communication introduces network and availability concerns.
- Scaling individual Java modules independently is not possible initially.

## Alternatives Considered

### Full Microservices Architecture

Rejected for the initial version because it introduces unnecessary infrastructure and operational overhead.

### Single Application Including ML

Rejected because Java business logic and Python ML dependencies have different runtime requirements.

## Implementation Notes

- Use package-level module boundaries in the Java backend.
- Keep ML service APIs explicit and versioned.
- Use asynchronous messaging where appropriate.
- Define service health checks.
- Handle ML service failures without blocking log ingestion.

## Review Conditions

Revisit this decision when:

- Individual modules require independent scaling.
- Deployment bottlenecks become measurable.
- Team ownership requires independent release cycles.
- Operational complexity justifies service extraction.
