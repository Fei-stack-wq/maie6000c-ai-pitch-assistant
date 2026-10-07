# AI Startup Pitch Feedback Assistant — Architecture Proposal

## 1. Project Goal

Our team is building an AI Startup Pitch Feedback Assistant for student entrepreneurs.

Students may have a business idea but find it difficult to explain the problem, target customer, proposed solution, and value proposition clearly. Our system will accept a short pitch and return structured feedback that helps them improve their explanation before presenting it to teachers, classmates, or potential collaborators.

This document describes our planned architecture. We will build on the course starter repository and update the design as implementation progresses.

## 2. Scope

Users will submit a project title and a short pitch description.

The system will return:
- A short summary of the idea.
- The problem, target customer, proposed solution, and value proposition.
- Missing or unclear information.
- Three improvement suggestions.

Users will review the feedback and decide which suggestions to use, change, or ignore. The system will preserve the original pitch.

We will not build investment prediction, commercial viability evaluation, complete market research, investor matching, or a mobile application. Our goal is to improve how a business idea is communicated.

## 3. Main User Workflow

1. A student submits a project title and pitch description.
2. The API checks the input.
3. The system saves the pitch and creates a background analysis job.
4. The user receives a pitch ID and can check the processing status.
5. The worker sends the pitch to the AI service.
6. The AI service returns structured feedback.
7. The worker saves the result and marks the job as completed.
8. The student retrieves and reviews the feedback.

If the analysis fails, the system will show a failed status and allow the user to retry. The original pitch will remain available.

Results will be saved before they are displayed to the user.

## 4. System Architecture

We will retain the four main components of the course starter.

| Component | Responsibility |
| --- | --- |
| API service | Accept pitches, validate input, create records and jobs, and provide access to status and results. |
| PostgreSQL | Store submitted pitches, processing jobs, and analysis results. |
| Worker | Process queued jobs, call the AI service, and save results or failures. |
| Internal AI service | Extract the main business elements and generate concise feedback. |

The main processing path is:

Student → API → Database and queued job → Worker → AI service → Saved result → Student review

The API will return after saving the submission. It will not wait for the AI analysis to finish. This allows users to check progress while the worker handles the slower task.

For the initial demonstration, we can use the API interface. A simple web form can be added after the main workflow works.

## 5. Data and Persistence Plan

We plan to use three related tables.

| Table | Main information stored | Purpose |
| --- | --- | --- |
| pitches | Pitch ID, title, original description, submission time | Keep the original user input. |
| jobs | Job ID, pitch ID, status, start and finish times, error information | Track each analysis attempt. |
| analysis_results | Result ID, job ID, summary, extracted business elements, missing information, three suggestions, creation time | Keep the feedback returned by the AI. |

Relationships:
- Each job belongs to one pitch.
- A pitch can have more than one job if an analysis is retried.
- Each completed job has one analysis result.
- A failed job has no successful result.

We will use foreign keys to maintain these relationships. The pitch and its first job will be created in the same transaction so that an accepted submission is not left without a processing job.

The job record will be the source of processing status. The API will report the status of the latest job for a pitch.

Retries will create new job records rather than overwrite previous attempts. This allows us to trace a result back to the original pitch and understand what happened when an analysis failed.

## 6. API and Service Boundaries

The following endpoints are proposed for our project. They will replace or extend the starter's generic case workflow.

| Endpoint | Purpose |
| --- | --- |
| POST /pitches | Accept a title and description, save the pitch, and return the pitch ID and job ID. |
| GET /pitches/{pitch_id} | Return the original pitch, latest processing status, and feedback when available. |
| GET /jobs/{job_id} | Return the status of a specific analysis attempt. |
| POST /pitches/{pitch_id}/retry | Create a new analysis job after a failed attempt. |

The API will reject empty titles or descriptions. It will return a clear error when a requested record does not exist or a retry is requested while a job is still active.

The worker will call an internal analysis endpoint on the AI service. It will send the title and pitch description and receive structured feedback.

The AI service will not be called directly by users and will not write to the database. The worker will check and save its output.

## 7. Background Jobs and Failure Handling

We will use four job states:

- queued: waiting for the worker.
- processing: being analyzed.
- completed: the result has been saved.
- failed: the analysis could not be completed.

We will start with one worker and use the database to track pending work. This keeps the design close to the starter and avoids adding a separate queue service at this stage.

The worker will claim each job before processing it to avoid duplicate execution. It will apply a timeout when calling the AI service and check that the response contains the required fields.

If the AI service is unavailable, times out, or returns an invalid response, the worker will record the failure. The user can then retry without resubmitting the original pitch.

For the initial version, retries will be triggered by the user. We will also check for jobs left in processing after a worker interruption so that they do not remain stuck indefinitely.

## 8. AI Output and Human Review

The AI will analyze the submitted text and return a consistent structure containing:
- Summary.
- Problem.
- Target customer.
- Proposed solution.
- Value proposition.
- Missing or unclear information.
- Three improvement suggestions.

If the pitch does not explain an element, the AI should identify it as missing rather than invent information.

The worker will validate the response structure before saving it. We will also review sample outputs manually to check whether the feedback is relevant and grounded in the pitch. Correct formatting alone does not prove that the feedback is useful.

Users remain responsible for deciding whether to follow the suggestions. They can revise their pitch outside the system; a separate suggestion-editing feature is not required for the initial demonstration.

The model and provider have not yet been selected. We will confirm access, cost, and output quality before the Week 7 demonstration. Mock responses may be used to test the workflow, but they will be labeled and will not count as a working AI demonstration.

## 9. Quality Priorities and Verification

Our main quality priorities are reliability, response time, and traceability.

| Priority | Our approach | How we will check it |
| --- | --- | --- |
| Reliability | Save job status, record failures, and allow retries. | Test a successful analysis, an AI failure, and a retry. |
| Response time | Run analysis in the worker rather than inside the submission request. | Confirm that users receive an ID before analysis finishes and can check progress. |
| Traceability | Keep original pitches, job records, and saved results. | Retrieve a result and identify the input and job that produced it. |

We will also check that:
- Empty input is rejected.
- Each accepted pitch has a linked job.
- Results contain the required fields and three suggestions.
- Missing information is flagged.
- Saved pitches and results remain available after services restart.
- Failed analyses are not shown as completed.

These are planned checks. We will record actual results as the system is implemented.

## 10. Deployment and Operations

We will use the starter's Docker Compose setup to run the API, worker, AI service, and PostgreSQL locally. Database changes will be managed through Alembic migrations.

We will retain health checks and structured logs. Logs will include job IDs, state changes, and error categories so that we can investigate failures. We will monitor analysis duration and the number of completed and failed jobs.

Testing and demonstrations will use public, synthetic, anonymized, or instructor-approved inputs. Credentials will remain outside the repository, and application logs will avoid recording full pitch text.

## 11. Week 7 Demonstration

Our minimum Week 7 demonstration will show one complete workflow:

1. Submit a project title and short pitch.
2. Save the pitch and create a background job.
3. Process the pitch through the worker and AI service.
4. Retrieve structured feedback and three improvement suggestions.
5. Show processing status and demonstrate how a failed analysis can be retried.

We will prioritize this workflow before improving the interface. If time is limited, we will demonstrate it through the API.

The main risks are unavailable model access, inconsistent AI output, and spending too much time on extra features. We will address these by confirming model access early, testing with a small set of sample pitches, and keeping the agreed scope.
