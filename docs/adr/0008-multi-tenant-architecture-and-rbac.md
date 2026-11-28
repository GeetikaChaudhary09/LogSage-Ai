
# ADR-0008: Multi-Tenant Architecture and Role-Based Access Control

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI is a multi-tenant observability platform.

Multiple organizations must be able to use the same platform while keeping their projects, logs, services, incidents, alerts and other resources isolated.

Users may belong to one or more organizations, and their permissions may differ between organizations.

The system needs a consistent authorization model that prevents unauthorized access to organization-owned resources.

## Decision

LogSage AI will implement organization-based multi-tenancy with centralized role-based access control (RBAC).

Each organization will own its projects and operational resources.

Users will access organization resources through explicit organization membership.

The backend will enforce organization isolation and role permissions for every protected operation.

## Multi-Tenant Data Ownership

The following ownership hierarchy will be used:

Organization
- Organization Members
- Projects
  - API Keys
  - Services
  - Logs
  - Anomaly Predictions
  - Incidents
  - Alert Rules
  - Alert History
  - Audit Logs

Every organization-owned resource must be associated with its owning organization directly or through a validated parent relationship.

A project must belong to exactly one organization.

A project-scoped API key must belong to the project for which it was generated.

## Organization Membership

A user must have an organization membership before accessing that organization's protected resources.

Membership records will contain:

- Membership ID.
- User ID.
- Organization ID.
- Assigned role.
- Membership status.
- Created timestamp.
- Updated timestamp.

The same user may belong to multiple organizations with different roles.

Membership status must be checked during authorization.

Inactive, suspended or removed memberships must not grant access.

## Role-Based Access Control

LogSage AI will initially support four organization roles.

### OWNER

The organization owner has full control over the organization.

Responsibilities:

- Manage organization settings.
- Manage members and roles.
- Manage projects.
- Create, revoke and rotate project API keys.
- Manage alert rules.
- Access operational data.
- Transfer ownership through an explicitly authorized workflow.
- Delete the organization through a controlled process.

Each organization must have a clearly defined owner.

### ADMIN

Administrators can manage most organization resources.

Responsibilities:

- Manage projects.
- Invite and manage members within permitted limits.
- Configure alert rules.
- Manage API keys where permitted.
- Access logs, incidents and metrics.
- Update operational settings.

Administrators must not perform owner-exclusive operations unless explicitly authorized.

### MEMBER

Members can use assigned organization resources.

Responsibilities:

- View operational dashboards.
- Explore permitted logs.
- Investigate and update incidents according to policy.
- View services and anomaly information.
- Use operational features granted to their role.

Members must not manage organization ownership or privileged membership settings.

### VIEWER

Viewers have read-only access to permitted organization resources.

Responsibilities:

- View dashboards.
- Search and inspect logs.
- View incidents, services and alert information.
- Inspect permitted operational metrics.

Viewers must not create, modify or delete protected resources.

## Permission Enforcement

The backend will enforce authorization through centralized Spring Security authorization logic and service-layer checks.

Frontend route protection and hidden UI controls are usability features only.

They must never replace backend authorization.

Every protected operation must verify:

1. The user is authenticated.
2. The organization exists and is accessible.
3. The user has an active membership.
4. The requested action is permitted for the assigned role.
5. The requested resource belongs to the authorized organization.

## Organization Isolation

The authenticated user's organization context must be resolved and validated by the backend.

The backend must not trust organization identifiers supplied by the client without authorization checks.

All organization-scoped database queries must include the authorized organization context.

Resource access through guessed UUIDs or modified request parameters must not bypass isolation.

Cross-organization access attempts must be rejected without exposing sensitive resource information.

## Project Isolation

Projects must be accessed only through their owning organization.

The backend must verify both organization membership and project ownership.

Project-scoped API keys must not grant access to another project's data.

Ingestion authentication and user-facing management authorization must remain separate security mechanisms.

## Database Design Requirements

The database schema will include:

### Organizations

- Organization ID.
- Organization name.
- Slug or unique identifier.
- Owner user ID where required by the ownership model.
- Created timestamp.
- Updated timestamp.

### Organization Members

- Membership ID.
- Organization ID.
- User ID.
- Role.
- Membership status.
- Created timestamp.
- Updated timestamp.

### Projects

- Project ID.
- Organization ID.
- Project name.
- Environment or configuration metadata.
- Created timestamp.
- Updated timestamp.

Foreign keys and uniqueness constraints must protect organization membership and resource ownership relationships.

Queries and indexes must support efficient organization-scoped access.

## Audit Logging

Important organization-level operations must generate audit events.

Examples include:

- Member invitations.
- Member removal.
- Role changes.
- Project creation and deletion.
- API key creation and revocation.
- Organization setting changes.
- Ownership transfer.

Audit events must identify the actor, action, affected resource and timestamp.

Sensitive credentials and raw tokens must never be included in audit payloads.

## Security Requirements

- Enforce authorization on the backend.
- Validate organization membership on protected operations.
- Apply organization filters to resource queries.
- Prevent insecure direct object reference vulnerabilities.
- Enforce project ownership.
- Protect privileged owner operations.
- Record important administrative actions.
- Avoid leaking information through authorization errors.
- Test cross-organization access attempts.
- Keep ingestion credentials separate from user authorization.

## Testing Requirements

Planned authorization tests:

- Organization creation and ownership.
- Membership creation and removal.
- Multiple organizations per user.
- Different roles across organizations.
- OWNER permission enforcement.
- ADMIN permission enforcement.
- MEMBER permission enforcement.
- VIEWER read-only enforcement.
- Inactive membership rejection.
- Cross-organization project access rejection.
- Cross-organization log access rejection.
- API key project isolation.
- Unauthorized resource modification rejection.
- Ownership transfer authorization.
- Audit event creation.

## Alternatives Considered

### Single Shared Organization

Rejected because it does not provide independent tenant boundaries.

### Frontend-Only Role Checks

Rejected because frontend restrictions can be bypassed.

### Separate Database for Every Organization

Not selected for the initial architecture because it increases infrastructure and operational complexity.

A shared PostgreSQL database with strict organization-level isolation is selected for the initial platform.

### Unrestricted Resource Access by UUID

Rejected because knowing a resource identifier must never grant access to that resource.

## Consequences

### Positive

- Clear organization ownership.
- Independent organization memberships.
- Consistent role-based permissions.
- Better tenant isolation.
- Centralized authorization.
- Easier auditing and administration.
- Scalable initial database architecture.

### Negative

- Every organization-scoped query requires careful authorization.
- Permission management adds implementation complexity.
- Cross-tenant isolation requires extensive automated testing.
- Shared database isolation depends on correct backend enforcement.

## Implementation Notes

- Use Spring Security.
- Centralize authorization rules.
- Validate organization membership in service-layer operations.
- Include organization context in repository queries.
- Use foreign keys and appropriate database indexes.
- Add organization-isolation integration tests.
- Do not rely on frontend-only permission checks.
- Keep API key ingestion authorization separate from user RBAC.

## Review Conditions

Revisit this decision if:

- Tenant isolation requirements change.
- Dedicated database or schema-per-tenant architecture becomes necessary.
- Organization roles become more granular.
- External identity or enterprise access requirements are introduced.
- Security testing identifies weaknesses in tenant isolation.
