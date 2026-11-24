
# LogSage AI - Troubleshooting Guide

## 1. Overview

This document records common development and operational problems encountered while building and running LogSage AI.

The solutions in this guide are intended to be verified against the actual project configuration before being treated as confirmed fixes.

## 2. General Development Issues

### Project command is not recognized

Possible causes:
- Required runtime is not installed.
- Runtime is not added to PATH.
- Terminal was not restarted after installation.

Troubleshooting:
1. Check the installed runtime version.
2. Verify PATH configuration.
3. Restart the terminal.
4. Reopen the project.

### Dependency installation fails

Possible causes:
- Network connectivity issues.
- Incompatible dependency versions.
- Package registry unavailable.
- Corrupted local dependency cache.

Troubleshooting:
1. Read the complete error message.
2. Verify the runtime version.
3. Check dependency compatibility.
4. Retry installation.
5. Avoid deleting lock files without understanding the impact.

## 3. Git and GitHub Issues

### Changes are not appearing in Git status

Check:

```bash
git status
```

Possible causes:
- File is ignored by `.gitignore`.
- Changes were already committed.
- File was created in a different directory.

Inspect ignored files:

```bash
git check-ignore -v path/to/file
```

### Push is rejected

Check the current branch:

```bash
git branch
```

Check the remote:

```bash
git remote -v
```

Fetch remote changes:

```bash
git fetch origin
```

Review differences before merging or rebasing.

Never force-push shared branches without understanding the consequences.

## 4. Backend Issues

### Backend fails to start

Possible causes:
- Incorrect Java version.
- Invalid application configuration.
- Port already in use.
- Database connection failure.
- Missing environment variables.

Troubleshooting:
1. Read the complete startup exception.
2. Verify the Java version.
3. Check application configuration.
4. Confirm required services are available.
5. Verify the configured port.
6. Restart after correcting the root cause.

### Maven build fails

Run:

```powershell
.\mvnw.cmd clean verify
```

Check:
- Dependency resolution.
- Java compatibility.
- Compilation errors.
- Test failures.
- Maven Wrapper configuration.

Do not skip failing tests simply to obtain a successful build.

### Authentication fails

Check:
- Login request payload.
- Password hashing configuration.
- JWT signing configuration.
- Token expiration.
- Security filter configuration.
- User account status.

Never print passwords or complete authentication tokens while debugging.

## 5. Database Issues

### PostgreSQL connection refused

Possible causes:
- PostgreSQL service is stopped.
- Incorrect host or port.
- Incorrect connection settings.
- Database server is not ready.

Verify the PostgreSQL service and connection configuration.

### Database authentication fails

Check:
- Database username.
- Database password.
- Database name.
- Connection URL.
- Database permissions.

Do not commit database credentials.

### Flyway migration fails

Check:
- Migration file naming.
- SQL syntax.
- Existing database schema.
- Migration history.
- Database permissions.

Do not manually edit an already-applied migration in a shared environment.

Create a new migration for schema changes.

## 6. Frontend Issues

### Frontend development server does not start

Run:

```bash
npm install
npm run dev
```

Check:
- Node.js version.
- Dependency installation.
- Vite configuration.
- Port conflicts.
- Terminal error output.

### API requests fail

Check:
- Backend availability.
- API base URL.
- CORS configuration.
- Authentication headers.
- Network response status.
- Request and response payloads.

Do not treat a failed request as successful data retrieval.

### Page displays incorrect or missing data

Check:
- API response.
- TanStack Query configuration.
- Loading and error states.
- Data transformation.
- Browser console errors.

Avoid inserting fabricated values to hide API failures.

## 7. Python ML Service Issues

### Virtual environment activation fails

PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

If script execution is restricted, review the PowerShell execution policy rather than permanently weakening system security.

### Python dependency installation fails

Check:
- Python version.
- Package compatibility.
- Virtual environment activation.
- Dependency versions.
- Package installation output.

### FastAPI service does not start

Run the configured Uvicorn command and inspect the complete traceback.

Check:
- Application module path.
- Installed dependencies.
- Port availability.
- Environment configuration.

### Model loading fails

Check:
- Model artifact path.
- File permissions.
- Model version.
- Training completion.
- Joblib compatibility.

A missing or invalid model must produce a clear service error.

It must not silently report normal predictions.

## 8. RabbitMQ Issues

### Message is not processed

Check:
- Broker availability.
- Queue declaration.
- Exchange and routing configuration.
- Consumer status.
- Message acknowledgement.
- Consumer logs.

### Messages are repeatedly retried

Investigate:
- Invalid payloads.
- Transient database failures.
- Consumer exceptions.
- Retry configuration.
- Dead-letter queue routing.

Messages must not be acknowledged before required persistence succeeds.

## 9. Redis Issues

### Redis connection fails

Check:
- Redis availability.
- Host and port.
- Authentication configuration.
- Network connectivity.

Redis is not the primary source of persistent application data.

The application should handle Redis outages according to the relevant feature's failure policy.

## 10. Docker Issues

### Docker command is unavailable

Check:
- Docker Desktop installation.
- Docker Desktop startup.
- Docker CLI availability.
- System PATH.

### Container exits unexpectedly

Inspect:

```bash
docker compose ps
docker compose logs
```

Check:
- Startup errors.
- Environment configuration.
- Port conflicts.
- Container health.
- Dependency readiness.

### Data disappears after restart

Verify that the relevant persistent Docker volumes are configured correctly.

Do not assume container filesystem data is persistent.

## 11. API and Ingestion Issues

### Log ingestion returns an error

Check:
- API key validity.
- Project association.
- Request content type.
- Payload schema.
- Log level.
- Timestamp format.
- Payload and batch limits.
- Rate limiting.

### Logs are not visible in search

Check:
- Ingestion response.
- Message processing status.
- Database persistence.
- Search filters.
- Time range and timezone.
- Organization and project permissions.

Do not assume that an accepted request means a log is already searchable.

## 12. Anomaly Detection Issues

### ML prediction is unavailable

Check:
- ML service health.
- Model availability.
- Input feature schema.
- Prediction request logs.
- Model version.

An ML service failure must not block log ingestion.

### Unexpected anomaly results

Check:
- Feature extraction.
- Training data quality.
- Model configuration.
- Evaluation metrics.
- Model version.
- Prediction thresholds.

Do not modify thresholds simply to hide incorrect predictions.

## 13. Security and Configuration Issues

### Environment variables are missing

Check the relevant environment example and application configuration.

Ensure required variables are present without exposing their secret values.

### Unauthorized API response

Check:
- Authentication status.
- Token validity.
- User role.
- Organization membership.
- Resource ownership.

Do not bypass backend authorization to resolve a frontend access issue.

## 14. Debugging Guidelines

When investigating a problem:

1. Reproduce the issue.
2. Record the exact error message.
3. Identify the affected component.
4. Check relevant application logs.
5. Isolate the root cause.
6. Apply the smallest appropriate fix.
7. Run relevant tests.
8. Verify that existing functionality still works.
9. Document recurring issues.

Avoid changing multiple unrelated components during debugging.

## 15. Reporting a Bug

A useful bug report should include:

- Short issue title.
- Environment and runtime versions.
- Steps to reproduce.
- Expected behaviour.
- Actual behaviour.
- Error message or relevant logs.
- Screenshots when useful.
- Relevant Git branch or commit.

Never include passwords, tokens, API keys or other secrets in bug reports.

## 16. Implementation Status

This guide is a planned troubleshooting reference.

Solutions will be marked verified only after reproducing the issue, applying the fix and confirming the result.
