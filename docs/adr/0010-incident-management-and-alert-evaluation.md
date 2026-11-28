
# ADR-0010: Incident Management and Alert Evaluation Architecture

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI must convert operational signals from application logs and anomaly detection into actionable incidents and alerts.

Simply detecting an unusual log pattern is not sufficient. Developers need a centralized workflow to investigate issues, understand supporting evidence, track incident status and receive notifications when important conditions occur.

The platform must support threshold-based alert rules, anomaly-driven incident creation, incident lifecycle management, cooldowns and duplicate prevention.

The incident and alert engines must operate independently of the ingestion API and must not block log collection.

## Decision

LogSage AI will implement separate but integrated incident management and alert evaluation components within the Java Spring Boot backend.

These components will consume persisted log events, anomaly predictions and operational metrics.

The system will:

1. Evaluate configured alert rules.
2. Identify actionable anomalies and threshold violations.
3. Correlate related evidence.
4. Create or update incidents.
5. Maintain incident lifecycle and history.
6. Generate alert events.
7. Deliver in-app notifications.
8. Record relevant audit events.

PostgreSQL will remain the source of truth for incidents, alert rules, alert history and notifications.

RabbitMQ will support asynchronous event-driven processing.

## Incident Lifecycle

Incidents will initially support the following statuses:

### OPEN

A new incident has been detected and requires attention.

### ACKNOWLEDGED

A user has acknowledged the incident and confirmed awareness of the issue.

### INVESTIGATING

The incident is actively being investigated.

### RESOLVED

The issue has been addressed and the incident is considered resolved.

### IGNORED

The incident has been intentionally dismissed according to the applicable permissions and workflow.

Status transitions must be validated by the backend.

The system must record who performed a transition and when it occurred.

## Incident Creation

Incidents may be created from:

- Significant anomaly predictions.
- Repeated application errors.
- Configured alert rule violations.
- Other validated operational signals.

Incident creation must be based on actual persisted telemetry and evidence.

The system must not create fabricated incidents or use demonstration data as real production activity.

## Incident Correlation and Deduplication

Repeated events related to the same underlying issue should not create unlimited duplicate incidents.

The incident engine will use a configurable correlation strategy based on available evidence, such as:

- Organization.
- Project.
- Service.
- Environment.
- Error category or fingerprint.
- Relevant time window.
- Related trace or request identifiers where available.

The exact fingerprint and correlation algorithm will be finalized during implementation and testing.

When new evidence matches an existing active incident, the system should update that incident instead of creating an unrelated duplicate.

Resolved incidents may be reopened when new matching evidence satisfies the configured reopening policy.

## Incident Evidence

Each incident must retain references to its supporting evidence.

Evidence may include:

- Related log records.
- Anomaly predictions.
- Triggered alert rules.
- Affected service.
- Environment.
- Time range.
- Relevant trace and request identifiers.
- Operational metrics.

Evidence must be traceable to actual stored records.

The platform must distinguish observed facts from generated explanations or hypotheses.

## Incident Timeline

Each incident must maintain a timeline of important events, including:

- Incident creation.
- Status changes.
- Assignment changes.
- New evidence.
- User comments where supported.
- Resolution.
- Reopening.
- Relevant automated actions.

Timeline entries must include timestamps and actor information where applicable.

Timestamps must use UTC consistently.

## Incident Assignment

The system should support assigning incidents to authorized organization members.

Assignment operations must:

- Validate organization membership.
- Enforce role permissions.
- Record the assigned user.
- Maintain assignment history.
- Generate relevant notifications where configured.

Users must not assign incidents to unauthorized members of another organization.

## Alert Rule Management

Authorized users will be able to create and manage alert rules.

Initial alert rule categories will include:

### Error Count

Trigger when the number of error logs exceeds a configured threshold within an evaluation window.

### Error Rate

Trigger when the error ratio exceeds a configured percentage.

### Log Volume

Trigger when the total number of logs exceeds a configured threshold.

### Service-Specific Errors

Trigger when a selected service produces an unusual or excessive number of errors.

### Anomaly Score

Trigger when a valid anomaly prediction meets the configured score or decision criteria.

Alert rules must support project and service scope where applicable.

## Alert Rule Configuration

Each rule should support:

- Rule identifier.
- Organization identifier.
- Project identifier.
- Rule name.
- Description.
- Rule type.
- Enabled or disabled status.
- Threshold or condition.
- Evaluation window.
- Severity.
- Optional service scope.
- Cooldown duration.
- Created and updated timestamps.

Configuration must be validated before saving.

Only authorized users may create, update, enable, disable or delete alert rules.

## Alert Evaluation Engine

The alert evaluation component will evaluate configured rules against actual operational data.

Evaluation must:

1. Load enabled rules.
2. Identify the relevant project and service scope.
3. Retrieve the required time-window metrics.
4. Evaluate the configured condition.
5. Determine whether the condition is met.
6. Apply cooldown and deduplication rules.
7. Generate an alert event when appropriate.
8. Update alert history.

Evaluation should be asynchronous and must not block log ingestion.

## Evaluation Windows

Alert rules must support configurable evaluation windows.

Examples include:

- Error count over the last five minutes.
- Error rate over the last ten minutes.
- Log volume over a defined interval.
- Service-specific error count over a configured period.

Window boundaries, timezone handling and aggregation behaviour must be consistent.

The system must distinguish unavailable telemetry from a genuine zero value.

Missing data must not automatically be interpreted as a healthy condition.

## Cooldown and Deduplication

Repeated evaluation of the same condition must not generate uncontrolled alert notifications.

Each rule must support a configurable cooldown period.

The alert engine must track previous firing events and prevent duplicate notifications during the cooldown window.

When a condition recovers, the system should record the recovery according to the alert lifecycle policy.

A new notification may be generated when the condition fires again after the applicable cooldown and recovery rules.

## Alert Severity

Alert severity must be configurable.

The initial severity levels may include:

- INFO
- WARNING
- CRITICAL

Severity must be preserved in alert history and notification records.

Severity must not be inferred from arbitrary client-provided text without validation.

## Anomaly-Driven Incidents

Anomaly predictions from the Python ML service may be used as inputs to incident correlation and alert evaluation.

The incident engine must distinguish:

- A confirmed model prediction.
- An inference failure.
- Missing prediction data.
- A genuine anomaly signal.

An ML service outage or inference failure must never automatically be treated as an anomaly.

Anomaly-based incident creation must follow configurable confidence or decision policies and use supporting telemetry evidence.

The ML pipeline and anomaly detection strategy are defined in ADR-0005.

## In-App Notifications

The initial notification channel will be in-app notifications.

Notifications may be generated for:

- New incidents.
- Critical alerts.
- Incident assignments.
- Incident status changes.
- Incident resolution.
- Reopened incidents.

Notification records should contain:

- Notification ID.
- Organization ID.
- Recipient user ID.
- Event type.
- Related incident or alert reference.
- Message or structured content.
- Read status.
- Created timestamp.

Notification access must be restricted to the authorized recipient and organization.

Email, SMS and external messaging integrations may be considered later.

## Asynchronous Event Processing

RabbitMQ will support event-driven workflows for incident and alert processing.

Relevant event categories may include:

- Incident creation events.
- Incident update events.
- Alert triggered events.
- Alert recovered events.
- Notification delivery events.

Consumers must be idempotent.

Retries and dead-letter handling must follow the messaging reliability strategy defined in ADR-0003.

The incident engine must not acknowledge required events before the corresponding persistence operation succeeds.

## Database Requirements

The initial database design will include:

### Incidents

- Incident ID.
- Organization ID.
- Project ID.
- Title.
- Description.
- Status.
- Severity.
- Affected service.
- Assigned user ID.
- First detected timestamp.
- Last detected timestamp.
- Resolved timestamp.
- Created timestamp.
- Updated timestamp.

### Incident Logs

- Relationship ID.
- Incident ID.
- Log ID.
- Association timestamp.

### Alert Rules

- Alert rule ID.
- Organization ID.
- Project ID.
- Rule name.
- Rule type.
- Configuration.
- Enabled status.
- Cooldown.
- Created and updated timestamps.

### Alert History

- Alert event ID.
- Alert rule ID.
- Organization ID.
- Project ID.
- Triggered timestamp.
- Evaluation result.
- Severity.
- Recovery information where applicable.

### Notifications

- Notification ID.
- Organization ID.
- Recipient user ID.
- Event type.
- Related resource reference.
- Read status.
- Created timestamp.

Foreign keys, indexes and constraints must protect resource relationships and support efficient queries.

## Security and Authorization

- Enforce organization isolation.
- Validate project ownership.
- Apply RBAC to incident and alert operations.
- Restrict notification access to authorized recipients.
- Validate incident status transitions.
- Protect administrative alert rule operations.
- Record important user actions.
- Avoid exposing sensitive log content in notifications.
- Never trust client-provided ownership information.

## Observability

The incident and alert components must expose operational measurements such as:

- Incidents created.
- Incidents resolved.
- Incidents reopened.
- Active incident count.
- Alert rules evaluated.
- Alert rules triggered.
- Alert evaluation failures.
- Notifications generated.
- Duplicate events prevented.
- Processing latency.

These measurements must be derived from actual system activity.

## Testing Requirements

Planned incident and alert tests:

- Incident creation from valid evidence.
- Incident correlation.
- Duplicate incident prevention.
- Incident reopening.
- Valid status transitions.
- Invalid status transition rejection.
- Incident assignment authorization.
- Organization isolation.
- Alert rule creation and validation.
- Error count threshold evaluation.
- Error rate evaluation.
- Log volume evaluation.
- Service-specific alert evaluation.
- Anomaly-based alert evaluation.
- Evaluation window boundaries.
- Cooldown enforcement.
- Duplicate alert prevention.
- Alert recovery handling.
- Notification generation.
- Notification recipient authorization.
- RabbitMQ retry and failure handling.
- ML service failure without false anomaly creation.

## Alternatives Considered

### Synchronous Incident Evaluation During Ingestion

Rejected because it would increase ingestion latency and tightly couple log collection to downstream analysis.

### Independent Incident and Alert Microservices

Not selected for the initial architecture because separate deployments would add operational complexity.

Both components will initially remain modules inside the Java modular monolith.

### Notification-Only Monitoring

Rejected because notifications alone do not provide a structured investigation, assignment and resolution workflow.

### Database-Driven Periodic Evaluation Only

Not selected as the sole mechanism because asynchronous events can support more responsive processing.

Scheduled evaluation may still be used for time-window rules and recovery checks.

## Consequences

### Positive

- Structured incident investigation.
- Evidence-based incident creation.
- Reduced duplicate incidents.
- Configurable alert conditions.
- Controlled notification behaviour.
- Clear incident lifecycle.
- Independent asynchronous processing.
- Better operational visibility.

### Negative

- Correlation and deduplication require careful design.
- Evaluation windows introduce processing complexity.
- Cooldown and recovery policies require configuration.
- Event-driven workflows require idempotency and retry handling.
- Incident and alert components require comprehensive integration testing.

## Implementation Notes

- Implement incident and alert modules in the Java backend.
- Use PostgreSQL as the source of truth.
- Use RabbitMQ for asynchronous event handling.
- Derive alert conditions from actual telemetry.
- Maintain incident evidence and timelines.
- Implement cooldown and deduplication.
- Enforce organization and role permissions.
- Add integration tests for failure and recovery scenarios.
- Keep ML inference failures separate from anomaly outcomes.

## Review Conditions

Revisit this decision if:

- Alert volume requires a dedicated evaluation engine.
- External notification channels are introduced.
- Incident correlation requirements change.
- New alert types are added.
- Event processing reliability requirements change.
- Production testing identifies weaknesses in incident lifecycle management.
