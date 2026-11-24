
# LogSage AI — API Reference

## 1. Overview

LogSage AI exposes versioned REST APIs for authentication, workspace management, project configuration, log ingestion, monitoring, anomaly detection, incidents and alerts.

**Base path:** `/api/v1`

**Status:** Planned API contract. Endpoints will be considered implemented only after backend implementation and verification.

## 2. API Conventions

- Protocol: HTTPS in production
- Data format: JSON
- Character encoding: UTF-8
- API version: v1
- Timestamps: ISO 8601 UTC
- Identifiers: UUID where appropriate
- Authentication: Bearer access token or ingestion API key

### Standard response format

Successful response:

```json
{
  "success": true,
  "message": "Request completed successfully",
  "data": {},
  "timestamp": "2026-11-24T12:00:00Z"
}
```

Error response:

```json
{
  "success": false,
  "message": "Validation failed",
  "errorCode": "VALIDATION_ERROR",
  "details": [],
  "timestamp": "2026-11-24T12:00:00Z"
}
```

Actual response structures will be finalized during implementation.

## 3. Authentication

User authentication uses JWT access and refresh tokens.

| Method | Endpoint | Purpose |
|---|---|---|
| POST | /auth/register | Register account |
| POST | /auth/login | Authenticate user |
| POST | /auth/refresh | Refresh access token |
| POST | /auth/logout | Revoke refresh session |
| GET | /auth/me | Get current user |

Access token:

```http
Authorization: Bearer <access-token>
```

Refresh-token rotation or an equivalently secure workflow will be implemented.

## 4. Organization and Workspace APIs

| Method | Endpoint | Purpose |
|---|---|---|
| POST | /organizations | Create organization |
| GET | /organizations | List accessible organizations |
| GET | /organizations/{organizationId} | Get organization |
| PATCH | /organizations/{organizationId} | Update organization |
| GET | /organizations/{organizationId}/members | List members |
| POST | /organizations/{organizationId}/members | Invite or add member |
| PATCH | /organizations/{organizationId}/members/{memberId} | Update member role |
| DELETE | /organizations/{organizationId}/members/{memberId} | Remove member |

All operations require membership and role authorization.

## 5. Project APIs

| Method | Endpoint | Purpose |
|---|---|---|
| POST | /organizations/{organizationId}/projects | Create project |
| GET | /organizations/{organizationId}/projects | List projects |
| GET | /projects/{projectId} | Get project |
| PATCH | /projects/{projectId} | Update project |
| DELETE | /projects/{projectId} | Delete or archive project |

Project access must be validated against the authenticated user's organization membership.

## 6. API Key Management

| Method | Endpoint | Purpose |
|---|---|---|
| POST | /projects/{projectId}/api-keys | Create ingestion key |
| GET | /projects/{projectId}/api-keys | List key metadata |
| DELETE | /api-keys/{keyId} | Revoke key |
| POST | /api-keys/{keyId}/rotate | Rotate key |

Security requirements:

- Generate cryptographically secure random keys.
- Store only secure hashes.
- Show raw key only once.
- Never return raw keys from list or detail endpoints.
- Validate project association and key status.

## 7. Log Ingestion API

### Endpoint

```http
POST /api/v1/ingest/logs
X-LogSage-API-Key: <ingestion-key>
Content-Type: application/json
```

### Single log request

```json
{
  "timestamp": "2026-11-24T10:30:00Z",
  "level": "ERROR",
  "serviceName": "payment-service",
  "environment": "production",
  "message": "Payment processing failed",
  "traceId": "trace-123",
  "spanId": "span-456",
  "requestId": "request-789",
  "endpoint": "/api/payments",
  "httpStatus": 500,
  "durationMs": 1250,
  "metadata": {
    "paymentProvider": "example"
  }
}
```

### Batch log request

```json
{
  "logs": [
    {
      "timestamp": "2026-11-24T10:30:00Z",
      "level": "INFO",
      "serviceName": "payment-service",
      "environment": "production",
      "message": "Payment request received"
    },
    {
      "timestamp": "2026-11-24T10:30:01Z",
      "level": "ERROR",
      "serviceName": "payment-service",
      "environment": "production",
      "message": "Payment processing failed"
    }
  ]
}
```

Supported levels:

- TRACE
- DEBUG
- INFO
- WARN
- ERROR
- FATAL

The API will validate payload size, field types, log levels and API key permissions.

### Ingestion response

An accepted request will return an appropriate success status and an ingestion acknowledgement.

For asynchronous processing, acknowledgement of acceptance must not be represented as proof that every log has already been persisted.

## 8. Log Search APIs

| Method | Endpoint | Purpose |
|---|---|---|
| GET | /projects/{projectId}/logs | Search project logs |
| GET | /projects/{projectId}/logs/{logId} | Get log details |

Supported query parameters:

- page
- size
- sort
- startTime
- endTime
- level
- service
- environment
- search
- traceId
- requestId

All query parameters must be validated and safely parameterized.

## 9. Metrics APIs

| Method | Endpoint | Purpose |
|---|---|---|
| GET | /projects/{projectId}/metrics/summary | Summary metrics |
| GET | /projects/{projectId}/metrics/log-volume | Log volume trend |
| GET | /projects/{projectId}/metrics/error-rate | Error rate trend |
| GET | /projects/{projectId}/metrics/severity | Logs by severity |
| GET | /projects/{projectId}/metrics/services | Service metrics |

Metrics must be calculated from actual persisted records.

Unavailable metrics must be distinguished from genuine zero values.

## 10. Anomaly APIs

| Method | Endpoint | Purpose |
|---|---|---|
| GET | /projects/{projectId}/anomalies | List anomaly results |
| GET | /anomalies/{anomalyId} | Get prediction details |
| GET | /ml/health | ML service health |
| GET | /ml/model/info | Model metadata |

The ML service independently exposes:

```http
GET /health
GET /model/info
POST /predict
```

Prediction results must include:

- Anomaly score
- Anomaly flag
- Model version
- Prediction timestamp
- Input feature summary

An inference failure must never be converted into an anomaly result.

## 11. Incident APIs

| Method | Endpoint | Purpose |
|---|---|---|
| GET | /projects/{projectId}/incidents | List incidents |
| GET | /incidents/{incidentId} | Incident details |
| POST | /incidents/{incidentId}/acknowledge | Acknowledge incident |
| POST | /incidents/{incidentId}/investigate | Start investigation |
| PATCH | /incidents/{incidentId}/assign | Assign incident |
| POST | /incidents/{incidentId}/resolve | Resolve incident |
| POST | /incidents/{incidentId}/reopen | Reopen incident |
| GET | /incidents/{incidentId}/timeline | Incident history |
| GET | /incidents/{incidentId}/logs | Related log evidence |

Incident transitions must be validated by the backend.

## 12. Alert APIs

| Method | Endpoint | Purpose |
|---|---|---|
| POST | /projects/{projectId}/alert-rules | Create rule |
| GET | /projects/{projectId}/alert-rules | List rules |
| GET | /alert-rules/{ruleId} | Get rule |
| PATCH | /alert-rules/{ruleId} | Update rule |
| DELETE | /alert-rules/{ruleId} | Delete or disable rule |
| GET | /projects/{projectId}/alert-history | Alert history |
| GET | /notifications | In-app notifications |
| PATCH | /notifications/{notificationId}/read | Mark as read |

Rules may evaluate:

- Error count
- Error rate
- Log volume
- Service-specific errors
- Anomaly detection score

Evaluation windows and cooldown periods must be configurable.

## 13. Service APIs

| Method | Endpoint | Purpose |
|---|---|---|
| GET | /projects/{projectId}/services | List services |
| GET | /services/{serviceId} | Service details |
| GET | /services/{serviceId}/metrics | Service metrics |
| GET | /services/{serviceId}/logs | Service logs |

Service information will be derived from actual ingested telemetry.

## 14. Settings and Audit APIs

| Method | Endpoint | Purpose |
|---|---|---|
| GET | /users/me | User profile |
| PATCH | /users/me | Update profile |
| GET | /organizations/{organizationId}/audit-logs | Audit history |

Sensitive operations must generate appropriate audit events.

## 15. Pagination

Paginated endpoints will use a consistent response structure.

```json
{
  "success": true,
  "data": {
    "items": [],
    "page": 0,
    "size": 20,
    "totalElements": 0,
    "totalPages": 0
  }
}
```

Maximum page size will be configurable.

## 16. HTTP Status Codes

| Status | Meaning |
|---|---|
| 200 | Successful request |
| 201 | Resource created |
| 202 | Accepted for asynchronous processing |
| 204 | Successful request without response body |
| 400 | Invalid request |
| 401 | Authentication required |
| 403 | Permission denied |
| 404 | Resource not found |
| 409 | Conflict |
| 422 | Validation or semantic error |
| 429 | Rate limit exceeded |
| 500 | Internal server error |
| 503 | Required service unavailable |

## 17. API Security

- JWT authentication for user-facing endpoints.
- Ingestion API key authentication for log ingestion.
- Backend authorization for every organization-owned resource.
- Request validation.
- Rate limiting.
- Request-size limits.
- CORS restrictions.
- Secure error responses.
- No secret values in responses or logs.

## 18. API Documentation

OpenAPI documentation will be generated from the implemented backend endpoints.

Swagger UI will be enabled in local development and configured securely for production.

## 19. Implementation Status

This file defines the planned API contract.

Endpoints will be marked implemented only after:

1. Controller and service implementation.
2. Request and response validation.
3. Authentication and authorization tests.
4. Integration tests.
5. API documentation verification.

Endpoint paths and response contracts may be refined during implementation. Any changes must be reflected in this document.
