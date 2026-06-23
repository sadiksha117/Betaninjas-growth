# Phase 3 Assignment 1 - Map the Coverage

Target: BetaninjasSEOLMS - everything from Phase 1 and Phase 2

This assignment maps every coverage concept onto the test cases and code already worked through in Phase 1 and Phase 2.

## Step 1 - The Testing Pyramid

### Categorizing the Phase 1 and Phase 2 test cases

Phase 1 (Day 3, login feature):
- TC-01 valid login - System
- TC-02 cancel Google flow - System
- TC-03 protected page while logged out - System (middleware behaviour end to end)
- TC-04 new Google account creates member - System
- TC-05 session persists on refresh - System
- GB-01 role assigned only on first sign-in - Integration (auth callback plus database)
- GB-02 login and protected route redirects - Integration (middleware plus session)
- White box getDaysRemaining edge cases - Unit
- Step 4 testing types for login - Acceptance (requirement-level reasoning)

Phase 2 (Day 4, task submission feature):
- BV-01 to BV-05 date boundaries - Integration (timeline route plus validation schema)
- Character limit finding - Unit (validation schema on response)
- EP-01 to EP-05 equivalence classes - Integration (progress route plus schema)
- DT-1 to DT-5 decision table rows - Integration (progress route plus auth plus schema)
- ST-01 to ST-07 state transitions - Integration (progress route plus database upsert)
- Step 5 testing levels mapping - Acceptance

### Count by category

- Unit: about 3 (getDaysRemaining edge cases, character-limit schema check)
- Integration: about 18 (most of Phase 2: boundaries, equivalence, decision table, transitions, plus the two grey box login cases)
- System: 5 (the five black box login cases)
- Acceptance: 2 (the testing-types and testing-levels reasoning steps)

### The shape

The current shape is not a clean pyramid. It bulges in the middle at the integration level and is thin at the unit base, with a moderate system layer on top. It looks more like a diamond (fat middle, narrow top and bottom) than a pyramid. This makes sense, because the testing so far was exploratory and design-technique driven against routes and the live app, not unit tests written against individual functions.

### Ideal pyramid for BetaninjasSEOLMS

- Base (about 70 percent) Unit: the pure functions in lib/utils have no external dependencies and are cheap to test. getDaysRemaining, getPhaseStatus, isDone, calcPhaseProgress, calcOverallProgress, and isPhaseUnlocked should each have several unit tests covering their branches.
- Middle (about 20 percent) Integration: the API routes (progress, tasks, timeline) tested together with the validation schemas and the database, confirming auth gating, validation, and the correct database writes.
- Top (about 10 percent) End to end: a few full-journey tests such as log in, open a phase, submit a task, see progress update.

What is currently missing at each level:
- Unit: almost entirely missing. The lib/utils functions were analyzed by reading but no automated unit tests exist.
- Integration: covered by manual and design-technique test cases, but not automated.
- System / E2E: covered manually on the live app, but no automated end-to-end suite exists.

## Step 2 - Line, Branch, and Path Coverage of getDaysRemaining

```typescript
export function getDaysRemaining(dueDate: Date, now: Date = new Date()): number {
  const diff = dueDate.getTime() - now.getTime()
  return Math.ceil(diff / (1000 * 60 * 60 * 24))
}
```

Line coverage: calling the function once with any valid date executes both statements, the diff line and the return line. So a single test hits 100 percent line coverage. But executing the lines does not check that the returned number is correct, which is the weakness of line coverage.

Branch coverage: this function has no if/else statements, so there are zero explicit branches. By the strict definition, one test gives full branch coverage because there are no branches to split on. The meaningful behaviour, though, comes from the value ranges (positive, zero, negative) created by Math.ceil, not from code branches. So branch coverage looks complete with one test while still missing the important cases. This is the key lesson: a function can have full branch coverage and still be undertested.

Path coverage: a path is a complete route from entry to exit. With no branches, there is exactly one structural path through the code, so one test gives full structural path coverage. Path coverage costs more than branch coverage in general because in functions with multiple branches the number of paths multiplies (every combination of branch outcomes is its own path), while branches only need each individual condition exercised. Here both happen to be one because the function is straight-line, but the meaningful input partitions (future date, today, past date) still need three tests to actually verify behaviour.

Difference in my own words, using this function:
- Line coverage asks did this line run. One test makes both lines run, so it says 100 percent while never checking a single result.
- Branch coverage asks did every if/else direction run. This function has none, so it is trivially full, which shows branch coverage can be blind to value-based behaviour.
- Path coverage asks did every full route run. Straight-line code has one route, so again one test. The real risk lives in the input values, not the structure, which is why coverage numbers can look perfect while bugs (a negative days-remaining for a past date) go unverified.

For getPhaseStatus, the contrast is sharper. It has multiple if conditions (progress equals 100, now before start, now after due with progress under 100, then the on-track versus behind comparison). Branch coverage there needs several tests to take each condition both true and false, and path coverage needs more still to cover the combinations, which is exactly why getPhaseStatus is the function that deserves the most unit tests.

## Step 3 - Function Coverage

Utility functions found in lib/utils:

timeline.ts
- getPhaseStatus
- getDaysRemaining

progress.ts
- isDone (private helper, not exported)
- calcPhaseProgress
- calcOverallProgress
- isPhaseUnlocked

Which functions had test cases in Phase 2:
- getDaysRemaining: covered. It was analyzed in Phase 2 (white box analysis with three edge cases). Coverage by reading, not by automated test, but it was the one function deliberately exercised.

Which functions have zero coverage:
- getPhaseStatus: not tested, only referenced. It is the most logic-heavy function and most deserves tests.
- isDone: not tested.
- calcPhaseProgress: not tested.
- calcOverallProgress: not tested.
- isPhaseUnlocked: not tested, and critically it currently returns true unconditionally with the real logic commented out.

Function coverage report:
Of six utility functions, only one (getDaysRemaining) has received any deliberate test attention, and even that was analysis rather than an automated test. Function coverage is about 1 of 6, roughly 17 percent. The five untested functions include the two most important ones for correctness: getPhaseStatus (drives every status badge) and isPhaseUnlocked (controls phase gating, and is currently broken). This is the clearest coverage gap in the project.

## Step 4 - Requirement Coverage

Requirements drawn from the spec (sections 1.2, 4, and 12) for the task submission and progress feature, mapped to Phase 2 Assignment 2 test cases.

| Requirement (from spec) | Test case in Phase 2 | Status |
|---|---|---|
| Member can submit evidence per task (tick, text, or link) | EP-01 valid input, DT-4 and DT-5 | Covered |
| CHECKBOX sets status to Completed, can be unticked back | ST-01 and ST-02 | Covered |
| TEXT submit sets status to Submitted, response saved | ST-04, DT-5 | Covered |
| LINK validates a valid URL before saving | EP-05 invalid link rejected | Covered |
| Task status moves Not started to In progress to Submitted to Completed (correct order) | ST-03 to ST-06 plus the finding that order is not enforced | Partially Covered (tested, and found not enforced) |
| Submission requires authentication | DT-1 unauthenticated returns 401 | Covered |
| Phases unlock only when previous phase is 100 percent complete; locked phases cannot be entered | none | Not Covered |
| Phase progress percent updates after submission | none directly (calcPhaseProgress untested) | Not Covered |
| Overall progress percent across all phases | none directly (calcOverallProgress untested) | Not Covered |
| Submitted response or link shown back on reopening the task | none | Not Covered |
| Member can edit and resubmit at any time | implied by upsert, not explicitly tested | Partially Covered |

Requirement coverage gap (the Not Covered items):
1. Sequential phase gating. The spec clearly requires phases to lock until the previous is fully complete, and that locked phases cannot be entered. No test covers this, and the code (isPhaseUnlocked returning true) actively violates it.
2. Phase and overall progress calculation. calcPhaseProgress and calcOverallProgress have no tests, so the requirement that progress percentages are correct is unverified.
3. Persistence on reopen. No test confirms a saved response or link is shown again when the task is reopened.

## Step 5 - Risk-Based Coverage

| Feature | Risk | Reasoning |
|---|---|---|
| Login / auth | High | If it breaks, no one can access anything, and a flaw could expose another user's data or admin functions. It is the gate to the whole app. |
| Task submission / progress saving | High | This is where user data is written. If a submission is lost or saved to the wrong user, the member loses real work and the records become untrustworthy. |
| Progress tracking (calc functions) | High | If progress percentages are wrong, the core promise of the product (showing where you are) fails, and phase unlocking depends on it. |
| Phase navigation / gating | Medium to High | The spec treats sequential gating as core. It is currently broken (unlock-all), which is medium risk for a small internal team but high against the spec. |
| Dashboard | Medium | A display problem (like the completed-and-overdue bug) reduces trust but does not lose data or block access. |
| Admin view (team table, export, reminders) | Medium | Important for admins but used by few people; failure is visible but not data-destroying for members. |
| Glossary | Low | Read-only reference content; failure is a minor inconvenience. |

Are the high-risk features covered the most? Honest answer: no. The coverage so far is heaviest on task submission (good, that is high risk) and login (good). But progress tracking and phase gating, both high risk, have almost no coverage. calcPhaseProgress, calcOverallProgress, and isPhaseUnlocked are exactly the high-risk functions with zero tests. So the biggest gap between risk and coverage is in the progress and gating logic: high importance, near-zero verification.

## Step 6 - Coverage vs Confidence

Kristi's bug from Phase 1: Phase 2 was accessible without completing Phase 1. In the code this is visible directly: isPhaseUnlocked has `return true // for testing, unlock all phases` with the real gating logic commented out beneath it.

Would a test that calls the phase navigation function and passes mean the bug was caught? No. A test could call isPhaseUnlocked, the function would execute its single return line, and the test would pass. That gives 100 percent line and function coverage for that function while completely missing the bug, because the function does run, it just returns the wrong answer.

Why line or function coverage alone would not catch it:
Line coverage only confirms the return statement executed. Function coverage only confirms the function was called at least once. Neither checks what the function returned. The bug is not an unexecuted line; it is a correctly executed line that produces the wrong result. Coverage measures execution, not correctness.

The specific assertion needed to catch it:
A test would need to set up a scenario where the previous phase is not complete and then assert that the function returns false. For example: given phase 2, where phase 1 has incomplete tasks in the progress map, assert that isPhaseUnlocked(2, phases, progressMap) is false. With the current code that assertion fails, because the function returns true. That failing assertion is what catches the bug. A second assertion (previous phase complete, expect true) confirms the gating works in the positive case too.

The difference between a test that runs code and a test that proves something:
A test that runs code just calls the function and finishes without checking the output, so it turns green as long as nothing throws. A test that proves something makes a specific claim about the result (the unlocked phase must be false when the prior phase is incomplete) and fails when that claim is untrue. Real confidence comes from coverage plus meaningful assertions plus thinking about which results actually matter for the risk. Coverage tells you the code ran; assertions tell you it was right.