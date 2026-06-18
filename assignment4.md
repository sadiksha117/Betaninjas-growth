#  Day 3 Assignment 1 - Same Feature, Three Lenses

Target: Login feature - BetaninjasSEOLMS

Note on the target: the live app does not use a password login form. Authentication is Google OAuth only (next-auth with the Google provider). The test cases below are adapted to test what is actually present, and the mismatch with the original brief is noted as a finding.

## Step 1 - Black Box Test Cases (Login)

TC-01 - Valid login (happy path)
Steps: Open /login, click "Sign in with Google", choose a valid Google account, approve.
Expected: User is authenticated, a session is created, and the user is redirected to the homepage (/).
Result: Pass

TC-02 - Cancel or abandon the Google flow
Steps: Click "Sign in with Google", then close the Google popup or click Cancel without choosing an account.
Expected: No session created, user remains on /login, no crash, can retry.
Result: Pass

TC-03 - Access a protected page while logged out (replaces "empty fields")
Steps: Without logging in, manually visit / or /dashboard in the URL bar.
Expected: Redirected to /login with no protected content shown.
Result: Pass

TC-04 - Login with a Google account not previously in the system (replaces "invalid email format")
Steps: Sign in with a brand new Google account that has never used the app.
Expected: Login succeeds, a new member account is created automatically, default role is "member".
Result: Pass

TC-05 - Session persists on refresh and in a new tab (replaces "very long input")
Steps: After logging in successfully, refresh the page, then open / in a new tab.
Expected: User stays logged in, no re-login required, session persists. The app uses JWT sessions, so this should hold.
Result: Pass

Finding: The brief's wrong-password, empty-field, and long-input cases do not apply because there is no credential form. Flagging that mismatch is itself a valid black box finding.

## Step 2 - Grey Box Test Cases (from reading auth.ts and middleware.ts)

These are cases that could only be designed after seeing the internals.

GB-01 - Role is assigned only on first sign-in
Reading auth.ts: role is set during the signIn upsert as "admin" if the email matches ADMIN_EMAIL, otherwise "member". The role is written on create but not on update.
Test case: Sign in with an account before its email is added to ADMIN_EMAIL, then add the email to ADMIN_EMAIL and sign in again. Expected likely result: the user stays "member" because the existing record's role is never updated on subsequent logins, only on first creation. A black box tester would never predict this.

GB-02 - Login page and protected route redirects
Reading middleware.ts: if logged in and visiting /login, the user is redirected to /; if logged out and visiting any non-API, non-login page, the user is redirected to /login.
Test case: While logged in, manually navigate to /login and confirm the redirect to /. Then log out and confirm any protected route bounces back to /login. Additional catch: /api routes are skipped by the logged-out redirect, so an API route is not protected by this middleware branch. Worth testing whether an unauthenticated API call is handled elsewhere.

## Step 3 - White Box Analysis: getDaysRemaining

    export function getDaysRemaining(dueDate: Date, now: Date = new Date()): number {
      const diff = dueDate.getTime() - now.getTime()
      return Math.ceil(diff / (1000 * 60 * 60 * 24))
    }

Input: dueDate (a Date), and optionally now (a Date, defaults to the current time).

Returns: a number, the whole days remaining until the due date, rounded up via Math.ceil. The divisor 1000 * 60 * 60 * 24 converts milliseconds to days.

Edge cases (found by reading, no running needed):

1. Due date is in the past. diff is negative, so the function returns a negative number (for example -2). The function does not clamp to 0, so any UI showing "X days remaining" would display a negative value unless the caller handles it.

2. Due date is today or only a few hours away. Because of Math.ceil, any positive fraction rounds up to at least 1. A deadline 0.04 days away (about an hour) returns 1, not 0. So "1 day remaining" can actually mean "due within the next hour", which is misleading.

3. Due date equals now exactly. diff is 0, 0 divided by 86400000 is 0, and Math.ceil(0) is 0. Returns 0, the one input that gives exactly zero. Anything a millisecond later returns negative; anything a millisecond earlier returns 1.

Extra: getTime() works in UTC milliseconds, so daylight-saving boundaries and timezone differences between server and client could shift the day count by one.

## Step 4 - Testing Types That Apply to Login

Functional - Confirms the core behavior works: a valid Google account signs in and lands on the right page, and protected routes are gated. If login is broken, nothing else in the app is reachable.

Regression - Auth touches middleware, session callbacks, and role logic, so any change risks breaking sign-in or route protection. It must be re-tested after every deployment.

Security - Login is the primary attack surface. This needs checks on session and JWT handling, the admin-email role check, and the gap where /api routes bypass the logged-out redirect.

Usability - The user must clearly understand how to sign in (single Google button) and what happens if they cancel. Confusing or dead-end states hurt adoption.

Accessibility - The sign-in button and any redirects must work with keyboard navigation and screen readers, since login is the entry point everyone must pass through.

Compatibility - OAuth popups and redirects behave differently across browsers and mobile, so sign-in should be verified across environments.

## Step 5 - Reflection

Grey box testing found the most interesting issues for me, because reading auth.ts exposed the role-on-create-only behavior that I never would have guessed from the outside. Black box was the easiest to perform but the most limited here, since the app only has Google sign-in and the brief's password scenarios did not even apply. White box was the hardest in a different way: the getDaysRemaining function is only three lines, but reasoning through Math.ceil on negative and near-zero values took the most careful thinking. Overall, having access to the code turned vague guesses into specific, testable predictions.