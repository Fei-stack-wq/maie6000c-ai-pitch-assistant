# Problem and Primary User

Early-stage founders and student entrepreneurs may struggle to explain their startup ideas clearly and identify missing information in a short Pitch.
The assistant helps them review the clarity and completeness of their Pitch. It extracts the problem, target customer, solution, and value proposition, summarises the idea, and provides three improvement suggestions.
The primary user is a founder preparing or revising a short startup Pitch. The founder remains responsible for deciding whether the feedback is relevant. The system does not assess investment potential or determine business viability.

# Primary Workflow

1. The founder submits a title and short Pitch text through the API.
2. The API trims whitespace and validates the input. Initially, the proposed limits are 3-200 characters for the title and 5-4,000 characters for the Pitch text.
3. In one database transaction, the API stores the original Pitch with status queued and creates a linked analysis Job with status pending.
4. The API returns the Pitch ID, Job ID, and initial statuses without waiting for AI analysis.
5. The Worker claims the Job, records the claim time, increments the attempt count, and reads the stored Pitch.
6. The Worker calls the internal AI service to extract Pitch elements, summarise the idea, identify missing information, and generate three suggestions.
7. The Worker validates the output and stores the result. The result and successful status updates are committed together.
8. The founder retrieves the current status and available feedback through the API and reviews the suggestions.

If the AI request fails or its output is invalid, the Worker records the error and marks the Job and Pitch as failed. A failed analysis does not produce a successful result record.

# Components and Architecture Flow

### Components

- API service: Validates submissions, creates Pitch and Job records, and retrieves stored statuses and results.
- PostgreSQL: Stores original inputs, processing Jobs, analysis results, and related metadata.
- Worker: Polls pending Jobs and performs analysis outside the submission request. It coordinates AI calls, output validation, and database updates.
- Internal AI service: Accepts Pitch content and uses a pretrained language model to produce structured feedback. The model/provider will be selected according to available access.
- Docker Compose: Runs the API, database, Worker, and internal AI service in a reproducible local environment.

### Architecture Flow

Founder → API: submit a title and Pitch text.
API → PostgreSQL: store the Pitch and create the linked Job.
Worker ↔ PostgreSQL: claim a pending Job and read its Pitch.
Worker ↔ Internal AI service: request analysis and receive structured output.
Worker → PostgreSQL: store the result and status updates, or record failure.
Founder → API → PostgreSQL → API → Founder: retrieve the current status and feedback.

# Persistent Entities and Schema/ERD

The proposed database contains three core tables:

### pitches

- Table: pitches.
- Primary key: id, a UUID string.
- Important fields: title, pitch_text, status, created_at, and updated_at.
- Foreign keys: None in the initial design.
- Status/lifecycle: Created as queued; changes to completed after successful analysis or failed after processing failure. The original Pitch text remains unchanged; revised text is stored as a new submission.

### jobs

- Table: jobs.
- Primary key: id, an auto-incrementing integer.
- Important fields: job_type, status, attempts, error, created_at, claimed_at, completed_at, and updated_at.
- Foreign keys: pitch_id → pitches.id, non-null.
- Status/lifecycle: pending → claimed → completed/failed. Claiming records the claim time and increments attempts. Completion or failure records the processing end time; failure also records an error.

### analysis_results

- Table: analysis_results.
- Primary key: id, an auto-incrementing integer.
- Important fields: summary, problem, target_customer, solution, value_proposition, missing_information, suggestions, model_identifier, prompt_version, and created_at.
- Foreign keys: job_id → jobs.id, unique and non-null.
- Status/lifecycle: Created only after successful analysis and output validation. No separate processing status is required. The result is retained without being overwritten by future analyses.

# State Transitions

The proposed state transitions are:

- Pitch: queued → completed or queued → failed.
- Job: pending → claimed → completed/failed.

The API creates the initial states. The Worker changes a pending Job to claimed, records claimed_at, and increments attempts. Claiming a Job does not change the Pitch status; the Pitch remains queued until analysis finishes.
On success, the Worker stores the result and updates the Job and Pitch to completed in one transaction. On failure, it records the Job error and marks both records as failed. The Job's completed_at records when processing ends, whether successfully or unsuccessfully.

# API/Service Boundaries

- POST /pitches: Validates input, stores the Pitch, and creates an analysis Job. Returns 201 with Pitch ID, Job ID, and initial statuses.
- GET /pitches/{pitch_id}: Returns the stored Pitch, current status, and available feedback. The result is absent while processing is pending or after failure.
- GET /jobs/{job_id}: Returns Job status, attempts, processing timestamps, and a safe error message if processing failed.
- Internal POST /analyse-pitch: Receives Pitch content from the Worker and returns structured analysis for validation.
- /health/live and /health/ready: Retain service liveness and readiness checks.
  Invalid input receives a validation error. Unknown Pitch or Job IDs return 404.

# Worker and AI-Enabled Processing Plan

### Worker Plan

- Finds the oldest pending Job.
- Marks it as claimed and records the processing attempt.
- Reads the linked Pitch from PostgreSQL.
- Calls the internal AI service with a configured timeout.
- Validates the returned structure and required fields.
- Stores the result and successful status changes in one transaction, or records processing failure.

### AI Input and Output

The AI receives the stored title and Pitch text. Its output includes:

- A short idea summary.
- Problem.
- Target customer.
- Solution.
- Value proposition.
- Missing or unclear information.
- Exactly three improvement suggestions.

# Failure, Retry, and Review Behavior

### Failure Handling

AI timeouts, service errors, and invalid outputs are treated as processing failures. The Worker stores an error on the Job and marks the Job and Pitch as failed.
The API returns a clear failure status without exposing credentials or internal stack traces. The system does not substitute invented feedback for failed analysis.
If a database failure prevents status updates, the Worker logs the event for investigation. If the Worker stops after claiming a Job, that Job may remain claimed; automatic recovery is outside the initial scope.

### Retry Policy

The initial version does not automatically retry failed Jobs. A user may submit the Pitch again as a new submission.
If controlled retry or repeated analysis is added later, it will create a new Job while preserving previous Jobs and results. The interface and status-selection rules will be defined before enabling that feature.

### Human Review and Data Handling

The founder reviews feedback before acting on it and may ignore or revise suggestions. A formal approval system and stored review records are outside the initial scope.

# Risks and Scope Cuts

- AI output quality: Output may be invalid or unsupported by the Pitch. We will validate its structure, explicitly flag missing information, and review representative outputs.
- External service reliability: AI access, latency, or availability may prevent analysis. We will confirm access early, configure timeouts, and expose recorded failures.
- Integration consistency: API, schema, Worker, and AI output contracts may become inconsistent. We will agree on shared contracts and verify changes through integration checks.
- Worker interruption: A claimed Job may remain unfinished after the Worker stops. We will document manual investigation and defer automatic recovery.
- Project scope: Optional features may delay the core workflow. We will prioritise one complete submission-to-feedback workflow.

The first scope cuts are dashboards, repeated-analysis interfaces, result comparisons, automatic retries, and stored review records.
The protected core is submission, relational persistence, background AI analysis, output validation, failure visibility, and result retrieval.
