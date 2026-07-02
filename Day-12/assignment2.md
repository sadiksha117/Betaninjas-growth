# Phase 5 Assignment 5.2 - Test Doubles and Setup/Teardown

Target: app/api/progress/route.ts (the POST handler)

The POST handler calls three things that cannot run in a unit test: auth(), request.json(), and prisma.progress.upsert.

## Test double type for each of the three

auth(): Stub. We replace it to return a hardcoded session object (for example { user: { id: "user-456" } }) so the handler gets past the auth check without a real session or network call. We only care that it returns a known value, not how it was called, which is what a stub is for.

request.json(): Stub. We replace it to return a fixed parsed body (for example { taskId: 1, status: "COMPLETED" }) so the handler receives controlled input without a real HTTP request. Again we just need it to hand back a known value on demand.

prisma.progress.upsert: Mock. We replace it with a mock because the assertion is about the interaction: we want to verify it was called exactly once and with the right where and data arguments. A mock records the calls and arguments, which is exactly what we need to prove the handler wrote the correct record. (If the test only needed a return value and did not check the call, a stub would do; here we are checking the call, so it is a mock.)

## beforeEach block for calcPhaseProgress tests

What goes in beforeEach (the shared setup that should reset before every test):
  BEFORE EACH TEST:
    progressMap = {}   reset the progress map to empty so no test leaks state into the next

What goes in each individual test's ARRANGE (only what that specific test needs):
  Test 1 ARRANGE:
    progressMap[1] = { status: "COMPLETED" }
    tasks = [ { id: 1 } ]
  Test 2 ARRANGE:
    tasks = [ { id: 1 }, { id: 2 } ]   (progressMap is already {} from beforeEach, so this stays 0 percent)

The principle: beforeEach holds the reset that every test shares (a clean empty progressMap). Each test's ARRANGE adds only the specific data that test needs. This keeps tests short and, more importantly, isolated, so one test cannot contaminate another through leftover map entries.

## Risk of passing null instead of a Dummy

Passing null is risky because null is not an inert placeholder, it is a value that can change the code's behaviour or crash it. If the handler ever reads a property on the object (for example session.user.id), null throws a "cannot read property of null" error, so the test fails for the wrong reason, an accidental crash, not the behaviour under test. A Dummy is a harmless object shaped just enough to satisfy the parameter (for example { user: { id: "test-user" } }), so the code can touch it safely without the dummy affecting the result. In short, null can trigger a different code path or an exception, which makes the test misleading; a proper Dummy keeps the test honest by being present but inactive.

## One AAA test using a Stub for prisma.progress.findMany

Scenario: a user with no completed tasks gets back an empty array from the GET handler.

ARRANGE:
  stub prisma.progress.findMany to return [ ]   (an empty array, no DB call)
  session = { user: { id: "user-789" } }   (a Dummy, just satisfies the auth check)
  stub auth() to return that session
ACT:
  result = await GET()   (the GET handler runs, calling the stubbed findMany)
ASSERT:
  result status is 200
  result body is an empty array [ ]
  prisma.progress.findMany was called with { where: { userId: "user-789" } }

Why a stub here: the test only needs findMany to hand back a known value (the empty array) so the handler has something to return. We are checking the handler's output, not deeply verifying the call, so a stub that returns a hardcoded empty array is the right double. The session is a dummy because the handler only needs user.id to be present to pass the auth check.