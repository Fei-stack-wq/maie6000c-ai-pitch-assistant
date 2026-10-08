# AI Startup Pitch Readiness Assistant

## Project Overview

The AI Startup Pitch Readiness Assistant helps early-stage founders and student entrepreneurs review the clarity and completeness of a short startup pitch.

A founder submits a title and pitch text. The system stores the original submission, creates a background analysis job, and uses an internal AI service to extract the problem, target customer, proposed solution, and value proposition. It also produces a concise summary, identifies missing or unclear information, and generates three improvement suggestions.

The founder remains responsible for deciding whether the feedback is relevant. The system does not assess investment potential or determine whether a business is commercially viable.

## Primary Workflow

1. The founder submits a title and short pitch through the API.
2. The API validates and stores the original pitch and creates a linked analysis job.
3. A worker claims the job and sends the stored pitch to the internal AI service.
4. The worker validates and persists the structured analysis result.
5. The founder retrieves the processing status and reviews the feedback through the API.

If the AI request fails or returns invalid output, the worker records the failure and marks the job and pitch as failed.

## Main Components

- **API service**: validates submissions, creates pitch and job records, and exposes statuses and results.
- **PostgreSQL**: stores pitches, analysis jobs, analysis results, and related metadata.
- **Worker**: processes analysis jobs outside the submission request and coordinates AI calls and database updates.
- **Internal AI service**: returns structured analysis and feedback for the submitted pitch.
- **Docker Compose**: runs the API, database, worker, and AI service in a reproducible local environment.

## Scope Boundary

The initial system focuses on one complete submission-to-feedback workflow. It does not evaluate investment potential, determine commercial viability, perform complete market research, or provide formal legal or business advice.

## Project Documentation

- [Detailed system architecture](docs/architecture.md)
- [Week 4 proposal and project plan](submissions/week04/README.md)

## Setup and Run

The system is designed to run locally with Docker Compose. Make sure Docker Desktop and Docker Compose are available, then start the services with:

```bash
docker compose up --build
```

To stop the services:

```bash
docker compose down
```

## Team Members

- FEI, Wenxiang
- XIANG, Ke
- XU, Chao
- ZHANG, Mingyang
