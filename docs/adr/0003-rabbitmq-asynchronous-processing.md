
# ADR-0003: RabbitMQ for Asynchronous Processing

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI receives application logs from external applications.

Each incoming log may require several operations:

- Authentication and validation.
- Payload normalization.
- Persistent database storage.
- Feature extraction.
- Machine learning analysis.
- Incident evaluation.
- Alert and notification processing.

Performing every operation synchronously inside the ingestion request could increase response time and make ingestion dependent on downstream service availability.

The platform therefore needs an asynchronous messaging system.

## Decision

RabbitMQ will be used as the message broker for asynchronous processing in LogSage AI.

The Java Spring Boot backend will publish processing events, and background consumers will process them independently.

Log ingestion must not wait for machine learning inference or notification delivery.

## Responsibilities

RabbitMQ will support asynchronous workflows for:

- Log ingestion processing.
- Log processing.
- Anomaly analysis.
- Incident events.
- Notification delivery.

## Planned Queues

| Queue | Responsibility |
|---|---|
| logs.ingestion | Incoming validated log events |
| logs.processing | Background log persistence and processing |
| anomaly.analysis | ML feature analysis and inference |
| incidents.events | Incident creation and lifecycle events |
| notifications.delivery | Notification processing |

Queue and exchange topology will be finalized during implementation.

## Message Processing Workflow

1. External application sends logs to the ingestion API.
2. Backend authenticates the project API key.
3. Backend validates and normalizes the payload.
4. A processing message is published.
5. RabbitMQ routes the message to the appropriate queue.
6. A background consumer receives the message.
7. The consumer performs the required processing.
8. Required database persistence is completed.
9. The message is acknowledged after successful processing.

The exact persistence and publishing sequence must be designed to avoid losing accepted log events.

## Reliability Requirements

### Message Acknowledgements

Use manual acknowledgements where required.

A message must not be acknowledged before its required processing and persistence steps succeed.

### Retry Policy

Implement bounded retries for recoverable failures.

Retries should use an appropriate delay or backoff strategy to avoid repeatedly overwhelming an unavailable dependency.

### Dead Letter Queue

Messages that cannot be processed after the configured retry policy should be routed to a Dead Letter Queue.

Dead-letter messages must be inspectable and recoverable through a controlled process.

### Idempotency

Consumers must handle duplicate message delivery safely.

A repeated message must not create duplicate persistent records or repeated business actions where idempotency is required.

### Durable Messaging

Use durable queues and appropriate persistent message delivery settings for important processing workflows.

### Correlation IDs

Use correlation identifiers to trace related events across ingestion, processing, anomaly analysis and incident workflows.

## Failure Handling

### Database Unavailability

If required persistence fails:

- Do not acknowledge the message.
- Apply the configured retry policy.
- Preserve the message for recovery.
- Avoid losing accepted events.

### ML Service Unavailability

If ML inference is unavailable:

- Log ingestion must continue.
- Events must not be silently discarded.
- Retry inference safely or route exhausted messages through the configured failure workflow.
- Never treat inference failure as an anomaly.

### Notification Failure

Notification failures must not roll back already persisted incidents.

Notification delivery should be retried independently.

## Message Design

Messages should contain only the information required by their consumer.

Message schemas should be versioned and documented.

Include appropriate:

- Event identifier.
- Event type.
- Correlation ID.
- Creation timestamp.
- Schema version.
- Required payload.

Avoid placing secrets or unnecessary sensitive information in messages.

## Alternatives Considered

### Synchronous Processing Only

Rejected because it couples ingestion response time to database, ML and notification operations.

### Redis Pub/Sub

Not selected as the primary message-processing mechanism because the platform requires durable queueing, acknowledgement, retry and dead-letter handling.

### Direct Database Polling

Rejected as the primary event-processing approach because it would introduce unnecessary polling and coordination complexity.

## Consequences

### Positive

- Faster ingestion responses.
- Independent background processing.
- Better failure isolation.
- Controlled retries.
- Dead-letter recovery.
- Easier scaling of consumers.
- Clear event-driven boundaries.

### Negative

- Additional infrastructure to operate.
- Eventual consistency between ingestion and dashboard visibility.
- Duplicate delivery must be handled.
- Message ordering and retry behaviour require careful design.
- Queue monitoring and recovery procedures are necessary.

## Implementation Notes

- Use Spring AMQP for Java integration.
- Configure durable queues and appropriate exchanges.
- Implement manual acknowledgement where required.
- Add bounded retry policies.
- Configure dead-letter routing.
- Implement idempotent consumers.
- Add message correlation IDs.
- Monitor queue depth and consumer health.
- Test broker and consumer failure scenarios.

## Review Conditions

Revisit this decision if:

- Message throughput changes significantly.
- Queue processing becomes a measurable bottleneck.
- Operational requirements change.
- Another messaging technology provides a clearly justified advantage.
