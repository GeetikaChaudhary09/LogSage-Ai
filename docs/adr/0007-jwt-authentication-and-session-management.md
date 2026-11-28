
# ADR-0007: JWT Authentication and Session Management

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI is a multi-tenant observability platform where users can create organizations, manage projects, configure API keys, explore logs, investigate incidents and manage alert rules.

The platform requires a secure authentication and authorization mechanism.

Authentication must identify users, while authorization must ensure that users can only access resources permitted by their organization membership and assigned role.

The system also needs secure session management, token expiration, refresh, rotation and logout.

## Decision

LogSage AI will use Spring Security with JSON Web Tokens (JWT) for user authentication.

The authentication system will use:

- Short-lived JWT access tokens.
- Refresh tokens for session renewal.
- Secure password hashing.
- Refresh token rotation.
- Server-side refresh token revocation.
- Role-based access control.
- Organization-level resource authorization.

JWT access tokens will be used to authenticate requests to protected backend APIs.

## User Registration

The registration workflow will:

1. Accept the user's required registration details.
2. Validate the request.
3. Check whether the email address is already registered.
4. Hash the password using BCrypt or another approved password-hashing algorithm.
5. Store the user record in PostgreSQL.
6. Create the initial authenticated session according to the registration flow.

Passwords must never be stored in plaintext.

## User Login

The login workflow will:

1. Accept email and password.
2. Locate the user account.
3. Verify the supplied password against the stored password hash.
4. Reject invalid credentials with a generic authentication error.
5. Generate an access token and refresh token.
6. Establish the authenticated session.

Authentication errors must not reveal whether a particular email address exists.

## Access Tokens

Access tokens will:

- Be signed using a securely managed signing key.
- Have a short configurable expiration period.
- Identify the authenticated user.
- Include necessary claims such as subject and token identifier.
- Include only the minimum authorization information required.
- Be validated by Spring Security on protected requests.

Sensitive information, passwords and raw refresh tokens must never be included in JWT claims.

The signing secret or private key must be provided through secure environment configuration and must never be committed to Git.

## Refresh Tokens

Refresh tokens will be used to obtain new access tokens without requiring the user to log in repeatedly.

The refresh token strategy will:

- Use cryptographically secure random values.
- Store only a secure hash of each refresh token in PostgreSQL.
- Associate each token with a user session.
- Include expiration and revocation information.
- Support token rotation after successful refresh.
- Reject expired, revoked or previously used tokens.

A refresh token must not be accepted more than once after successful rotation.

## Refresh Token Rotation

The refresh workflow will:

1. Receive the existing refresh token.
2. Verify its hash against the stored session record.
3. Check expiration and revocation status.
4. Revoke or invalidate the existing token.
5. Generate a replacement refresh token.
6. Generate a new access token.
7. Persist the replacement refresh token hash.
8. Return the new tokens according to the configured response strategy.

Reuse of an already rotated refresh token should be treated as a possible session compromise and handled through the session security policy.

## Logout and Session Revocation

Logout must invalidate the active refresh token or session.

The backend will:

- Revoke the corresponding refresh token.
- Prevent further refresh operations using that token.
- Record relevant security audit information.
- Support revocation of all sessions when required, such as after a password change or suspected account compromise.

An already issued stateless access token may remain valid until its expiration unless an additional access-token revocation mechanism is implemented.

This limitation must be considered when selecting access-token lifetime.

## Token Transport and Browser Security

The frontend and backend must use HTTPS in production.

Refresh tokens should be transported using Secure, HttpOnly cookies with an appropriate SameSite policy when the browser-based authentication design supports it.

Cookie-based refresh flows must include appropriate CSRF protection.

Access tokens must not be placed in URLs or exposed through application logs.

The final frontend token-storage strategy must be documented before implementation.

## Role-Based Access Control

LogSage AI will initially support the following organization roles:

| Role | Responsibility |
|---|---|
| OWNER | Full organization control and ownership |
| ADMIN | Manage organization resources and members within permitted limits |
| MEMBER | Use assigned projects and operational features |
| VIEWER | Read-only access to permitted resources |

Permissions must be enforced by the backend, not only by frontend visibility.

Role definitions and permission checks must be centralized and testable.

## Organization-Level Authorization

LogSage AI is a multi-tenant application.

Every organization-owned resource must be accessed within the authenticated user's authorized organization context.

The backend must verify:

- The authenticated user belongs to the organization.
- The user's membership is active.
- The requested operation is allowed for the user's role.
- The requested project or resource belongs to the authorized organization.

A client-provided organization identifier must never be trusted without server-side authorization.

Cross-organization access must be rejected.

## Password Management

The authentication system should support:

- Password hashing.
- Password change.
- Secure password reset.
- Current password verification where appropriate.
- Session revocation after sensitive account changes.

Password reset tokens must be random, time-limited, single-use and securely stored.

## Database Requirements

The authentication database design will include:

### Users

- User ID.
- Name.
- Email.
- Password hash.
- Account status.
- Created timestamp.
- Updated timestamp.

### Refresh Tokens

- Token ID.
- User ID.
- Token hash.
- Session identifier.
- Expiration timestamp.
- Revocation timestamp.
- Created timestamp.
- Replaced-by token reference where required.

### Organization Memberships

- Membership ID.
- User ID.
- Organization ID.
- Assigned role.
- Membership status.
- Created timestamp.

Database constraints and indexes must support secure lookups and prevent invalid membership relationships.

## Security Requirements

- Use secure password hashing.
- Validate JWT signatures and expiration.
- Protect signing keys through environment configuration or a secure secret manager.
- Enforce authorization on every protected backend operation.
- Prevent organization-level data leakage.
- Apply rate limiting to authentication endpoints.
- Avoid revealing sensitive authentication details in errors.
- Protect refresh-token cookies against CSRF where applicable.
- Use HTTPS in production.
- Redact tokens and passwords from logs.
- Record important authentication and authorization events.

## Testing Requirements

Planned authentication tests:

- Successful registration.
- Duplicate email rejection.
- Password hashing verification.
- Successful login.
- Invalid password rejection.
- JWT signature validation.
- Expired access token rejection.
- Refresh token renewal.
- Refresh token rotation.
- Reuse of rotated token rejection.
- Logout and refresh-token revocation.
- Role-based permission enforcement.
- Organization isolation.
- Unauthorized project access rejection.
- Password reset token expiration and reuse prevention.

## Alternatives Considered

### Session-Based Authentication

Traditional server-side sessions were considered.

JWT access tokens with server-managed refresh sessions were selected to support API authentication while retaining control over long-lived sessions.

### Long-Lived Access Tokens

Rejected because a compromised access token would remain usable for an unnecessarily long period.

### Storing Plaintext Refresh Tokens

Rejected because database exposure could reveal active refresh credentials.

### Frontend-Only Authorization

Rejected because frontend checks can be bypassed and cannot provide reliable resource security.

## Consequences

### Positive

- Standardized API authentication.
- Short-lived access credentials.
- Controlled session renewal.
- Refresh-token revocation and rotation.
- Centralized authorization.
- Organization-level isolation.
- Better security auditability.

### Negative

- More complex session lifecycle management.
- Token rotation requires careful concurrency handling.
- Cookie-based refresh flows require CSRF protection.
- Stateless access tokens cannot be immediately invalidated without additional mechanisms.

## Implementation Notes

- Use Spring Security.
- Use a maintained JWT library.
- Keep token lifetimes configurable.
- Store refresh token hashes, not raw tokens.
- Use secure random token generation.
- Centralize role and permission checks.
- Validate organization membership on protected operations.
- Add automated authentication and authorization tests.

## Review Conditions

Revisit this decision if:

- Authentication requirements change.
- OAuth2 or external identity providers are introduced.
- Session security requirements change.
- A dedicated identity provider is adopted.
- Security testing identifies weaknesses in the token lifecycle.
