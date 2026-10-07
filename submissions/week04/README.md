# W04 Team Architecture Proposal and Project Plan

## 1. Project Overview

- Team name: Project Group 15
- Project title: AI Startup Pitch Feedback Assistant
- Required submission tag: `w04-proposal`
- Architecture proposal: [docs/architecture.md](../../docs/architecture.md)

Our team plans to build an AI Startup Pitch Feedback Assistant for student entrepreneurs. The system will accept a short startup pitch and provide structured feedback to help users explain their business ideas more clearly.

This submission presents our proposed design and semester plan. It does not claim that the planned features have already been implemented or tested.

## 2. Problem and Primary User

Student entrepreneurs may have an initial business idea but struggle to explain the problem, target customer, proposed solution, and value proposition. They need clear feedback before presenting their ideas to teachers, classmates, competitions, or potential collaborators.

Our primary user is a student entrepreneur with a short written pitch.

The system will help the user identify missing or unclear information and improve the pitch. It will not determine whether the business will succeed or deserves investment.

## 3. Core Workflow

1. The user submits a project title and pitch description.
2. The API validates the input.
3. The system stores the original pitch and creates a background analysis job.
4. The user receives an ID and can check the processing status.
5. The worker calls the internal AI service.
6. The AI returns a summary, extracted business elements, missing information, and three improvement suggestions.
7. The worker saves the result and updates the job status.
8. The user retrieves the feedback and decides which suggestions to use, change, or ignore.

If analysis fails, the system will preserve the pitch, show a failed status, and allow a retry.

Human review remains part of the workflow. The system will not automatically rewrite or replace the original pitch.

## 4. Functional Requirements

| ID | Requirement | Evidence of completion |
| --- | --- | --- |
| F1 | Accept a project title and pitch description. | A valid submission returns a pitch ID. |
| F2 | Store the original pitch and processing status. | The submitted content and current status can be retrieved. |
| F3 | Create a background analysis job for each accepted pitch. | A job record is linked to the pitch and processed by the worker. |
| F4 | Generate structured pitch analysis. | The result contains the problem, target customer, solution, value proposition, and three improvement suggestions. |
| F5 | Store and expose the analysis result. | The user can retrieve the saved feedback through the API or a web interface. |

The result will also include a concise summary and identify missing or unclear information.

We will check failure handling by simulating an unavailable AI service and verifying that the user can retry the failed analysis.

## 5. Architecture Summary

We will extend the course starter repository using its four main components.

| Component | Planned responsibility |
| --- | --- |
| API service | Validate submissions, save pitches and jobs, and expose status, results, and retries. |
| PostgreSQL | Store original pitches, processing attempts, and analysis results. |
| Worker | Process queued jobs outside the submission request, call the AI service, and record outcomes. |
| Internal AI service | Extract key business elements and generate structured feedback. |

The API will return after saving the pitch and job. AI analysis will run in the worker so that the submission request does not wait for the result.

We will initially use one worker and a database-backed job queue. This keeps the system close to the starter and limits the number of services we need to manage.

Detailed component responsibilities, proposed API routes, and failure handling are described in [the architecture proposal](../../docs/architecture.md).

## 6. Data and Processing Plan

We plan to store three types of records:

- **Pitch:** title, original description, identifier, and submission time.
- **Processing job:** linked pitch, status, timestamps, and error information.
- **Analysis result:** linked job, summary, extracted business elements, missing information, and improvement suggestions.

A pitch can have multiple jobs when a failed analysis is retried. Each completed job will have one result.

We will use the job states `queued`, `processing`, `completed`, and `failed`. The API will report the latest job status for a pitch.

We will preserve original inputs and previous attempts so that a result can be traced back to the pitch and job that produced it.

## 7. Project Scope

### Required for the core workflow

- Text submission with a title and description.
- Persistent pitch and job records.
- Background AI analysis.
- Structured feedback with three improvement suggestions.
- Status and result retrieval.
- Failure reporting and user-triggered retry.
- Reproducible local deployment and verification.

### Optional after the core workflow works

- A simple web form for submission.
- A basic result page showing progress and feedback.

The API interface is sufficient for the initial demonstration if the web interface is not ready.

### Out of scope

- Investment prediction or recommendations.
- Commercial viability or financial profitability evaluation.
- Complete market research.
- Investor matching.
- A mobile application.

## 8. Proposed Semester Plan

The following schedule is our proposed implementation sequence. We will review progress regularly and keep the core workflow as the priority.

| Stage | Planned work | Completion evidence |
| --- | --- | --- |
| Week 4 | Agree on scope, architecture, data records, interfaces, and risks. | Proposal and architecture documents are reviewed and included in the submission version. |
| Weeks 5–6 | Confirm model access; implement pitch storage, job creation, status retrieval, and worker-to-AI communication. | A sample submission is saved and processed through the background workflow. |
| Week 7 | Demonstrate the minimum complete workflow. | Submit a pitch, retrieve structured AI feedback, and demonstrate a failed attempt and retry. |
| Weeks 8–10 | Improve output validation and failure recovery; review feedback quality using sample pitches. Add a simple web interface if time allows. | Recorded checks cover complete pitches, incomplete pitches, and service failures. |
| Weeks 11–12 | Stabilize deployment, tests, logs, and documentation. | Another team member can start the system from a clean checkout and follow the demo instructions. |
| Week 13 | Prepare the final demonstration and report limitations. | A reproducible demonstration and documented verification results are available. |

Our Week 7 minimum demonstration will include the problem, target customer, solution, value proposition, and three improvement suggestions. We will also retain the summary and missing-information feedback defined in our project scope.

## 9. Team Coordination

We will divide implementation work into four areas:
- API and database.
- Worker and AI integration.
- Verification and deployment.
- Documentation and demonstration.

Named owners will be agreed before implementation begins. Team members may cover more than one area, depending on team size.

Changes to the data model or API will be discussed with the members working on connected components. We will integrate changes regularly and update the documentation when a design decision changes.

Before each milestone, the team will review the complete workflow together rather than checking individual components only.

## 10. Risks, Assumptions, and Fallbacks

| Risk or assumption | Planned response | Fallback or scope cut |
| --- | --- | --- |
| Model access, cost, or quotas may prevent reliable analysis. | Confirm access and run a sample request during Weeks 5–6. | Select another accessible model. Use clearly labeled mock responses for engineering tests only; they do not count as a working AI demonstration. |
| AI output may omit required fields or include unsupported claims. | Validate the response structure and manually review sample outputs against the original pitch. | Keep failed validation visible and allow retry. Narrow the prompt to the required analysis fields. |
| AI requests may time out or the worker may stop. | Record failures, preserve input, and check for jobs stuck in processing. | Allow a new attempt without losing the original pitch or previous job record. |
| The project may expand into market research or investment evaluation. | Review new features against the agreed scope. | Remove features outside pitch clarity and completeness feedback. |
| Frontend work may delay the main workflow. | Build and verify the API workflow first. | Demonstrate through the API and defer interface polish. |
| Feedback quality may be difficult to judge consistently. | Use a small set of synthetic or approved pitches, including examples with missing business elements. | Report observed limitations and retain human review rather than claim objective business evaluation. |

We assume that a short text pitch provides enough information for useful communication feedback. When information is missing, the system should flag the gap rather than invent it.

## 11. Verification Plan

We will record the outcome of the following checks during implementation:

1. A valid title and description return a pitch ID and create linked database records.
2. Empty input is rejected before a job is created.
3. The submission request returns before AI analysis finishes.
4. The worker processes a queued job and saves a valid result.
5. The result includes the required business elements and three suggestions.
6. An incomplete pitch produces missing-information feedback.
7. A failed AI request produces a failed status and supports retry.
8. Pitches and results remain available after services restart.
9. A displayed result can be traced to its original pitch and processing job.

For feedback quality, team members will compare the AI output with the input and check whether the suggestions are relevant, understandable, and supported by the pitch.

These checks are planned. No test results are claimed in this proposal.

## 12. AI Use Statement

Generative AI assistance has been used to help prepare this milestone.

### Tools and purposes

- ChatGPT/Codex: helped interpret the assignment, organize the team's project notes, draft and revise the architecture proposal and project plan, and explain the GitHub submission steps.
- GitHub Copilot: suggested the architecture commit message and extended description shown in the GitHub interface.

### Human review record

Before the final team submission, we will record what the team actually checked, changed, or rejected:

- Verified by the team: [Add the requirements and design decisions the team checked against the project brief.]
- Changed by the team: [Add the revisions made after reviewing the AI-assisted draft.]
- Rejected or deferred by the team: [Add any suggestions the team did not adopt, or state “None” if accurate.]

AI-assisted drafting does not establish that the proposed system has been implemented or tested. The team is responsible for understanding the submitted design and reporting actual implementation and verification results.
