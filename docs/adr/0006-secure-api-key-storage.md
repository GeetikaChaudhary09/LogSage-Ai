
# ADR-0006: Secure API Key Storage and One-Time Reveal

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI allows external applications to send application logs through a project-specific ingestion API.

Each project requires credentials to authenticate incoming ingestion requests.

API keys are sensitive credentials. If raw keys are stored in the database or exposed through application logs and API responses, a database leak or accidental disclosure could allow unauthorized log submission.

A secure key generation, storage, verification and revocation strategy is required.

## Decision

LogSage AI will generate cryptographically secure, project-scoped ingestion API keys.

The raw API key will be displayed only once when it is created.

Only a secure hash of the key will be stored in PostgreSQL.

The backend will verify incoming API keys against their stored hashes.

## Key Generation

API keys must:

- Use a cryptographically secure random generator.
- Contain sufficient random entropy.
- Be unique.
- Be associated with a specific project.
- Have a recognizable non-secret prefix where useful for identification.
- Never be derived from predictable values.

The generation strategy must be documented and tested.

## One-Time Reveal

When a user creates an API key:

1. The backend generates the raw key.
2. The backend calculates its secure hash.
3. Only the hash and non-sensitive metadata are persisted.
4. The raw key is returned in the creation response.
5. The frontend displays the raw key once.
6. The user is instructed to copy and store it securely.

The raw key must not be retrievable through later list or detail API requests.

If the user loses the raw key, they must create a replacement or rotate the existing credential.

## Secure Storage

PostgreSQL will store:

- API key identifier.
- Project identifier.
- Secure key hash.
- Non-secret display prefix.
- Creation timestamp.
- Revocation timestamp or status.
- Last-used timestamp where available.
- Relevant audit metadata.

The raw key must never be stored in plaintext.

A suitable one-way password-style hashing approach or a keyed cryptographic hash with a separately protected server-side secret must be selected during implementation.

The selected verification approach must account for API key entropy and secure comparison.

## Authentication Header

External applications will send the API key through:

```http
X-LogSage-API-Key: <raw-api-key>
```

The backend will:

1. Read the API key header.
2. Validate the key format.
3. Identify the appropriate key record.
4. Verify the supplied key against its stored hash.
5. Check revocation and validity.
6. Resolve the associated project.
7. Authorize ingestion only for that project.

Client-provided organization or project identifiers must never override the ownership determined by the authenticated API key.

## Revocation

Authorized users must be able to revoke an API key.

After revocation:

- The key must no longer authenticate ingestion requests.
- The key must not be reactivated without an explicit new credential workflow.
- Revocation must be recorded in audit history.
- Relevant key metadata may remain available for authorized inspection.

## Rotation

Key rotation must allow a project to replace a credential safely.

The workflow should:

1. Generate a replacement key.
2. Display the new raw key once.
3. Allow the user to update the external application.
4. Revoke the old key according to the defined rotation policy.
5. Record the operation.

Any overlap period must be explicit and controlled.

## Authorization

API key management operations require authenticated user access and appropriate organization-level permissions.

A user must not create, inspect, revoke or rotate keys belonging to an unauthorized organization or project.

Ingestion keys must not grant access to user-facing management APIs.

## Logging and Exposure Prevention

Never expose raw API keys in:

- Application logs.
- Error messages.
- Database query logs.
- Analytics events.
- Audit event payloads.
- API key list responses.
- Monitoring dashboards.
- Frontend URLs.

Sensitive headers must be redacted where request logging is enabled.

## Testing Requirements

Planned security tests:

- Secure key generation.
- Key uniqueness.
- Hash-only persistence.
- Successful key verification.
- Invalid key rejection.
- Revoked key rejection.
- Project association enforcement.
- Organization isolation.
- One-time reveal behaviour.
- Rotation behaviour.
- Secret redaction.

## Alternatives Considered

### Plaintext API Key Storage

Rejected because database exposure would reveal active credentials directly.

### Reversible Encryption

Not selected as the default approach because ingestion authentication only requires verification, not recovery of the original key.

### Shared Organization-Wide Key

Rejected because project-scoped credentials provide better isolation and independent revocation.

## Consequences

### Positive

- Reduced impact of database exposure.
- Project-level credential isolation.
- Controlled key lifecycle.
- Safer credential rotation.
- Better auditability.

### Negative

- Raw keys cannot be recovered after creation.
- Verification requires secure hashing logic.
- Users must manage their credentials carefully.
- Rotation workflows require clear frontend guidance.

## Implementation Notes

- Use a cryptographically secure random generator.
- Store only secure hashes.
- Show raw credentials once.
- Use constant-time comparison where appropriate.
- Enforce project and organization ownership.
- Redact credentials from logs.
- Add security-focused automated tests.

## Review Conditions

Revisit this decision if:

- API key authentication requirements change.
- Credential rotation policy changes.
- Security review identifies a better verification strategy.
- A new credential management mechanism is introduced.
