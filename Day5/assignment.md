# Assignment- Document It Like It Matters

Target: Dashboard feature - betaninjas-seo-learning-tracker.vercel.app

## Step 1 - Test Scenarios and Test Cases

### Scenario 1: Overall progress summary displays correctly

TC-1.1 - Overall progress percentage matches task counts
Steps:
1. Log in as a member and open /dashboard.
2. Read the "Overall Progress" card: note the percentage and the "X of Y tasks complete" text.
3. Calculate X divided by Y as a percentage and round.
Expected result: The displayed percentage equals the rounded value of completed tasks divided by total tasks. For 23 of 26, the value should read 88%.

TC-1.2 - Progress bar fill matches the percentage
Steps:
1. On /dashboard, look at the Overall Progress bar.
2. Compare the filled width of the bar to the stated percentage.
Expected result: The filled portion of the bar visually matches the percentage shown (about 88% filled, not full and not empty).

TC-1.3 - Task totals across phase cards add up to the overall total
Steps:
1. On /dashboard, read the "X / Y tasks" count on each of the five phase cards.
2. Sum the completed counts and the total counts across all phases.
Expected result: The sum of all phase totals equals the total shown in Overall Progress (26), and the sum of completed equals the completed count (23).

### Scenario 2: Phase cards show correct status and metadata

TC-2.1 - Phase status badge matches task completion
Steps:
1. On /dashboard, look at each phase card's badge (Completed or Overdue).
2. Compare the badge to the card's "X / Y tasks" count.
Expected result: A card showing all tasks complete (for example 6 / 6) shows Completed; a card with incomplete tasks past its due date shows Overdue. Phase 5 (0 / 3) should show Overdue.

TC-2.2 - Clicking a phase card opens that phase's detail page
Steps:
1. On /dashboard, click the Phase 1 card.
2. Observe the page that loads.
Expected result: The Phase 1 detail page opens, showing the breadcrumb "Dashboard > Phase 1 - SEO Basics", the phase title, the learning objectives, resources, and the task list (6 tasks).

TC-2.3 - Completed phase does not also show as overdue
Steps:
1. On /dashboard, find a phase with a Completed badge and all tasks done (for example Phase 1, 6 / 6).
2. Read any due-date or overdue text on the same card.
Expected result: A phase marked Completed with 100 percent of tasks done should not display "Overdue by X days", because a finished phase cannot be overdue.

## Step 2 - Bug Report

Title: Dashboard and phase detail show "Completed" and "Overdue" at the same time for fully completed phases

Steps to reproduce:
1. Log in as a member and open /dashboard.
2. Look at the Phase 1 - SEO Basics card.
3. Note the green "Completed" badge and the "6 / 6 tasks" count.
4. On the same card, note the red text "Overdue by 59 days".
5. Click the card to open the Phase 1 detail page.
6. At the top of the detail page, note "Completed", "100% complete", and "Overdue by 59 days" shown together.

Expected result: A phase that is 100 percent complete shows only the Completed status. No overdue label appears, because a finished phase cannot be overdue.

Actual result: Phases 1, 2, 3, and 4 each show a Completed badge with all tasks done and at the same time show "Overdue by X days" (Phase 1: 59 days, Phase 2: 61 days, Phase 3: 21 days, Phase 4: 43 days). The contradiction appears on both the dashboard cards and the phase detail page.

Environment: Google Chrome on Windows, viewing the deployed app at betaninjas-seo-learning-tracker.vercel.app.

Severity: Low to medium. The data is not lost or wrong at the task level, and the feature still works, but the status display is logically contradictory and undermines trust in the dashboard's accuracy.

Priority: Medium. It is highly visible, appears on four of five phases, and is the first thing a user sees, so it should be fixed soon even though it does not block usage.

## Step 3 - Defect Lifecycle

Bug: Completed phases also display as Overdue.

New: The tester (Sadiksha) files the bug in the tracker with full reproduction steps, severity, and priority. The bug exists but no one is assigned yet.

Assigned: The QA lead or project manager reviews the report, confirms it is valid, and assigns it to a developer who owns the dashboard or timeline status logic.

Open: The developer accepts the bug and begins investigating. The likely cause is that the overdue label is driven by the due date alone (a past due date) without checking whether the phase is already 100 percent complete, so the completed state does not suppress the overdue text.

Fixed: The developer changes the status logic so that a phase at 100 percent complete never shows Overdue, commits the fix, and deploys it to the test or staging environment. The bug status is set to Fixed.

Retest: The tester re-runs the reproduction steps on the environment with the fix. They check all five phases, including the completed ones and Phase 5, which should still correctly show Overdue.

Verified: The tester confirms the fix works. For this specific bug, Verified means the completed phases (1 to 4) now show only Completed with no overdue text, while Phase 5, which is genuinely incomplete and past due, still correctly shows Overdue. The fix solved the problem without breaking the legitimate overdue case.

Closed: With the fix verified in the released build and no side effects found, the QA lead or tester marks the bug Closed. For this bug, Closed means the contradiction no longer appears anywhere it was reported (dashboard cards and phase detail pages) and the ticket needs no further action.

## Step 4 - Entry and Exit Criteria

### Entry criteria (when the dashboard is ready to test)
1. The dashboard build is deployed to the test environment and the URL loads without a server error.
2. A test member account exists and can log in successfully through Google sign-in.
3. Test data is seeded: at least five phases exist, each with tasks, and a mix of completed, in-progress, and overdue states so all status paths can be checked.
4. The Overall Progress card, the five phase cards, and the phase detail pages all render.
5. The test cases for the dashboard (Step 1) are written and reviewed.

### Exit criteria (when dashboard testing is done enough to release)
1. All Step 1 test cases have been executed and recorded as pass, fail, or skipped.
2. No severity 1 (blocker or critical) bugs remain open on the dashboard.
3. The status display is logically consistent: no phase shows two contradictory states at once.
4. The Overall Progress percentage and the phase task counts agree with each other.
5. Any remaining open bugs are low severity, documented, and accepted by the QA lead or project manager for a later fix.

## Step 5 - Test Report

What was tested:
The dashboard of the SEO learning tracker, covering the Overall Progress summary and the five phase cards, plus navigation into a phase detail page. Two scenarios and six test cases were run as a logged-in member.

What passed:
- TC-1.1: Overall Progress showed 88% for 23 of 26 tasks, which matches the calculation.
- TC-1.2: The progress bar fill visually matched 88%.
- TC-1.3: Phase task counts (6+8+4+5+3 = 26 total, 6+8+4+5+0 = 23 complete) add up to the Overall Progress totals.
- TC-2.1: Phase 5 with 0 of 3 tasks and a past due date correctly showed Overdue.
- TC-2.2: Clicking the Phase 1 card opened the correct phase detail page with breadcrumb, objectives, resources, and the 6-task list.

What failed:
- TC-2.3: Phases 1 to 4 show a Completed badge with all tasks done while also showing "Overdue by X days". A completed phase should not display as overdue. The same contradiction appears on the phase detail page (Completed, 100% complete, and Overdue by 59 days together). This is logged as a bug in Step 2.

What was skipped and why:
No test cases were skipped. All six were executed against the live deployment.

Summary:
The dashboard's core data is accurate. Progress totals, task counts, navigation, and the genuinely overdue phase all behave correctly. The one defect is a status-display contradiction where completed phases are also labelled overdue, which is visible but does not block use. The feature is usable, and quality is good overall once the overdue-label logic is corrected for completed phases.
