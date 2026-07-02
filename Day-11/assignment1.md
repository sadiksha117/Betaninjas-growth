# Phase 5 Assignment 5.1 - Identify Pure Functions

Target: lib/utils/timeline.ts and lib/utils/progress.ts

## List of all functions (name, file, approximate line)

timeline.ts
1. getPhaseStatus - timeline.ts - around line 9
2. getDaysRemaining - timeline.ts - around line 25

progress.ts
3. isDone - progress.ts - around line 3 (private helper, not exported)
4. calcPhaseProgress - progress.ts - around line 8
5. calcOverallProgress - progress.ts - around line 16
6. isPhaseUnlocked - progress.ts - around line 24

(Line numbers are approximate and depend on spacing; the order within each file is correct.)

## Pure or Not Pure

| Function | File | Pure? |
|---|---|---|
| getPhaseStatus | timeline.ts | Pure (see note on the clock) |
| getDaysRemaining | timeline.ts | Pure (see note on the clock) |
| isDone | progress.ts | Pure |
| calcPhaseProgress | progress.ts | Pure |
| calcOverallProgress | progress.ts | Pure |
| isPhaseUnlocked | progress.ts | Pure, but currently broken (real logic commented out, always returns true) |

Reasons for any Not Pure: none of these six are truly impure. All six take their data as arguments, perform calculations, and return a value. None call prisma, fetch, auth, or mutate external state. They are all unit-test targets.

One honest nuance on getPhaseStatus and getDaysRemaining: both have a now parameter that defaults to new Date(). If you call them without passing now, they read the real system clock, which is a hidden dependency that would make a test non-deterministic. They stay testable as pure functions only because the default can be overridden: always pass an explicit now in tests, and the function becomes fully deterministic (same input, same output). So they are pure for testing purposes as long as now is supplied.

A note on isDone: it is pure but not exported, so it is tested indirectly through calcPhaseProgress and calcOverallProgress rather than directly.

## Easiest function to unit test: calcPhaseProgress

It is the easiest because its inputs are plain data (an array and an object) with no Date and no clock dependency at all, so there is nothing to override and the output is fully determined by the arguments.

### Test 1 - all tasks done returns 100
ARRANGE:
  tasks = [ { id: 1 }, { id: 2 } ]
  progressMap = { 1: { status: "COMPLETED" }, 2: { status: "SUBMITTED" } }
ACT:
  result = calcPhaseProgress(tasks, progressMap)
ASSERT:
  result equals 100
(Both tasks count as done because isDone treats SUBMITTED and COMPLETED as done.)

### Test 2 - no tasks done returns 0
ARRANGE:
  tasks = [ { id: 1 }, { id: 2 } ]
  progressMap = { 1: { status: "IN_PROGRESS" } }
ACT:
  result = calcPhaseProgress(tasks, progressMap)
ASSERT:
  result equals 0
(Task 1 is only IN_PROGRESS, task 2 has no entry, so neither counts as done.)

### Test 3 - empty task list returns 0 (the guard branch)
ARRANGE:
  tasks = [ ]
  progressMap = { }
ACT:
  result = calcPhaseProgress(tasks, progressMap)
ASSERT:
  result equals 0
(This exercises the if (tasks.length === 0) return 0 branch that prevents a divide-by-zero.)

A fourth scenario worth noting (partial completion): with tasks of ids 1, 2, 3 and only task 1 COMPLETED, the function returns 33 (one of three, rounded). This shows the rounding behaviour if the team wants a fourth test.