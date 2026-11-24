
# LogSage AI - Contributing Guide

## 1. Introduction

Thank you for your interest in contributing to LogSage AI.

This document defines the development workflow, coding standards, Git conventions and review process for the project.

All contributions should maintain the platform's security, reliability, maintainability and documentation quality.

## 2. Project Structure

The repository is organized into the following main areas:

- `frontend/` - React and TypeScript application
- `backend/` - Java Spring Boot application
- `ml-service/` - Python ML and inference service
- `sample-apps/` - Example applications for log ingestion
- `infrastructure/` - Docker, Nginx and deployment configuration
- `docs/` - Architecture and project documentation
- `scripts/` - Development and maintenance scripts
- `.github/workflows/` - GitHub Actions automation

## 3. Development Setup

Before contributing:

1. Install the required development runtimes.
2. Clone the repository.
3. Review the README.
4. Read the architecture and development documentation.
5. Configure local environment variables.
6. Install the relevant component dependencies.
7. Run the available tests.

Do not commit local environment files or generated dependencies.

## 4. Branching Strategy

The project uses a simple branch-based workflow.

Main branch:

- `main` - stable integration branch

Feature branches:

- `feature/short-description`
- `fix/short-description`
- `docs/short-description`
- `refactor/short-description`
- `test/short-description`

Examples:

```bash
git switch -c feature/log-ingestion
git switch -c fix/jwt-validation
git switch -c docs/update-api-guide
```

Avoid committing unfinished work directly to `main`.

## 5. Commit Message Convention

Use Conventional Commits.

Format:

```text
<type>: <short description>
```

Common types:

| Type | Purpose |
|---|---|
| feat | New functionality |
| fix | Bug fix |
| docs | Documentation changes |
| refactor | Code restructuring |
| test | Test additions or changes |
| chore | Maintenance tasks |
| build | Build system changes |
| ci | CI/CD changes |
| perf | Performance improvements |
| security | Security-related improvements |

Examples:

```text
feat: add structured log ingestion
fix: validate expired refresh tokens
docs: update deployment guide
test: add organization isolation tests
chore: update development dependencies
```

Write clear, concise commit messages.

## 6. Coding Standards

### General

- Prefer readable and maintainable code.
- Use meaningful variable and function names.
- Keep functions focused.
- Avoid unnecessary duplication.
- Handle errors explicitly.
- Validate external input.
- Document non-obvious behaviour.
- Avoid hardcoded secrets and environment-specific values.

### Java Backend

- Follow consistent Java naming conventions.
- Keep controllers focused on HTTP handling.
- Place business logic in services.
- Use repositories for data access.
- Use DTOs for API request and response models.
- Validate incoming requests.
- Apply authorization checks on protected operations.
- Write tests for important business logic.

### Frontend

- Use TypeScript for application code.
- Build reusable components.
- Keep API communication in a dedicated layer.
- Handle loading, empty, success and error states.
- Validate forms.
- Avoid fabricated production data.
- Keep components accessible and responsive.

### Python ML Service

- Follow Python naming conventions.
- Use type hints where appropriate.
- Validate request and response schemas.
- Separate training, feature engineering and inference.
- Keep model configuration explicit.
- Write tests for feature extraction and predictions.
- Document model assumptions and limitations.

## 7. Testing Requirements

Contributors should run the relevant tests before submitting changes.

Planned testing layers:

- Java unit tests
- Java integration tests
- Python tests with Pytest
- Frontend component tests
- API tests
- End-to-end tests

A change must not be considered verified merely because the application starts.

Do not ignore or remove failing tests without understanding the cause.

## 8. API and Database Changes

For API changes:

- Follow the `/api/v1` versioning convention.
- Maintain consistent response structures.
- Validate request payloads.
- Document new endpoints.
- Enforce authorization.
- Add tests for relevant cases.

For database changes:

- Use Flyway migrations.
- Never silently change an already-applied migration.
- Define appropriate constraints and indexes.
- Consider existing data compatibility.
- Test migration behaviour.

## 9. Security Requirements

Every contribution must follow the security architecture.

Never commit:

- Passwords
- JWT secrets
- API keys
- Database credentials
- Cloud credentials
- Private certificates
- Production environment files

Additional requirements:

- Enforce backend authorization.
- Protect organization-level data.
- Validate untrusted input.
- Avoid exposing sensitive information in logs.
- Use secure dependency versions.
- Review security implications of new endpoints.

## 10. Documentation Requirements

Update documentation when a change affects:

- Architecture
- API contracts
- Database schema
- Environment configuration
- ML pipeline
- Security behaviour
- Deployment procedures
- Development workflows

Keep implementation status accurate.

Do not document planned functionality as completed functionality.

## 11. Pull Request Process

Before opening a pull request:

1. Ensure the branch is up to date.
2. Review the changed files.
3. Run relevant tests.
4. Check for accidental secrets.
5. Update related documentation.
6. Write a clear commit message.

A pull request should include:

- Purpose of the change
- Summary of implementation
- Important design decisions
- Tests performed
- Known limitations
- Screenshots when UI changes are involved

Keep pull requests focused and reasonably small.

## 12. Code Review

Reviewers should check:

- Correctness
- Readability
- Security
- Authorization
- Error handling
- Performance
- Test coverage
- Documentation
- Compatibility with existing functionality

Resolve review comments before merging.

## 13. Reporting Issues

Bug reports should include:

- Clear description
- Reproduction steps
- Expected result
- Actual result
- Environment information
- Relevant error output
- Screenshots where useful

Never include credentials or private user data.

## 14. Contribution Principles

All contributors should:

- Communicate respectfully.
- Prefer evidence over assumptions.
- Avoid unnecessary complexity.
- Keep changes focused.
- Test before claiming completion.
- Document limitations honestly.
- Preserve existing working functionality.

## 15. Implementation Status

This document defines the planned contribution workflow.

Branch protection, automated checks and pull request requirements will be configured as the GitHub workflow is implemented.
