
# LogSage AI — Development Guide

## 1. Introduction

This guide explains how to configure the local development environment and contribute to LogSage AI.

The project is developed as a monorepo containing:

- React frontend
- Java Spring Boot backend
- Python FastAPI machine learning service
- PostgreSQL database
- Redis cache
- RabbitMQ message broker

**Status:** Development setup is being established. Commands will be verified as each service is implemented.

## 2. Prerequisites

Required tools:

| Tool | Planned version |
|---|---|
| Java | 24 |
| Node.js | 24 LTS |
| npm | Bundled with Node.js |
| Python | 3.14 |
| PostgreSQL | 17 |
| Git | Latest stable |
| Docker | Latest stable |

Docker, Redis and RabbitMQ setup will be covered in the infrastructure phase.

## 3. Repository Setup

Clone the repository:

```bash
git clone https://github.com/GeetikaChaudhary09/LogSage-Ai.git
cd LogSage-Ai
```

## 4. Backend Setup

The backend uses Java 24 and Spring Boot.

The Maven Wrapper will be included with the backend project.

Planned commands:

```powershell
cd backend
.\mvnw.cmd spring-boot:run
```

Run backend tests:

```powershell
.\mvnw.cmd test
```

These commands become usable after the Spring Boot project is generated.

## 5. Frontend Setup

The frontend uses React, TypeScript and Vite.

Planned setup:

```powershell
cd frontend
npm install
npm run dev
```

Run frontend production build:

```powershell
npm run build
```

## 6. Machine Learning Service Setup

Python 3.14 is used for the ML service.

Create a virtual environment from the repository root:

```powershell
py -3.14 -m venv ml-service\.venv
```

Activate it:

```powershell
.\ml-service\.venv\Scripts\Activate.ps1
```

Upgrade pip:

```powershell
python -m pip install --upgrade pip
```

Install dependencies after the ML requirements file is created:

```powershell
pip install -r ml-service\requirements.txt
```

Run the service after implementation:

```powershell
cd ml-service
python -m uvicorn app.main:app --reload
```

## 7. Database Setup

PostgreSQL is the primary persistent database.

Local development will use environment-based configuration.

Database credentials must never be committed to Git.

Database creation, Flyway migrations and connection configuration will be documented when the backend database integration is implemented.

## 8. Environment Variables

Copy the example configuration when it becomes available:

```powershell
Copy-Item .env.example .env
```

Configure local secrets and service connection values in `.env`.

Never commit `.env` or real credentials.

## 9. Testing

### Backend

```powershell
cd backend
.\mvnw.cmd test
```

### Frontend

```powershell
cd frontend
npm run build
```

### ML Service

```powershell
cd ml-service
pytest
```

Test commands will be expanded as the corresponding test suites are implemented.

## 10. Git Workflow

Use the following workflow:

1. Start from an updated main branch.
2. Create a feature branch.
3. Implement one focused change.
4. Run relevant tests.
5. Review Git changes.
6. Commit using Conventional Commits.
7. Push the branch.
8. Open a pull request when appropriate.

Example:

```bash
git switch main
git pull origin main
git switch -c feat/log-ingestion
```

Example commit messages:

```text
feat: implement structured log ingestion
fix: validate project API key association
test: add ingestion controller tests
docs: document local development setup
refactor: separate incident processing logic
```

## 11. Code Quality Guidelines

- Follow consistent naming conventions.
- Keep methods and components focused.
- Validate external input.
- Avoid exposing database entities through APIs.
- Handle errors explicitly.
- Add tests for important business logic.
- Keep dependencies justified and updated.
- Document significant architectural decisions.

## 12. Security Guidelines

- Never commit secrets or credentials.
- Use environment variables for local configuration.
- Hash ingestion API keys before storage.
- Enforce workspace authorization in the backend.
- Avoid logging passwords, tokens or raw API keys.
- Validate all external input.

## 13. Troubleshooting

Common development issues and verified solutions will be added as implementation progresses.

When reporting a problem, include:

- Command executed
- Complete error output
- Relevant application logs
- Operating system and tool version

## 14. Development Status

This guide is maintained alongside implementation.

Commands and workflows must be tested before being marked as verified.
