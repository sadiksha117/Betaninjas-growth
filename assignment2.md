# Assignment 2 — Test This Feature
**Name:** Sadiksha Gyawali  
**Feature:** Task Submission  
**Product:** Betaninjas SEO LMS  
**Date:** June 16, 2026  

---

## Scope

### What I am testing
- CHECKBOX task submission and toggle behaviour
- TEXT task submission, empty validation, and revision
- LINK task submission and URL validation
- Task state transitions — Not Started → In Progress → Submitted / Completed
- Progress percentage update on phase page and dashboard after submission

### What I am not testing
- Admin approving or rejecting individual submissions
- File uploads as evidence
- Notifications triggered by submission
- Public or peer-visible submissions
- Admin submitting on behalf of a member

---

## Entry Criteria
Before testing begins, the following must be in place:

- Live app is accessible at `betaninjas-seo-learning-tracker.vercel.app`
- A member account is available to log in with
- At least one task of each type (CHECKBOX, TEXT, LINK) is available in the app
- Feature spec has been reviewed and understood by the tester
- No active deployments or code changes are happening during the test session

---

## Exit Criteria
Testing is complete when:

- All 5 test cases have been executed
- Each test case has a clear Pass or Fail result recorded
- All failing test cases have been documented with steps to reproduce
- Progress percentage has been verified on both the phase page and the dashboard
- No blocker bugs are left without a Jira ticket raised

---

## Test Cases

---

### TC01 - Valid URL submission on a LINK task
**Type:** Happy path  
**Label:** Verification

**Steps:**
1. Log in as a member
2. Navigate to any LINK type task
3. Paste a valid URL — e.g. `https://www.hubspot.com/certificate/abc`
4. Click "Submit Link"
5. Return to the phase page
6. Return to the dashboard

**Expected result:**
- "Submitted!" confirmation appears after clicking Submit Link
- The submitted URL is visible when returning to the task
- Phase page progress percentage updates without a page reload
- Dashboard overall progress percentage also updates

**Result:** Pass

---

### TC02 - Invalid URL `https://a` should be blocked on a LINK task
**Type:** Edge case  
**Label:** Verification

**Steps:**
1. Log in as a member
2. Navigate to any LINK type task
3. Type `https://a` in the URL input field
4. Click "Submit Link"

**Expected result:**
- Submission is blocked
- An inline error message appears next to the field
- No success state is shown
- Task status does not change

**Result:** Fail   
**Actual behaviour:** The app accepts `https://a` as a valid URL, shows a green tick and "Submitted!" — no error shown, submission goes through successfully.  
**Note:** This passes verification in the sense that `https://a` begins with `https://` — but it fails validation entirely. No real user would submit `https://a` as a certificate or repo link. The validation logic is only checking for the presence of a prefix, not whether the URL is genuinely valid. This is the most valuable finding — it passes a surface-level check but fails real-world usefulness.

---

### TC03 - Revised TEXT response replaces old response
**Type:** Edge case  
**Label:** Validation

**Steps:**
1. Log in as a member
2. Navigate to any TEXT type task
3. Type a first response — e.g. `This is my first answer`
4. Click Submit
5. Return to the same task
6. Clear the textarea and type a new response — e.g. `This is my revised answer`
7. Click Submit again
8. Return to the task once more

**Expected result:**
- Only the new response `This is my revised answer` is visible
- The old response `This is my first answer` is gone
- The task remains in Submitted state

**Result:** Pass

---

### TC04 - Empty TEXT submission should be blocked
**Type:** Error scenario  
**Label:** Verification

**Steps:**
1. Log in as a member
2. Navigate to any TEXT type task
3. Leave the textarea completely empty
4. Observe the Submit button

**Expected result:**
- The Submit button is disabled
- Member cannot click Submit with an empty field
- No error toast or submission attempt occurs

**Result:** Pass

---

### TC05 - CHECKBOX task completes with one click
**Type:** Edge case  
**Label:** Validation

**Steps:**
1. Log in as a member
2. Navigate to any CHECKBOX type task
3. Click the checkbox directly
4. Observe whether the task status changes to Completed
5. Click the checkbox again
6. Observe whether the task toggles back to not completed

**Expected result:**
- One click on the checkbox marks the task as Completed
- A second click toggles it back
- No dropdown interaction is needed

**Result:** Fail   
**Actual behaviour:** Clicking the checkbox directly does nothing. The only way to change the task status is by manually selecting "Done" from a dropdown menu. The dropdown also uses the label "Done" which does not match the spec's defined states — the spec defines "Completed" for CHECKBOX tasks, not "Done."  
**Note:** This passes verification loosely — the status does eventually change — but it fails validation completely. A real user expects one click to complete a checkbox task as the spec describes. Having to open a dropdown and manually select a status defeats the purpose of a checkbox entirely and introduces the risk of members setting their own status without actually doing the work.

---

## Summary

| TC | Description | Type | Label | Result |
|----|-------------|------|-------|--------|
| TC01 | Valid URL submitted and saved | Happy path | Verification |  Pass |
| TC02 | `https://a` should be blocked | Edge case | Verification |  Fail |
| TC03 | Revised TEXT replaces old response | Edge case | Validation |  Pass |
| TC04 | Empty TEXT submit blocked | Error scenario | Verification |  Pass |
| TC05 | CHECKBOX completes with one click | Edge case | Validation |  Fail |

---

## Key Finding - Passes Verification but Fails Validation

Both failing test cases (TC02 and TC05) are examples where the feature passes a surface-level check but fails when tested against real user expectations:

- **TC02** - `https://a` has the correct URL prefix format, so it passes basic format validation. But no real user submits `https://a` as a certificate link. The validation is not meaningful enough to protect data quality.
- **TC05** - The task status does change eventually via the dropdown, so the underlying state logic works. But the spec explicitly says "one click to complete" — a dropdown is not one click, and it allows members to mark tasks done without any real interaction with the checkbox. The intended behaviour is broken even though the data layer is functioning.

These are the most valuable findings from this test session.