
# LogSage AI — Database Design

## 1. Overview

LogSage AI uses PostgreSQL as its primary persistent relational database.

The database is responsible for storing:

- User accounts
- Organizations and memberships
- Projects
- Ingestion API keys
- Application logs
- Registered services
- Anomaly predictions
- Incidents and incident evidence
- Alert rules and alert history
- Audit events
- Refresh tokens

Database schema changes will be managed using Flyway migrations.

**Status:** Planned database design. Tables, constraints, indexes and migrations will be implemented and tested incrementally.

## 2. Database Technology

| Technology | Responsibility |
|---|---|
| PostgreSQL 17 | Primary relational database |
| Spring Data JPA | Object-relational persistence |
| Flyway | Version-controlled schema migrations |
| UUID | Entity identifiers |
| JSONB | Flexible structured metadata |

PostgreSQL will remain the source of truth for persistent application data.

Redis will not replace PostgreSQL for storing logs or incidents.

## 3. Entity Overview

The planned database entities are:

| Entity | Purpose |
|---|---|
| users | Registered user accounts |
| organizations | Organization/workspace ownership |
| organization_members | User membership and roles |
| projects | Monitoring projects |
| api_keys | Hashed ingestion credentials |
| logs | Persisted application log events |
| services | Monitored application services |
| anomaly_predictions | ML prediction results |
| incidents | Incident records and lifecycle |
| incident_logs | Incident-to-log relationships |
| alert_rules | Configured alert conditions |
| alert_history | Alert evaluation and notification history |
| audit_logs | Security-sensitive activity history |
| refresh_tokens | Refresh-token lifecycle |

## 4. Entity Relationships

Planned relationships:

- One organization can contain multiple members.
- One user can belong to multiple organizations.
- One organization can contain multiple projects.
- One project can have multiple API keys.
- One project can contain multiple services.
- One project can contain multiple logs.
- One log can have associated anomaly analysis results.
- One incident can reference multiple logs.
- One project can contain multiple alert rules.
- One alert rule can produce multiple alert history records.
- One user can have multiple refresh-token records.

## 5. Relationship Diagram

```text
users
  |
  +---- organization_members
                 |
                 v
           organizations
                 |
                 v
              projects
                 |
       +---------+----------+
       |         |          |
       v         v          v
    api_keys   services    logs
                            |
                  +---------+---------+
                  |                   |
                  v                   v
          anomaly_predictions     incident_logs
                                      |
                                      v
                                  incidents

projects
   |
   v
alert_rules
   |
   v
alert_history

users
   |
   +---- refresh_tokens
   |
   +---- audit_logs
```

The exact foreign keys and cardinalities will be finalized in the migration implementation.

## 6. Identifier Strategy

UUID identifiers will be used for major domain entities where appropriate.

Benefits:

- Avoid predictable sequential public identifiers.
- Support distributed event references.
- Reduce identifier collisions.
- Simplify integration across services.

Database-generated or application-generated UUID strategies will be selected consistently during implementation.

## 7. Multi-Tenant Data Isolation

LogSage AI must isolate organization data.

Every organization-owned resource must be associated with its owning organization, directly or through a validated project relationship.

Authorization requirements:

1. Authenticate the user.
2. Determine the user's organization membership.
3. Validate the required role and permissions.
4. Verify project ownership within that organization.
5. Execute the authorized database operation.

A user must never access another organization's:

- Projects
- Logs
- API keys
- Incidents
- Alert rules
- Monitoring information

Authorization must be enforced in the backend, not only in frontend routes.

## 8. Log Storage Design

The logs table will store structured application events.

Planned fields:

- id
- organization_id
- project_id
- service_id
- timestamp
- level
- service_name
- environment
- message
- trace_id
- span_id
- request_id
- endpoint
- http_status
- duration_ms
- metadata
- created_at

The metadata field may use PostgreSQL JSONB for structured additional attributes.

Log timestamps will be normalized to UTC.

Large log collections must be retrieved using pagination and bounded queries.

## 9. API Key Storage

API keys must be generated using cryptographically secure random values.

The database must store:

- Key identifier
- Project association
- Secure key hash
- Display prefix
- Creation timestamp
- Expiration metadata where applicable
- Revocation timestamp
- Last-used timestamp

Raw API keys must never be stored or returned from normal database queries.

The raw key will only be displayed once during creation.

## 10. Indexing Strategy

Indexes will be added based on actual query patterns.

Initial candidates include:

- organization_members(user_id, organization_id)
- projects(organization_id)
- api_keys(project_id)
- logs(project_id, timestamp)
- logs(project_id, level, timestamp)
- logs(project_id, service_name, timestamp)
- logs(trace_id)
- logs(request_id)
- incidents(project_id, status)
- incidents(project_id, last_seen_at)
- alert_rules(project_id)

Unique constraints will protect important business rules.

Indexes will be validated using query plans and realistic test workloads before performance claims are made.

## 11. Database Migrations

Flyway will manage database schema changes.

Migration files will follow a versioned naming convention:

```text
V1__create_users_table.sql
V2__create_organizations_table.sql
V3__create_projects_table.sql
V4__create_api_keys_table.sql
V5__create_logs_table.sql
```

Migration requirements:

- Sequential versioning
- Reproducible schema creation
- Foreign-key constraints
- Unique constraints
- Index definitions
- Safe migration practices
- Migration testing

Applied migrations must not be edited casually. Schema corrections should use new migration versions.

## 12. Transactions and Data Integrity

Database operations must use transactions where multiple related changes must succeed or fail together.

Examples:

- Organization and initial owner membership creation
- Project creation
- Incident lifecycle updates
- API key rotation
- Alert history persistence

Foreign keys and unique constraints will enforce data integrity at the database level.

## 13. Log Retention and Scalability

Log retention must be configurable.

Potential future improvements:

- Time-based partitioning
- Retention policies
- Archival strategies
- Efficient bulk ingestion
- Query optimization
- Controlled deletion of expired records

These optimizations will be introduced when supported by measured workload requirements.

## 14. Security Considerations

- Never store plaintext passwords.
- Never store raw ingestion API keys.
- Never commit database credentials.
- Use parameterized database queries.
- Restrict database access to authorized application services.
- Enforce organization isolation.
- Avoid exposing internal database entities through APIs.
- Audit sensitive operations.
- Use environment-based configuration.

## 15. Testing Strategy

Database verification will include:

- Migration execution tests
- Entity relationship tests
- Repository tests
- Unique constraint tests
- Foreign-key integrity tests
- Pagination tests
- Organization isolation tests
- Transaction rollback tests
- Query performance inspection

## 16. Implementation Status

This document defines the planned database design.

The actual schema will be considered implemented only after:

1. Flyway migrations are created.
2. Migrations execute successfully.
3. Entity mappings are verified.
4. Repository tests pass.
5. Organization isolation is tested.
6. Important indexes are verified.

No database feature will be marked complete based only on its documentation.
