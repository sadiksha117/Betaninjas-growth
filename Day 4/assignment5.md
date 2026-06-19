# Assignment 2 - Design Tests Like a Pro

Target: Task submission feature - BetaninjasSEOLMS

Code observed for this assignment:
- Member submission route: app/api/progress/route.ts (GET and POST, POST upserts to the Progress table)
- Admin timeline route: app/api/admin/timeline/[phaseId]/route.ts (PUT, saves startDate and dueDate)
- Prisma schema: Progress.response is String? with no length limit; TaskType is CHECKBOX, TEXT, LINK; TaskStatus is NOT_STARTED, IN_PROGRESS, SUBMITTED, COMPLETED

Validation schemas (verified):
- TimelineUpdateSchema requires both dates in datetime format and has a refine rule: dueDate must be strictly greater than startDate ("Due date must be after start date").
- progressSchema: taskId is a positive integer, status is one of the four enum values, response is an optional string with no max length, link is an optional valid URL.

Two findings from reading the code shape this whole assignment:
1. The progress POST does not enforce status transitions. It upserts whatever status the schema accepts, with no check against the previous status. So a TEXT task can jump straight from NOT_STARTED to COMPLETED.
2. The response field has no length limit defined anywhere (no max in progressSchema, no length constraint in the Prisma model), so the text boundary is effectively unbounded.

## Step 1 - Boundary Value Analysis

### Date fields (Start Date and Due Date on /admin/timeline)

Rule under test: Due Date must be after Start Date. The route saves both dates directly, so any enforcement of this rule lives in TimelineUpdateSchema. The cases below test at and around the equality boundary.

BV-01 - Due date one day after start date (just inside valid)
Data: start = 2026-06-01, due = 2026-06-02
Expected: Accepted and saved.

BV-02 - Due date equal to start date (the exact boundary)
Data: start = 2026-06-01, due = 2026-06-01
Expected: Rejected with "Due date must be after start date". Confirmed by the refine rule, which uses strict greater-than, so equal dates fail. This is the boundary that proves the rule is "strictly after", not "on or after".

BV-03 - Due date one day before start date (just inside invalid)
Data: start = 2026-06-02, due = 2026-06-01
Expected: Rejected with a validation error.

BV-04 - Both fields empty
Data: start = empty, due = empty
Expected: Rejected. The schema should require both fields; the route would return a 400 validation error.

BV-05 - One field empty
Data: start = 2026-06-01, due = empty
Expected: Rejected with a validation error for the missing field.

### Character limit on task text submission

Finding: There is no character limit defined anywhere. Confirmed in two places: progressSchema declares response as z.string().optional() with no .max(), and the Prisma schema types Progress.response as String? with no length constraint, which maps to unbounded text in PostgreSQL.

What the boundary is: undefined. Since neither the schema nor the model sets a limit, the only practical ceiling is the database column size and the request body size, not a deliberate validation boundary.

What happens beyond it: with no application limit, a very large submission (for example tens of thousands of characters) would be accepted and stored, which risks slow queries, broken layouts when the text is rendered, and larger payloads. The recommendation is to define an explicit max length in progressSchema so the boundary is known and tested rather than discovered in production.

## Step 2 - Equivalence Partitioning

The task submission form sends taskId, status, and optionally response and link to the progress POST route, which validates with progressSchema. Inputs group into classes that behave the same way, so one representative test per class covers the class.

EP-01 - Valid input
A real taskId, a valid status enum value, and an appropriate response or link for the task type.
Expected: 200, Progress upserted with the given values.

EP-02 - Invalid input
A taskId that is not a number, or a status that is not one of the four enum values.
Expected: 400 validation error from safeParse.

EP-03 - Empty input
Required fields missing (for example no taskId or no status).
Expected: 400 validation error.

EP-04 - Unexpected type
Wrong data types, for example status sent as a number or taskId sent as an object.
Expected: 400 validation error from safeParse.

EP-05 - Invalid link format
A link value that is not a valid URL, for example "notaurl".
Expected: 400 validation error. progressSchema requires link to pass z.string().url(), so a malformed link is rejected while a valid URL is accepted.

Note: these tests cover the same ground that testing dozens of individual values would, because every value inside a class is handled identically by the validation layer. One representative per class is enough.

## Step 3 - Decision Table (progress POST)

Four conditions: user authenticated, task ID valid, status is a valid enum, text field non-empty. Authentication is checked first, so any unauthenticated request returns 401 before anything else runs, which collapses all unauthenticated rows into one outcome.

Conditions key: Auth = authenticated, ID = valid task ID, Status = valid enum, Text = text field non-empty.

| Row | Auth | ID | Status | Text | Expected outcome |
|-----|------|----|--------|------|------------------|
| 1 | N | any | any | any | 401 Unauthorized (short-circuits before validation) |
| 2 | Y | N | any | any | 400 validation error (schema rejects invalid taskId) |
| 3 | Y | Y | N | any | 400 validation error (status not a valid enum) |
| 4 | Y | Y | Y | N | 200, upsert succeeds (text is optional in the model) |
| 5 | Y | Y | Y | Y | 200, upsert succeeds with response saved |

Test case per row:
- TC-DT-1: Send any body with no session. Expect 401.
- TC-DT-2: Authenticated, taskId = "abc" (not a number). Expect 400.
- TC-DT-3: Authenticated, valid taskId, status = "DONE" (not in enum). Expect 400.
- TC-DT-4: Authenticated, valid taskId, status = IN_PROGRESS, no response text. Expect 200, progress updated.
- TC-DT-5: Authenticated, valid taskId, status = SUBMITTED, response = "my answer". Expect 200, response saved.

Note: rows 4 and 5 both pass because response is optional in the schema and model. If the business rule is that a TEXT task must include text before being SUBMITTED, that rule is not enforced by this route and would be a finding.

## Step 4 - State Transition

Task status has four states: NOT_STARTED, IN_PROGRESS, SUBMITTED, COMPLETED.

### CHECKBOX tasks (toggle between two states)

    NOT_STARTED  <-->  COMPLETED

Only two states are used. Checking the box moves NOT_STARTED to COMPLETED; unchecking moves COMPLETED back to NOT_STARTED.

### TEXT and LINK tasks (all four states)

    NOT_STARTED --> IN_PROGRESS --> SUBMITTED --> COMPLETED

The intended flow is start work, submit a response or link, then mark complete.

### Valid transition test cases

ST-01 - CHECKBOX: NOT_STARTED to COMPLETED. Expect 200, completedAt set.
ST-02 - CHECKBOX: COMPLETED back to NOT_STARTED. Expect 200, completedAt cleared to null.
ST-03 - TEXT: NOT_STARTED to IN_PROGRESS. Expect 200.
ST-04 - TEXT: IN_PROGRESS to SUBMITTED with a response. Expect 200, response saved.
ST-05 - TEXT: SUBMITTED to COMPLETED. Expect 200, completedAt set.

### Invalid transition test cases

ST-06 - TEXT: NOT_STARTED straight to COMPLETED, skipping IN_PROGRESS and SUBMITTED.
Key finding: the progress route does not enforce transition order. It upserts whatever valid status is sent, so this jump is currently accepted by the API even though the intended flow has middle states. Whether it should be allowed is a design question. For TEXT and LINK tasks it should arguably be blocked, because completing without submitting a response or link defeats the purpose of those task types.

ST-07 - CHECKBOX: NOT_STARTED to SUBMITTED.
A checkbox task only has two valid states, but the enum allows SUBMITTED, and the route does not restrict status by task type. So this invalid-for-type transition is also accepted by the API. This is a second transition finding: status values are not validated against the task's type.

## Step 5 - Testing Levels

Unit: Test progressSchema (and TimelineUpdateSchema) in isolation, confirming they accept valid input and reject invalid status values, bad types, and missing fields.

Integration: Test the progress POST route together with the database, confirming the upsert creates or updates the correct Progress row and that completedAt is set or cleared based on status.

System: Test the full submission flow in the browser end to end, from a logged-in member opening a task, entering a response, submitting, and seeing the status update reflected in the UI.

Acceptance: Confirm the feature meets the business requirement, that a member can record progress on their assigned tasks and reach COMPLETED, and that an admin can set phase start and due dates that drive the timeline.