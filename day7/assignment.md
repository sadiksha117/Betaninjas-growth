# Phase 3 Assignment 2 - Coverage Strategy and Mobile Thinking

Target: BetaninjasSEOLMS - thinking beyond what was tested to how it would be tested

This builds on the Day 1 risk matrix, requirement coverage map, and function coverage report.

## Step 1 - Coverage Strategy

A coverage strategy is a deliberate decision about where testing effort goes. Effort is matched to risk: deep where failure is costly, light where it is cheap.

### High risk features - deep coverage

Login / auth
Deep coverage means every authentication path is always tested: successful Google sign-in, cancelled sign-in, first-time login creating a member account, session persistence across refresh and new tab, logged-out access to a protected route redirecting to login, and a logged-in user hitting /login being redirected away. Never skip: the check that a member cannot reach /admin and that an unauthenticated request to a protected API returns 401. Auth is the gate to everything, so no release goes out without these passing.

Task submission / progress saving
Deep coverage means testing each submission type (CHECKBOX, TEXT, LINK) for a correct write, the upsert updating the right user's record (never another user's), validation rejecting bad input (invalid status, non-URL link, wrong types), and completedAt being set on COMPLETED and cleared on NOT_STARTED. Never skip: confirming a submission is saved to the correct userId and survives a reopen, because this is where real user work can be lost.

Progress tracking (calcPhaseProgress, calcOverallProgress)
Deep coverage means unit tests for the percentage math at the edges: zero tasks, all tasks done, some done, and the rounding boundary. Never skip: a known input producing a known percentage (for example 23 of 26 equals 88), because every status badge and the unlock logic depend on this being correct.

### Medium risk features - reasonable coverage

Phase navigation / gating
Reasonable coverage means at least one test proving a phase locks when the previous is incomplete and unlocks when it is complete. Minimum confidence: the isPhaseUnlocked function returns false for an incomplete previous phase and true for a complete one. This is currently the highest-value missing test because the function is broken.

Dashboard
Reasonable coverage means verifying the overall percentage matches the task counts, the phase cards show the right status, and a completed phase does not also show overdue. Minimum: the progress number and the card counts agree, and no card shows two contradictory states.

Admin view (team table, export, reminders)
Reasonable coverage means one test per admin action: the team table loads per-member percentages, the CSV export produces a file with the right rows, and a reminder send returns success. Minimum: an admin can see member progress and export it without error, and a non-admin cannot reach these routes.

### Low risk features - light coverage

Glossary
Light coverage means a single smoke test: the glossary page loads and search returns a matching term. Absolute minimum: the page renders and is reachable from the navbar. No deep testing needed because it is read-only reference content with no data writes.

## Step 2 - Mobile Coverage Thinking

BetaninjasSEOLMS is a web app, but the same thinking applies: do not test every device, make deliberate choices based on what the team actually uses and what carries real risk.

### Device coverage matrix (hypothetical mobile version)

Built from what a small Nepal-based startup team realistically carries, not from guessing every device on the market.

| Tier | Device / OS | Why it matters |
|---|---|---|
| Must test | Mid-range Android (for example Samsung Galaxy A series), recent Android version | Most common device type for the team; the realistic primary user device. |
| Must test | iPhone, recent two iOS versions | Covers the iOS rendering engine (Safari/WebKit), which behaves differently from Android Chrome. |
| Should test | Older budget Android, one or two versions back | Catches performance and layout issues on slower, smaller-screen devices. |
| Should test | Small screen width (around 360px) | The dashboard cards and progress bar must not break on the narrowest common screen. |
| Skip for now | Tablets, very old OS versions, niche brands | Low or no usage on the team; effort not justified. |

The principle: cover the two engines (Android Chrome and iOS WebKit), the common mid-range device, and one weaker device for performance. That is the right 80 percent.

### Network coverage matrix

| Condition | Must test | Most at-risk features |
|---|---|---|
| WiFi / fast | Yes (baseline) | None specific; this is the happy path. |
| 4G | Yes | Task submission round-trip; the user should get clear feedback that a submit succeeded. |
| Slow 3G | Yes | Task submission and progress updates. On slow 3G, a submit may take seconds; the risk is the user taps Submit twice or navigates away thinking it failed, creating duplicate or lost-feeling submissions. The form should disable the button and show a pending state. |
| Offline | Yes | Task submission is the biggest risk. If a member writes a long TEXT response and submits with no connection, the response could be lost with no error. The app must detect offline, block the submit with a clear message, and preserve the typed text. |
| Flaky / intermittent | Should | Submission and auth. A dropped request mid-submit could leave the UI thinking it failed while the write actually succeeded, or vice versa. |

Network thinking summary: the feature most at risk across every degraded condition is task submission, because it writes user data over the network. Read-only screens (dashboard, glossary) degrade gracefully; the write path is where slow and offline conditions cause real harm.

## Step 3 - The Gap Between Coverage and Confidence

### What the Not Covered requirements could break

Sequential phase gating (locked phases cannot be entered): if never tested, a member could access and submit later-phase tasks before completing earlier ones, which breaks the entire guided learning model the product is built on. This is already broken in code (isPhaseUnlocked returns true).

Phase and overall progress calculation: if never tested, the percentages could be wrong, so a member sees the wrong completion state and phases could unlock or lock incorrectly.

Persistence on reopen (submitted response or link shown again): if never tested, a member could submit a response, reopen the task, see it blank, and assume their work was lost, then resubmit or give up.

### Honest evaluation of a Covered test case

Take EP-05 (invalid link rejected). As written, it confirms that sending "notaurl" returns a 400. That genuinely proves the requirement, because it asserts a specific outcome (rejection) tied to the rule (link must be a valid URL). It is closer to confidence than coverage.

Now contrast with ST-04 (TEXT submit sets status to Submitted). If that test only sends the request and checks for a 200, it is mostly going through the steps. To turn it into confidence it must also assert that the stored status is SUBMITTED and the response text was actually saved to the correct user's record. Running the submit is coverage; verifying the saved value is confidence.

The difference: a test that goes through the steps confirms the code did not crash. A test that verifies the right outcome confirms the code did the right thing. Coverage says it ran; confidence says it was correct.

### The test case that gives the most confidence

From the Phase 2 work, DT-1 (unauthenticated request returns 401) gives the most confidence. It is unambiguous, it tests the highest-risk gate (auth runs before everything else), and the assertion is exact: no session in, 401 out. There is no interpretation gap between what it checks and what the requirement demands, which is what makes it trustworthy.

## Step 4 - What Good Coverage Looks Like for BetaninjasSEOLMS

### Currently covered (from Phase 1 and 2)

Login and auth gating are well exercised through the black box and grey box cases. Task submission is covered across validation, equivalence classes, the decision table, and state transitions. The character-limit and date-boundary findings document real edges. This is solid coverage of the two highest-risk write and access paths, mostly at the integration and system levels.

### The most important gap

Phase gating. The spec makes sequential unlocking a core requirement, isPhaseUnlocked currently returns true unconditionally with the real logic commented out, and there is no test for it. Fixing the code and adding a test here would give the most confidence per unit of effort, because it protects the central promise of the product and a single assertion catches the regression.

### Five test cases to write next, with reasons

1. isPhaseUnlocked returns false when the previous phase is incomplete. Reason: directly catches the current broken gating, the highest-value missing test.
2. isPhaseUnlocked returns true when the previous phase is complete. Reason: confirms gating allows progress in the positive case, so the fix does not over-lock.
3. calcPhaseProgress returns the correct percentage for a known mix (for example 4 of 6 done equals 67). Reason: progress math is high risk and currently has zero unit tests; badges and unlocking depend on it.
4. calcOverallProgress returns 88 for 23 of 26 done. Reason: verifies the dashboard headline number that users trust, with an exact known-input assertion.
5. Submission persistence: submit a TEXT response, refetch the task, assert the saved response and status come back. Reason: covers the Not Covered persistence requirement and protects against the silent loss of user work.

This is the senior view of coverage: not chasing a percentage, but spending effort where a gap would hurt most. For this product, that is the progress and gating logic, where importance is highest and current verification is near zero.