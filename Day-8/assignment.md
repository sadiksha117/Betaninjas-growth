# Phase 3 Day 8 Assignment 1 - Read Three Functions

Target: lib/utils/timeline.ts and lib/utils/progress.ts

Reading and reasoning only, nothing is run.

## Function 1 - getPhaseStatus (lib/utils/timeline.ts)

```typescript
export function getPhaseStatus(
  progressPercent: number,
  startDate: Date,
  dueDate: Date,
  now: Date = new Date()
): TimelineStatus {
  if (progressPercent === 100) return 'COMPLETED'
  if (now < startDate) return 'NOT_STARTED'
  if (now > dueDate && progressPercent < 100) return 'OVERDUE'

  const totalDuration = dueDate.getTime() - startDate.getTime()
  const elapsed = now.getTime() - startDate.getTime()
  const timeElapsedPercent = Math.min((elapsed / totalDuration) * 100, 100)

  return progressPercent >= timeElapsedPercent ? 'ON_TRACK' : 'BEHIND'
}
```

### Inputs (4)
1. progressPercent - number
2. startDate - Date
3. dueDate - Date
4. now - Date, and this is the one with a default value (defaults to new Date(), the current time)

### Outputs (5 possible return values)
All are strings of the TimelineStatus union type:
1. 'COMPLETED'
2. 'NOT_STARTED'
3. 'OVERDUE'
4. 'ON_TRACK'
5. 'BEHIND'

### Branches (each path written as a sentence)
1. If progressPercent is exactly 100, the function returns COMPLETED.
2. Otherwise, if now is before startDate, the function returns NOT_STARTED.
3. Otherwise, if now is after dueDate and progressPercent is less than 100, the function returns OVERDUE.
4. Otherwise, if the final comparison holds and progressPercent is greater than or equal to timeElapsedPercent, the function returns ON_TRACK.
5. Otherwise (progressPercent is less than timeElapsedPercent), the function returns BEHIND.

### Minimum tests
Five test cases are needed to take every path at least once, one per return value:
1. progressPercent = 100 to reach COMPLETED.
2. now before startDate to reach NOT_STARTED.
3. now after dueDate with progressPercent under 100 to reach OVERDUE.
4. inside the window with progressPercent at or above the elapsed-time percent to reach ON_TRACK.
5. inside the window with progressPercent below the elapsed-time percent to reach BEHIND.
Why five: each return statement is a distinct outcome, and the last two outcomes (ON_TRACK vs BEHIND) come from the two sides of the final ternary, so they need separate tests. Hitting all five outcomes is the minimum to cover every path.

### One data type edge case
If progressPercent is passed as the string "100" instead of the number 100, the first branch does not fire. The check uses strict equality (===), which compares both value and type. "100" is a string and 100 is a number, so "100" === 100 is false. The function skips COMPLETED and falls through to the later branches, where "100" < 100 and the arithmetic with it can produce wrong or NaN results. This is exactly the kind of wrong-type bug that strict equality exposes: a value that looks right but is the wrong type silently takes the wrong path.

## Function 2 - calcPhaseProgress (lib/utils/progress.ts)

```typescript
function isDone(taskId: number, progressMap: ProgressMap): boolean {
  const status = progressMap[taskId]?.status
  return status === "SUBMITTED" || status === "COMPLETED"
}

export function calcPhaseProgress(
  tasks: { id: number }[],
  progressMap: ProgressMap
): number {
  if (tasks.length === 0) return 0
  const done = tasks.filter((t) => isDone(t.id, progressMap)).length
  return Math.round((done / tasks.length) * 100)
}
```

### Inputs
- tasks: an array of objects, each having at least an id of type number. It expects a list of tasks; only the id is read.
- progressMap: an object keyed by task id (number), where each value is an object with a status string. It maps a task id to that task's current status.

### Output
A number: the percentage of tasks that are done, rounded to a whole number. The range is 0 to 100 inclusive.

### The branch
There is one if condition: if tasks.length === 0 it returns 0. It checks for an empty task list and returns 0 early. This branch exists to prevent a division by zero. Without it, done / tasks.length would be 0 / 0, which is NaN, and the function would return NaN instead of a clean 0 percent.

### isDone
isDone treats two status values as done: SUBMITTED and COMPLETED. This means a task counts as done without being COMPLETED. A task that is only SUBMITTED is already counted toward progress. That is a deliberate design choice worth noting, because a member who has submitted but not had it confirmed still moves the percentage, and it matches the dashboard counting submitted work as progress.

### One edge case
If every task in the array is SUBMITTED but none are COMPLETED, isDone returns true for all of them (SUBMITTED counts as done). So done equals the number of tasks, and the function returns 100. An all-submitted phase reads as 100 percent complete even though nothing is marked COMPLETED.

### Isolation question
For a unit test of calcPhaseProgress I would use a fake progressMap, not the real database. The whole point of a unit test is to test this one function in isolation, fast and deterministically, without the slowness, setup, and flakiness of a real database connection. Because calcPhaseProgress already takes progressMap as a plain argument, I do not need to query anything: I would pass a hand-built object like { 1: { status: "COMPLETED" }, 2: { status: "SUBMITTED" }, 3: { status: "NOT_STARTED" } } along with a matching tasks array like [{ id: 1 }, { id: 2 }, { id: 3 }], and assert the function returns 67. This makes the test repeatable and lets me control exactly which statuses are present, so I can target edge cases (empty list, all done, none done, mixed) without touching real data. The function's design, taking the progress map as input rather than fetching it, is what makes it easy to unit test in the first place.

## Function 3 - isPhaseUnlocked (lib/utils/progress.ts)

```typescript
export function isPhaseUnlocked(
  phaseOrder: number,
  phases: { order: number; tasks: { id: number }[] }[],
  progressMap: ProgressMap
): boolean {
  return true // for testing, unlock all phases
  // if (phaseOrder === 1) return true
  // const prev = phases.find((p) => p.order === phaseOrder - 1)
  // if (!prev) return true
  // return prev.tasks.every((t) => isDone(t.id, progressMap))
}
```

### What it currently does
Right now the function returns true on the very first line, every time, for every input. No matter the phaseOrder, the phases, or the progressMap, it always returns true, meaning every phase is always treated as unlocked. The real logic beneath it never runs because the return is above it.

### What it is supposed to do
Reading the commented-out code: phase 1 is always unlocked (if phaseOrder is 1, return true). For any later phase, it finds the previous phase (order minus one). If there is no previous phase, it returns true. Otherwise it returns true only if every task in the previous phase is done (using isDone). In short, a phase unlocks only when the phase before it is fully complete, which is the sequential gating the spec requires.

### Why this is a bug
This is the same bug Kristi found in Phase 1: Phase 2 was accessible without completing Phase 1. Because the function always returns true, gating does not exist, so a member can enter any phase regardless of whether earlier phases are finished. The spec explicitly requires phases to unlock sequentially and that locked phases cannot be entered, so this directly violates a core requirement. It is the same defect, now located in the exact line that causes it.

### What a unit test would need to prove
If the real logic were uncommented, this test would catch the bug:
Input: phaseOrder = 2; phases = [{ order: 1, tasks: [{ id: 1 }, { id: 2 }] }, { order: 2, tasks: [{ id: 3 }] }]; progressMap = { 1: { status: "COMPLETED" } } (task 2 is not done).
Assert: isPhaseUnlocked(2, phases, progressMap) returns false, because phase 1 is not fully complete.
With the current code this assertion fails (the function returns true), which is exactly how the test exposes the bug. A companion test with task 2 also COMPLETED should assert the function returns true, confirming the gate opens correctly when the previous phase is finished.

## Data Types - One Finding (lib/validations/progress.ts)

```typescript
export const progressSchema = z.object({
  taskId: z.number().int().positive(),
  status: z.enum(["NOT_STARTED", "IN_PROGRESS", "SUBMITTED", "COMPLETED"]),
  response: z.string().optional(),
  link: z.string().url().optional(),
})
```

1. taskId as the string "5" instead of the number 5: Zod rejects it. z.number() does not coerce by default; it checks the actual type, and a string fails the number check, so safeParse returns a validation error (400 from the route). Coercion would only happen with z.coerce.number(), which this schema does not use.

2. taskId as -1: rejected. The chain requires int().positive(), and -1 is not positive (positive means greater than zero), so it fails validation.

3. status as "DONE" instead of "SUBMITTED": rejected. status is a z.enum limited to the four listed values, and "DONE" is not one of them, so validation fails.

### New decision table entry for the Phase 2 table
Add a row for the wrong-type taskId case:

| Auth | ID type | Status | Text | Expected outcome |
|---|---|---|---|---|
| Y | taskId sent as string "5" | valid | any | 400 validation error (z.number does not coerce a string, so the type check fails before any database work) |

This captures the data type edge case manual testers often miss: a value that looks correct ("5") but is the wrong type is rejected by the schema, and the team should test for it explicitly rather than assuming the field is always sent as a number.