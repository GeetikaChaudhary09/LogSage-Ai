
# ADR-0013: CI/CD Pipeline and Automated Testing Strategy

## Status

Accepted - Initial architecture decision.

## Context

LogSage AI consists of multiple application components, including a React frontend, Java Spring Boot backend, Python FastAPI ML service and supporting infrastructure.

As the platform grows, manual testing and deployment will become insufficient for maintaining reliability and code quality.

The project requires an automated validation process that checks application code, tests critical workflows, detects dependency vulnerabilities and verifies container builds.

The CI/CD strategy must support independent component validation while also checking important cross-service integration points.

## Decision

LogSage AI will use GitHub Actions for continuous integration and controlled continuous delivery.

The CI pipeline will validate frontend, backend and ML service changes through automated checks.

Docker image builds and configuration validation will be included as the infrastructure matures.

Production deployment will require a controlled release workflow with appropriate environment protection and approval.

No deployment will be considered successful merely because the source code was committed or a workflow was configured.

## CI/CD Architecture

The initial pipeline will contain the following stages:

1. Repository checkout.
2. Environment and dependency setup.
3. Formatting and static checks.
4. Frontend validation.
5. Backend validation.
6. ML service validation.
7. Integration testing.
8. Security and dependency scanning.
9. Docker build validation.
10. Release and deployment stages when configured.

The pipeline must report failures clearly and prevent invalid changes from being treated as verified.

## GitHub Actions

GitHub Actions will be the initial CI/CD platform.

Workflow definitions will be maintained inside:

`.github/workflows/`

Workflows should be organized by responsibility rather than placing every operation into one unnecessarily large workflow.

The initial workflow categories may include:

- Pull request validation.
- Main branch validation.
- Backend tests.
- Frontend tests.
- ML service tests.
- Docker image builds.
- Security scanning.
- Deployment.

Workflow reuse should be introduced where it improves maintainability.

## Trigger Strategy

Automated validation should run for relevant:

- Pull requests.
- Pushes to the main branch.
- Changes to application source code.
- Changes to dependency configuration.
- Changes to Docker and infrastructure configuration.
- Changes to CI workflow definitions.

Path filters may be used to avoid running unrelated expensive jobs, provided shared dependencies and integration requirements are not missed.

## Frontend Testing

The React and TypeScript frontend will use an appropriate automated testing stack.

Planned validation includes:

### Static Checks

- TypeScript type checking.
- ESLint.
- Formatting checks.
- Production build validation.

### Component Tests

- Component rendering.
- User interactions.
- Form validation.
- Loading states.
- Error states.
- Empty states.

### Feature Tests

- Authentication flows.
- Protected routing.
- Dashboard data rendering.
- Log filtering and pagination.
- Incident status updates.
- Alert rule forms.
- Organization and project navigation.

Frontend tests must use controlled test data and must not imply that mocked responses represent production telemetry.

## Backend Testing

The Java Spring Boot backend will use JUnit 5, Mockito and suitable Spring testing facilities.

Planned test categories include:

### Unit Tests

- Service-layer business logic.
- Validation.
- Authorization decisions.
- Incident correlation.
- Alert evaluation.
- Utility functions.

### Repository Tests

- Entity persistence.
- Relationships and constraints.
- Query correctness.
- Organization-scoped access.
- Pagination and filtering.

### Integration Tests

- Authentication.
- JWT validation.
- Refresh token rotation.
- Organization isolation.
- API key verification.
- Log ingestion.
- RabbitMQ publishing and consumption.
- Database persistence.
- Incident lifecycle.
- Alert evaluation.

Tests must verify failure scenarios as well as successful operations.

## ML Service Testing

The Python FastAPI ML service will use Pytest.

Planned validation includes:

- API endpoint behaviour.
- Request schema validation.
- Feature extraction.
- Missing and invalid feature handling.
- Model loading.
- Inference behaviour.
- Model artifact compatibility.
- Training pipeline checks.
- Model evaluation.
- Error handling.
- Service health checks.

ML tests must distinguish valid anomaly predictions from inference errors.

Synthetic training and test datasets must be clearly identified as synthetic.

Model evaluation must use appropriate metrics such as precision, recall, F1-score and confusion matrices where applicable.

## Integration Testing

Integration testing will verify communication between application components.

Important integration paths include:

- Frontend to backend API.
- Backend to PostgreSQL.
- Backend to Redis.
- Backend to RabbitMQ.
- Backend to ML service.
- RabbitMQ consumers to PostgreSQL.
- Anomaly events to incident processing.
- Alert events to notification processing.

Integration tests should use isolated test infrastructure or controlled test containers where appropriate.

Production credentials and production data must never be used in ordinary CI tests.

## End-to-End Testing

The platform will eventually include end-to-end tests covering the complete user journey.

The primary workflow will be:

1. Register a user.
2. Authenticate.
3. Create an organization.
4. Create a project.
5. Generate a project API key.
6. Submit application logs.
7. Verify accepted logs are persisted.
8. Retrieve logs through the application.
9. Execute anomaly analysis.
10. Verify evidence-based incident creation when the test scenario meets configured criteria.
11. Create and evaluate an alert rule.
12. Resolve an incident.
13. Verify relevant history and notifications.

End-to-end tests must use controlled test fixtures and clearly isolated environments.

A successful workflow must be verified through actual API and database behaviour, not merely frontend navigation.

## Test Data Management

Tests must use controlled and reproducible data.

The test strategy will distinguish:

- Unit test fixtures.
- Integration test data.
- End-to-end test data.
- Synthetic ML datasets.
- Production telemetry.

Test data must not leak between unrelated tests.

Tests must not create misleading production metrics or incidents.

## Security and Dependency Scanning

The CI pipeline should include automated checks for:

- Dependency vulnerabilities.
- Known insecure package versions.
- Secret exposure.
- Static code issues.
- Container image vulnerabilities where supported.

Potential tools may include GitHub dependency scanning, secret scanning and appropriate ecosystem-specific security tools.

The exact tooling and configuration will be finalized during CI implementation.

Security checks must not print credentials or sensitive environment variables.

## Code Quality

The project will maintain consistent quality checks across languages.

### Frontend

- ESLint.
- TypeScript checks.
- Formatting validation.
- Production build.

### Backend

- Java compilation.
- Unit tests.
- Integration tests.
- Static analysis where configured.

### ML Service

- Python test execution.
- Linting.
- Type and schema validation where configured.
- Model pipeline checks.

Quality checks must be repeatable locally and in CI.

## Docker Build Validation

Docker image builds will be validated for supported application components.

The pipeline should verify:

- Dockerfile syntax and build success.
- Required build dependencies.
- Runtime configuration.
- Expected application artifacts.
- Container startup.
- Health check behaviour where practical.

Docker Compose configuration should also be validated as the infrastructure environment becomes available.

A successful image build does not alone prove that the complete distributed application works correctly.

## Branch and Pull Request Strategy

The main branch should contain validated project changes.

Pull requests should undergo the required automated checks before merging.

The project should adopt:

- Clear commit messages.
- Focused pull requests.
- Review of significant architectural changes.
- Successful required CI checks.
- Appropriate branch protection settings.

Direct changes to the main branch should be restricted when repository collaboration requirements justify it.

## Release Strategy

Releases should use identifiable versions or release tags.

The release process should record:

- Source commit.
- Build result.
- Tested components.
- Image versions where applicable.
- Deployment environment.
- Deployment result.

Production deployments must use explicitly configured environments and protected credentials.

Rollback procedures must be documented for production releases.

## Environment Separation

The CI/CD process must distinguish:

### Development

Used for local development and rapid feedback.

### Testing

Used for automated tests and integration validation.

### Staging

Used for production-like verification before release when configured.

### Production

Used for real application traffic and persistent production data.

Secrets, databases and application configuration must remain appropriately isolated between environments.

## Secrets Management

CI/CD workflows must:

- Use protected GitHub secrets or an appropriate secret-management integration.
- Avoid hardcoded credentials.
- Avoid printing secrets in workflow logs.
- Restrict deployment permissions.
- Use least-privilege access.
- Separate staging and production credentials.

Production credentials must never be exposed to untrusted pull-request workflows.

## Failure Handling

When a required CI check fails:

1. The workflow must report the failing stage.
2. The relevant logs must be available for diagnosis.
3. The change must not be treated as validated.
4. The failure must be corrected and checks rerun.

Deployment failures must be reported separately from test failures.

The workflow must not silently ignore failed required checks.

## Quality Gates

The project will introduce quality gates incrementally.

Initial required gates should include:

- Successful dependency installation.
- Frontend type checking and build.
- Backend compilation and tests.
- ML service tests.
- Relevant integration checks.
- No accidental secret exposure.
- Successful Docker builds when configured.

Coverage thresholds and additional static-analysis gates should be introduced after a baseline is established.

Coverage percentage alone must not be treated as proof of software correctness.

## Observability of CI/CD

The project should maintain visibility into:

- Workflow success and failure.
- Test execution time.
- Build failures.
- Deployment results.
- Dependency scanning findings.
- Release history.

CI/CD failures must be distinguishable from application runtime failures.

## Testing Requirements for the Pipeline

Planned pipeline validation includes:

- Pull request workflow execution.
- Main branch workflow execution.
- Frontend build failure detection.
- Backend test failure detection.
- ML test failure detection.
- Dependency scanning.
- Secret scanning.
- Docker image build validation.
- Environment secret isolation.
- Deployment approval enforcement.
- Release tagging.
- Rollback procedure verification.

## Alternatives Considered

### Manual Testing and Deployment

Rejected as the only validation strategy because it is inconsistent and difficult to reproduce.

### Jenkins

Considered as an alternative CI platform but not selected for the initial implementation because GitHub Actions integrates directly with the repository.

### GitLab CI

Not selected because GitHub is the project's initial source-control and collaboration platform.

### Fully Automated Production Deployment Without Approval

Not selected initially because production deployment requires controlled credentials, environment protection and a verified release process.

## Consequences

### Positive

- Repeatable validation.
- Earlier defect detection.
- Better code quality.
- Improved security visibility.
- Consistent builds.
- Safer release management.
- Better confidence in integration workflows.
- Clearer project development history.

### Negative

- CI configuration requires maintenance.
- Automated tests increase development setup effort.
- Integration tests require additional infrastructure.
- Security scanning may produce findings requiring investigation.
- Deployment workflows require careful credential management.

## Implementation Notes

- Use GitHub Actions.
- Store workflows in `.github/workflows/`.
- Validate frontend, backend and ML independently.
- Add integration tests for critical service boundaries.
- Add end-to-end tests for the complete user journey.
- Use controlled test data.
- Add dependency and secret scanning.
- Validate Docker images.
- Protect production deployment credentials.
- Document release and rollback procedures.
- Do not declare CI/CD complete until actual workflows execute successfully.

## Review Conditions

Revisit this decision if:

- Repository hosting changes.
- CI execution costs become significant.
- Deployment infrastructure changes.
- Testing requirements expand.
- Security compliance requirements change.
- A different CI/CD platform becomes operationally preferable.
