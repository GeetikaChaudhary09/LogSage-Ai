
# ADR-0004: Redis for Caching and Rate Limiting

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI provides dashboard metrics, log search, project information and other frequently accessed data.

Repeatedly calculating expensive dashboard summaries and processing every rate-limit check through PostgreSQL could increase database load.

The platform needs a fast supporting data store for short-lived data and rate-limiting operations.

## Decision

Redis will be used as the in-memory supporting data store for caching, rate limiting and selected temporary processing state.

Redis will not be the primary persistent database.

PostgreSQL remains the authoritative source of truth for all critical application records.

## Responsibilities

### 1. Dashboard Caching

Redis may cache frequently requested dashboard summaries, including:

- Log volume summaries.
- Error count summaries.
- Error rate summaries.
- Service-level summaries.
- Time-bucketed metrics.

Cache entries must have appropriate expiration policies.

### 2. Rate Limiting

Redis may support rate limiting for:

- Log ingestion endpoints.
- Authentication endpoints.
- Sensitive API operations.
- Other configurable request limits.

Rate-limit keys must be scoped appropriately to users, projects, API keys or clients.

### 3. Temporary Processing State

Redis may store short-lived processing information where justified.

Temporary state must not become the only copy of critical business information.

### 4. Cache Expiration

Every cache entry must have an explicit and appropriate TTL.

Cache lifetime should depend on the type of information and its freshness requirements.

## Cache Consistency

PostgreSQL remains authoritative.

When underlying persistent data changes:

- Relevant cache entries should be invalidated or refreshed.
- Stale cache values must not be treated as authoritative.
- Cache failures must not silently produce incorrect business records.

The implementation will use an appropriate cache invalidation strategy.

## Failure Handling

If Redis becomes unavailable:

- Critical database persistence must continue where possible.
- Authentication and authorization must remain secure.
- Rate limiting must follow a documented safe fallback policy.
- Cached dashboard information may become temporarily unavailable.
- The application must not treat a cache miss as a database record.
- Redis recovery must not require rebuilding critical application data.

Security-sensitive endpoints must not become unrestricted merely because Redis is unavailable.

## Data Management

- Define TTL for temporary keys.
- Avoid storing unnecessary sensitive information.
- Use appropriate key naming conventions.
- Prevent unbounded cache growth.
- Monitor memory usage.
- Document eviction policy.
- Protect Redis network access and credentials.

## Alternatives Considered

### PostgreSQL for All Caching

Not selected because frequent cache operations and rate-limit counters are better handled by a dedicated in-memory supporting service.

### Redis as Primary Database

Rejected because PostgreSQL is the authoritative persistent store.

### In-Memory Application Cache Only

Not selected as the primary shared cache because multiple backend instances would not share the same cache state.

## Consequences

### Positive

- Faster repeated dashboard reads.
- Reduced unnecessary database queries.
- Shared rate-limit state across backend instances.
- TTL-based expiration.
- Independent cache scaling.

### Negative

- Additional infrastructure to operate.
- Cache invalidation complexity.
- Memory usage requires monitoring.
- Redis availability must be handled.
- Cache consistency requires careful implementation.

## Implementation Notes

- Integrate Redis with the Java backend.
- Define cache key conventions.
- Configure TTL policies.
- Implement rate limiting.
- Define safe Redis outage behaviour.
- Avoid storing critical persistent records only in Redis.
- Add tests for cache expiry, invalidation and Redis failures.

## Review Conditions

Revisit this decision if:

- Cache workload changes significantly.
- Rate-limiting requirements change.
- Memory usage becomes a measurable concern.
- A different caching strategy becomes technically justified.
