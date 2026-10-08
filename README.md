# AI Startup Pitch Feedback Assistant

## Project Overview

The AI Startup Pitch Feedback Assistant helps early-stage founders and student entrepreneurs review the clarity and completeness of a short startup Pitch.

A founder submits a title and short Pitch text. The system stores the original submission, creates a background analysis Job, and uses an internal AI service to extract the problem, target customer, solution, and value proposition. It also summarises the idea, identifies missing or unclear information, and generates three improvement suggestions.

The founder reviews the feedback and decides whether it is relevant. The system does not assess investment potential or determine business viability.

## Project Documentation

- [Detailed system architecture](docs/architecture.md)
- [Week 4 proposal and project plan](submissions/week04/README.md)

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

## Group Members

- FEI, Wenxiang
- XIANG, Ke
- XU, Chao
- ZHANG, Mingyang
