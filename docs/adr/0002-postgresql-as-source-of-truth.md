
# ADR-0002: PostgreSQL as the Source of Truth

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI requires persistent storage for application and platform data.

The platform manages:

- Users
- Organizations
- Organization memberships
- Projects
- API keys
- Application logs
- Registered services
- Anomaly predictions
- Incidents
- Incident-log relationships
- Alert rules
- Alert history
- Audit logs
- Refresh tokens

These records have relationships, ownership requirements, consistency requirements and authorization boundaries.

The platform also uses Redis for caching and RabbitMQ for asynchronous event processing.

A clear source of truth is required to prevent inconsistent or conflicting application state.

## Decision

PostgreSQL will be the primary persistent relational database and the source of truth for LogSage AI.

All critical application records must be persisted in PostgreSQL.

Redis and RabbitMQ will operate as supporting infrastructure services and will not replace PostgreSQL as the authoritative data store.

Flyway will manage database schema migrations.

## Responsibilities

### PostgreSQL

PostgreSQL will store:

- User and organization records.
- Organization membership and roles.
- Projects and API key metadata.
- Persisted application logs.
- Registered service information.
- Anomaly prediction records.
- Incidents and related evidence.
- Alert rules and alert history.
- Audit events.
- Refresh-token records.

The database will enforce appropriate:

- Primary keys.
- Foreign keys.
- Unique constraints.
- Check constraints.
- Indexes.
- Transaction boundaries.

UUID identifiers will be used where appropriate.

### Redis

Redis will be used for:

- Short-lived dashboard metrics caching.
- Rate limiting.
- Frequently accessed summaries.
- Temporary processing state where appropriate.

Redis data must have appropriate TTL policies.

The application must not depend on Redis as the only copy of critical persistent data.

### RabbitMQ

RabbitMQ will be used for:

- Asynchronous log processing.
- Anomaly analysis events.
- Incident events.
- Notification delivery events.

Messages must be acknowledged only after the required processing and persistence steps succeed.

Message retries and dead-letter handling will protect against processing failures.

## Data Consistency

PostgreSQL transactions will be used for operations that require atomic database updates.

Asynchronous workflows must account for:

- Message delivery failures.
- Duplicate message delivery.
- Consumer retries.
- Idempotent processing.
- Database transaction boundaries.

Where database changes and message publication must remain consistent, an appropriate reliable publishing strategy, such as the transactional outbox pattern, may be adopted.

## Database Design Principles

The schema will follow these principles:

1. Use normalized relational structures where appropriate.
2. Enforce organization-level ownership.
3. Use foreign keys for important relationships.
4. Add indexes based on actual query patterns.
5. Use pagination for log retrieval.
6. Avoid loading unbounded log collections into memory.
7. Store timestamps consistently in UTC.
8. Use database migrations for schema changes.
9. Protect sensitive records.
10. Document retention and deletion policies.

## Migration Strategy

Flyway will manage schema evolution.

Migration requirements:

- Use versioned migration files.
- Keep applied migrations immutable.
- Review schema changes.
- Test migrations against a clean database.
- Test upgrades against an existing database.
- Document breaking schema changes.

## Alternatives Considered

### Redis as the Primary Database

Rejected because Redis is intended for caching and temporary processing support, not as the primary relational store for this platform.

### MongoDB as the Primary Database

Not selected for the initial architecture because the platform has important relational ownership, membership, project, incident and audit relationships.

### Multiple Primary Databases

Rejected for the initial implementation because it would introduce unnecessary consistency and operational complexity.

## Consequences

### Positive

- A clear authoritative data store.
- Strong relational integrity.
- Transaction support.
- Structured querying.
- Reliable organization and project relationships.
- Easier auditability.
- Clear separation between persistence, caching and messaging.

### Negative

- PostgreSQL requires backup and maintenance.
- High-volume log storage requires query and index optimization.
- Database availability is critical for persistence.
- Retention and archival strategies will be needed as data grows.

## Implementation Notes

- Use PostgreSQL 17 for the current development environment.
- Integrate Spring Data JPA with PostgreSQL.
- Configure Flyway migrations.
- Add appropriate database indexes.
- Use Redis only for supporting cache and temporary state.
- Use RabbitMQ for asynchronous processing.
- Test persistence, migrations and organization isolation.

## Review Conditions

Revisit this decision if:

- Log volume requires specialized archival storage.
- Database performance becomes a measurable bottleneck.
- Data retention requirements change.
- A specialized analytical storage system becomes justified.

Any additional storage system must have a clearly documented responsibility and must not create ambiguous ownership of critical records.
