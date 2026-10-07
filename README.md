# AI Startup Pitch Feedback Assistant

A course project for MAIE 6000C: Engineering AI Products: From Prototype to Production.

## Project Overview

We are building an AI Startup Pitch Feedback Assistant for student entrepreneurs who want to explain their business ideas more clearly.

A user submits a project title and a short pitch description. The system stores the submission, creates a background analysis job, and returns structured feedback for the user to review.

The feedback will include:
- A concise summary.
- The problem, target customer, proposed solution, and value proposition.
- Missing or unclear information.
- Three improvement suggestions.

The user decides which suggestions to use, change, or ignore. The original pitch is preserved.

## Current Status

This repository is being developed from the course starter template.

The Week 04 documents describe our proposed architecture and implementation plan. The pitch-specific features are planned; their presence in the proposal does not mean they have already been implemented or tested.

The setup instructions below are inherited from the starter. We will update them and record verification results as implementation progresses.

## Planned Workflow

1. Submit a project title and pitch description.
2. Validate and store the pitch.
3. Create a background analysis job.
4. Have the worker call the internal AI service.
5. Save the structured feedback and processing status.
6. Let the user retrieve and review the result.

If analysis fails, the system will preserve the pitch, show a failed status, and allow a retry.

## Architecture

We will extend the starter's four main components:

| Component | Role |
| --- | --- |
| API service | Accept submissions and provide access to status, results, and retries. |
| PostgreSQL | Store pitches, processing jobs, and analysis results. |
| Worker | Process background jobs and save their outcomes. |
| Internal AI service | Analyze pitch content and generate structured feedback. |

We will retain Docker Compose for local deployment and Alembic for database migrations.

## Scope

Our priority is a complete text-based submission and feedback workflow. We will use the API interface for the initial demonstration and add a simple web interface if time allows.

The project does not include investment prediction, commercial viability evaluation, complete market research, investor matching, or a mobile application.

## Project Documents

- [Architecture proposal](docs/architecture.md)
- [Week 04 proposal and project plan](submissions/week04/README.md)
- [Operations documentation](docs/operations.md)

The required Week 04 submission tag is `w04-proposal`. It will identify the reviewed submission version in the team repository.

## Quick start

### 1. Copy environment file

```bash
cp .env.example .env
```

### 2. Start the stack

```bash
docker compose up --build
```

### 3. Optional observability profile

```bash
docker compose --profile observability up --build
```

### 4. Open the relevant endpoints

- API docs: `http://localhost:8000/docs`
- AI service docs: `http://localhost:8100/docs`
- Prometheus: `http://localhost:9090` if the observability profile is enabled

## Typical development commands

### Start the stack

```bash
docker compose up --build
```

### Stop the stack

```bash
docker compose down --remove-orphans
```

### Stop and remove volumes

```bash
docker compose down -v --remove-orphans
```

### Run tests

```bash
pytest -q
```

### Run smoke tests against a running stack

```bash
SMOKE_BASE_URL=http://localhost:8000 pytest tests/smoke -q
```

### Run migrations

```bash
docker compose run --rm api alembic upgrade head
```

### Seed demo data

```bash
docker compose run --rm api python scripts/seed_demo_data.py
```

## What teams are expected to change

Teams are expected to extend or replace:

- the domain model
- routes and workflows
- worker job types
- AI-enabled logic
- tests
- documentation
- observability depth

Teams are not expected to replace the course operating model or ignore the required repository structure.

## Required living documents for team repositories

By the time this template becomes a semester team project repo, it should maintain:

- `README.md`
- `docs/architecture.md`
- `docs/operations.md`
- `docs/adrs/`
- `submissions/week04/`
- `submissions/week07/`
- `submissions/week13/`

## Required milestone tags for team projects

When this repo is used as a semester project repository, the required milestone tags are:

- `w04-proposal`
- `w07-midterm`
- `w13-final`

## Branch conventions

Recommended branch prefixes:

- `feature/...`
- `fix/...`
- `docs/...`
- `submission/...`

Default branch:

- `main`

## Testing expectations

A strong project should maintain some combination of:

- unit tests
- integration tests
- smoke tests
- credible end-to-end verification

The exact test mix may vary, but important behavior must be checkable and explainable.

## Observability expectations

At minimum, projects should support:

- structured logs
- health checks
- basic metrics or equivalent observability signals

Do not wait until the end of the semester to add operational visibility.

## Data and privacy expectations

Projects in this course must use only:

- public data
- synthetic data
- anonymized data
- instructor-approved sources

Do not commit secrets to the repository.

Do not log sensitive or unnecessary data carelessly.

## AI use reminder

If you use generative AI tools during development, you must remain able to explain your work and include the required AI Use Statement in major submissions.

The AI-enabled feature in your system is separate from optional AI tool use during development.

## Suggested first steps for teams

When beginning your project, do the following early:

- confirm the repo boots from a clean clone
- understand the current workflow end to end
- choose a narrow core workflow
- identify your main persistent entities
- identify your worker path
- decide where the AI-enabled function belongs
- update documentation as soon as design changes

## Final reminder

This course rewards engineering maturity.

A smaller, reliable, well-documented system built from this template is stronger than a large but fragile system built on rushed changes.
