
# ADR-0009: Log Ingestion and Processing Architecture

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI collects application logs from external applications for centralized observability, anomaly detection and incident management.

External applications must be able to submit structured logs securely and reliably.

The ingestion system must support individual and batch requests, validate incoming data, authenticate project credentials, prevent unauthorized access and process accepted logs asynchronously.

High ingestion traffic must not block the main application API while logs are being persisted and analyzed.

The architecture must also handle duplicate requests, temporary failures, invalid payloads and downstream service outages.

## Decision

LogSage AI will implement a project-scoped HTTP log ingestion API backed by RabbitMQ asynchronous processing.

The primary endpoint will be:

`POST /api/v1/ingest/logs`

The ingestion workflow will:

1. Receive the HTTP request.
2. Authenticate the project API key.
3. Resolve the authorized project and organization.
4. Validate and normalize the payload.
5. Apply request and payload limits.
6. Publish accepted log events to RabbitMQ.
7. Return an appropriate ingestion response.
8. Process queued events asynchronously.
9. Persist validated logs in PostgreSQL.
10. Trigger downstream metrics, anomaly and incident workflows.

## Authentication

External applications must authenticate using the project-specific API key.

The key will be provided through:

`X-LogSage-API-Key`

The backend must validate the key and resolve its associated project.

The client must not be allowed to override the organization or project associated with the authenticated key.

Invalid, revoked or unauthorized keys must be rejected.

API key generation, storage and lifecycle management are defined in ADR-0006.

## Endpoint Design

### Single Log Ingestion

The endpoint must accept an individual structured log event.

### Batch Log Ingestion

The endpoint must also support multiple log events in one request.

Batch processing must validate individual entries and apply documented rules for partially invalid batches.

The maximum batch size and request body size must be configurable and enforced.

## Log Event Schema

The initial structured log event will support:

- timestamp
- level
- serviceName
- environment
- message
- traceId (optional)
- spanId (optional)
- requestId (optional)
- endpoint (optional)
- httpStatus (optional)
- durationMs (optional)
- metadata (optional)

The backend must validate required fields, field types, supported severity values and configured size limits.

Timestamps must be normalized to UTC.

Missing optional values must be handled consistently.

Unknown or unsupported fields must follow an explicitly documented validation policy.

## Log Levels

The initial supported severity levels will be:

- TRACE
- DEBUG
- INFO
- WARN
- ERROR
- FATAL

Severity values must be normalized consistently before persistence and filtering.

## Validation and Normalization

The ingestion API must validate:

- API key authentication.
- Required log fields.
- Timestamp format.
- Severity level.
- Service name.
- Environment.
- Message length.
- Optional identifiers.
- Numeric duration and HTTP status values.
- Metadata size and structure.
- Batch size.
- Request payload size.

Invalid requests must return structured validation errors without exposing internal implementation details.

The ingestion layer must not trust client-provided ownership information.

## Asynchronous Processing

RabbitMQ will decouple HTTP ingestion from database persistence and downstream processing.

The initial ingestion queue will be:

`logs.ingestion`

The processing workflow will:

1. Receive an accepted log event.
2. Validate its internal event envelope.
3. Apply idempotency checks.
4. Persist the log in PostgreSQL.
5. Update or maintain relevant service information.
6. Publish downstream events where required.
7. Acknowledge the message only after the required persistence operation succeeds.

The HTTP ingestion path must not wait for ML inference or incident analysis to finish.

## Queue Reliability

The messaging workflow must support:

- Durable queues and persistent messages.
- Manual message acknowledgement.
- Controlled retries.
- Dead Letter Queues (DLQs).
- Idempotent processing.
- Correlation identifiers.
- Failure monitoring.
- Recovery from temporary downstream failures.

A message must not be acknowledged before its required processing and persistence steps succeed.

Repeated failures must be handled through a bounded retry policy and then routed to an appropriate DLQ.

The retry strategy must prevent uncontrolled infinite processing loops.

RabbitMQ architecture is further defined in ADR-0003.

## Idempotency and Duplicate Handling

Log ingestion must account for client retries, network timeouts and repeated message delivery.

The system must define an idempotency strategy for ingestion requests or individual log events.

Where an idempotency identifier is provided, it must be scoped appropriately to the authenticated project.

Duplicate detection must not accidentally merge unrelated log events.

The selected database constraints, identifier strategy and retention policy must be documented during implementation.

Message processing must remain safe under at-least-once delivery.

Exactly-once delivery must not be assumed.

## Rate Limiting and Request Limits

The ingestion endpoint must enforce configurable limits for:

- Requests per project.
- Requests per time window.
- Batch size.
- Request body size.
- Individual message size.
- Metadata size.

Redis may be used for distributed rate limiting and temporary counters.

Rate limits must be applied after resolving the authenticated project identity.

Excessive requests must receive an appropriate rate-limit response.

Rate-limiting design is defined in ADR-0004.

## Response Semantics

The API must clearly distinguish between:

- Invalid or unauthorized requests.
- Requests rejected due to configured limits.
- Requests successfully accepted for asynchronous processing.
- Temporary service or queue failures.

A successful ingestion response must indicate acceptance for processing, not guarantee that every downstream operation has completed.

The API response contract and error format will be documented in the API specification.

## PostgreSQL Persistence

PostgreSQL will remain the source of truth for accepted and persisted log records.

Log records must preserve:

- Organization ownership.
- Project ownership.
- Service association where available.
- Original event timestamp normalized to UTC.
- Severity.
- Message.
- Trace and request identifiers where supplied.
- Relevant structured metadata.
- Ingestion and persistence timestamps where required.

Indexes must support common time-range, project, severity and service queries.

Database persistence and ownership rules are defined in ADR-0002 and the database design document.

## Downstream Processing

After successful persistence, the system may publish events for:

- Metrics aggregation.
- Anomaly analysis.
- Incident correlation.
- Alert evaluation.
- Operational notifications.

Downstream consumers must process events independently and safely.

An ML service outage must not block ingestion or cause accepted logs to be discarded.

An inference failure must be distinguishable from a genuine anomaly.

## Error Handling

The system must handle:

- Invalid API keys.
- Malformed JSON.
- Missing required fields.
- Unsupported severity values.
- Oversized payloads.
- Rate-limit violations.
- RabbitMQ unavailability.
- PostgreSQL failures.
- Duplicate events.
- Consumer processing failures.
- ML service unavailability.

The system must not report a request as successfully accepted when the required queue publication has failed.

Where queue publication is required for acceptance, publisher confirms and appropriate failure handling must be used.

## Observability

The ingestion pipeline must expose operational measurements such as:

- Requests received.
- Accepted log events.
- Rejected requests.
- Invalid log events.
- Rate-limit rejections.
- Queue publishing failures.
- Queue depth.
- Processing latency.
- Persistence failures.
- Retry counts.
- DLQ message counts.

Metrics must be derived from actual system activity.

Raw API keys and sensitive log content must not be exposed in monitoring labels or logs.

## Security Requirements

- Require project API key authentication.
- Enforce project ownership.
- Validate all incoming payloads.
- Apply request and batch limits.
- Redact sensitive headers.
- Avoid logging raw credentials.
- Restrict metadata and message sizes.
- Protect against unauthorized resource injection.
- Use secure transport in production.
- Apply organization isolation throughout processing.

## Testing Requirements

Planned ingestion tests:

- Successful single-log ingestion.
- Successful batch ingestion.
- Invalid API key rejection.
- Revoked API key rejection.
- Missing required fields.
- Invalid severity rejection.
- Invalid timestamp handling.
- Oversized request rejection.
- Batch size enforcement.
- Rate-limit enforcement.
- Project ownership enforcement.
- Duplicate event handling.
- RabbitMQ publication failure.
- Consumer retry behaviour.
- Dead Letter Queue routing.
- Database persistence.
- Message acknowledgement after successful persistence.
- Organization isolation.
- Downstream processing failure.
- ML service outage without ingestion interruption.

## Alternatives Considered

### Direct Synchronous Database Persistence

Not selected as the primary processing architecture because database operations and downstream processing would increase ingestion latency and couple the ingestion API to processing availability.

### Synchronous ML Analysis During Ingestion

Rejected because ML inference must not block log collection.

### In-Memory Queue

Rejected because it does not provide the durability and recovery capabilities required for accepted ingestion events.

### RabbitMQ Asynchronous Processing

Selected because it supports durable messaging, acknowledgements, retries, routing and dead-letter handling.

## Consequences

### Positive

- Decoupled ingestion and processing.
- Better handling of traffic spikes.
- Independent downstream consumers.
- Controlled retry and failure handling.
- Better operational monitoring.
- Reduced ingestion latency.
- Improved resilience to ML service outages.

### Negative

- More infrastructure and operational complexity.
- Eventual consistency between ingestion acceptance and database visibility.
- Duplicate delivery must be handled.
- Queue and consumer failures require monitoring.
- Message ordering must not be assumed globally.

## Implementation Notes

- Use Spring Boot REST controllers for ingestion.
- Authenticate with project-scoped API keys.
- Validate payloads before publishing.
- Use RabbitMQ durable queues and persistent messages.
- Enable publisher confirms.
- Use manual acknowledgements.
- Implement bounded retries and DLQs.
- Make consumers idempotent.
- Persist accepted events in PostgreSQL.
- Track correlation identifiers.
- Add integration and failure-recovery tests.

## Review Conditions

Revisit this decision if:

- Ingestion volume requires a different messaging architecture.
- Delivery and ordering requirements change.
- A different event streaming platform becomes necessary.
- Payload or batch requirements change.
- Reliability testing identifies weaknesses in the ingestion workflow.
