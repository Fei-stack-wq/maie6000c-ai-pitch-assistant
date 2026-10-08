# W04 Team Architecture Proposal and Project Plan

## 1. Project Overview

- **Team:** Project Group 15
- **Project Title:** AI Startup Pitch Feedback Assistant
- **Submission Tag:** `w04-proposal`

Our team proposes an AI Startup Pitch Feedback Assistant
designed to help student entrepreneurs improve how they
present their business ideas.

Users will submit a short startup pitch, and the system
will provide structured feedback covering four key elements:

- Problem
- Target Customer
- Solution
- Value Proposition

The system will also identify missing or unclear information
and provide three actionable improvement suggestions.

Our goal is to help students communicate their ideas more
clearly, rather than predict business success or provide
investment recommendations.

This Week 4 submission describes our proposed architecture,
implementation plan, milestones, and major risks. The system
has not yet been fully implemented or tested.


## 2. Primary Workflow and Architecture

### Primary Workflow

The proposed workflow consists of the following steps:

1. The user submits a project title and a short startup pitch.
2. The API validates the input and stores the original pitch.
3. The system creates a background analysis job.
4. A worker retrieves the job and calls the internal AI service.
5. The AI analyzes the pitch and generates structured feedback.
6. The worker saves the analysis result and updates the job status.
7. The user retrieves the feedback through the API or web interface.

The generated feedback will include:

- A concise pitch summary.
- The four key business elements.
- Missing or unclear information.
- Three actionable improvement suggestions.

If the AI analysis fails, the system will preserve the
original pitch, report the failure, and support retrying.

Users will review the generated feedback and decide which
suggestions to accept, modify, or ignore.

### Architecture Summary

The system will extend the course starter repository
using four main components:

| Component | Responsibility |
| --- | --- |
| API Service | Validate submissions, store pitches, and expose status and results. |
| PostgreSQL | Store pitches, processing jobs, and analysis results. |
| Worker | Process background jobs and handle AI requests. |
| Internal AI Service | Analyze startup pitches and generate structured feedback. |

We will initially use one worker and a database-backed
job queue to keep the architecture simple and manageable.

The API will return after creating the analysis job,
while AI processing will run asynchronously.

Detailed component responsibilities, API design,
data models, and failure handling are described in
the architecture document.


## 3. Milestone Plan

Our team plans to complete the project through the
following milestones:

| Timeline | Planned Work | Expected Outcome |
| --- | --- | --- |
| Week 4 | Define project scope, architecture, interfaces, and risks. | Completed proposal and architecture documents. |
| Weeks 5–6 | Implement pitch storage, job creation, status retrieval, and AI integration. | Basic background processing workflow. |
| Week 7 | Complete the minimum end-to-end workflow. | Submit a pitch, retrieve AI feedback, and demonstrate failure handling and retry. |
| Weeks 8–10 | Improve output validation, feedback quality, and reliability. | Verified results using representative pitch examples. |
| Weeks 11–12 | Improve testing, deployment, and documentation. | Reproducible deployment and stable core workflow. |
| Week 13 | Prepare the final demonstration and report. | Final demonstration and documented results. |

### Minimum Viable Product (MVP)

By Week 7, we aim to demonstrate:

- Pitch submission and persistent storage.
- Background AI analysis.
- Structured feedback covering the four business elements.
- Three improvement suggestions.
- Status and result retrieval.
- Basic failure handling and retry.

Our priority is to complete a reliable end-to-end workflow
before adding optional features.


## 4. Key Risks and Scope Management

### Core Scope

The project will focus on:

- Text-based startup pitch submission.
- Persistent storage and background processing.
- AI-generated structured pitch feedback.
- Missing-information identification.
- Three actionable improvement suggestions.
- Status retrieval, error handling, and retry.

### Optional Features

If time permits, we may add:

- A simple web interface for pitch submission.
- A result page displaying feedback and processing status.

The API workflow will be sufficient for the initial
demonstration if the web interface is not completed.

### Out of Scope

The following features are excluded:

- Investment prediction or financial recommendations.
- Commercial profitability evaluation.
- Comprehensive market research.
- Investor matching.
- Mobile application development.

### Key Risks and Mitigation

| Risk | Mitigation | Fallback |
| --- | --- | --- |
| AI model access or API limitations | Confirm model availability and test access early. | Use another accessible model. Mock responses may be used for engineering tests but not as evidence of working AI analysis. |
| Incomplete or inaccurate AI feedback | Validate output structure and review sample results. | Narrow the prompt and report missing information rather than inventing details. |
| AI service failures or timeouts | Track job status and preserve original submissions. | Support retry without losing the original pitch. |
| Implementation delays | Prioritize the core backend and AI workflow. | Defer frontend development and optional features. |
| Difficulty evaluating feedback quality | Review representative pitch examples against the original inputs. | Document limitations and retain human review. |

If development time becomes limited, we will prioritize
the API, background processing, and structured feedback
generation over frontend development and optional features.


## 5. Artifact Index

The following documents support this Week 4 submission:

| Artifact | Description |
| --- | --- |
| [Architecture Proposal](../../docs/architecture.md) | Detailed system architecture, component responsibilities, API design, data models, and failure handling. |
| [Week 4 README](README.md) | Project overview, workflow, milestone plan, scope, risks, and AI use statement. |

The architecture document provides the technical details
supporting this concise submission summary.


## 6. Team Coordination

Our team plans to organize implementation work into
four main areas:

1. API and database development.
2. Worker and AI service integration.
3. Testing, verification, and deployment.
4. Documentation and final demonstration.

Individual responsibilities will be confirmed before
implementation begins.

Team members will coordinate changes to shared APIs
and data models, integrate their work regularly,
and review the complete workflow before each milestone.


## 7. AI Use Statement

Generative AI tools were used to assist with the
preparation of this Week 4 milestone.

### Tools and Purposes

- **ChatGPT/Codex:** Assisted with interpreting assignment
  requirements, organizing project notes, and revising
  the architecture proposal and project plan.

- **GitHub Copilot:** Assisted with suggesting an
  architecture-related commit message and description.

### Human Review and Responsibility

AI-assisted content was reviewed and revised during
proposal preparation.

The revisions focused on:

- Clarifying the project objectives and requirements.
- Simplifying the architecture and implementation plan.
- Distinguishing proposed features from completed work.
- Keeping the project scope realistic and manageable.

A full team review is still pending.

Before submission, the team will verify the proposed
architecture, database structure, API interfaces,
milestone schedule, and project scope.

The model provider and individual task owners
also remain to be confirmed.

The team remains responsible for the accuracy,
feasibility, and final submission of this proposal.

This document describes planned work and does not
claim that the proposed system has already been
implemented or tested.
