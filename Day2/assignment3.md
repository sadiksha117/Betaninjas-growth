# Assignment 3 — Our Last Sprint — Honest Look
**Name:** Sadiksha  
**Sprint:** TalkTravel - Recent iOS/Web Sprint  
**Date:** June 16, 2026  

---

## When Did QA Get Involved?

QA got involved only after development was finished. We were not part of sprint planning, requirements discussion, or any early design decisions. Builds were handed over to us once the developers had completed their work, and our job was to test what was already built. This meant that by the time we found issues, the code was already written, sometimes already deployed, and fixing things cost significantly more time than it would have if we had caught them earlier in the process.

---

## One Real Example Each of QA, QC, and Testing

**QA - Preventing a problem**

Because we only joined after development, there is honestly very little to point to as pure defect prevention. However, one example where QA thinking helped was when we flagged that new builds were being deployed before previously reported bugs were fixed. By raising this pattern , that fixes were being shipped one at a time while the bug backlog kept growing , we were at least trying to prevent the situation from getting worse. It was not ideal QA involvement, but it was an attempt to shift the process in the right direction before more issues compounded.

**QC - Finding a problem after it was built**

A clear QC example is the My Profile Posts/Comments disappearing bug on TalkTravel iOS. After the profile feature was built and deployed, manual testing revealed that switching to the Posts tab from the Comments tab caused the entire Posts/Comments section to disappear , no tab selector, no content, no empty state, nothing rendered below the Friends/Achievements row. The broken state also persisted across navigation to other modules, and the only way to recover it was by accessing My Profile via the left sidebar avatar. This was found after the feature was already live, which is exactly what QC is , finding problems in a product that already exists. The bug has since been fixed.

**Testing - Running a test**

A specific test I ran was an automated Playwright test on the TalkTravel Web login flow that uncovered two issues (TVHK-2520). The test checked the login error behaviour by submitting invalid credentials and inspecting the DOM and network response. It found that the error message had no `role="alert"` element - meaning screen reader users received no audible announcement that their login had failed, which is a WCAG 4.1.3 accessibility violation. The same test also caught that the login API was returning HTTP 404 instead of the correct HTTP 401 for invalid credentials, which is semantically incorrect and misleading for developers and monitoring tools. Running this automated test against the live web app is what surfaced both issues .This is a direct example of testing, executing defined steps and assertions against the app to verify it behaves as expected.

---

## Which Agile Ceremonies Did QA Join and Which Did It Miss

The only ceremony QA consistently attended was the **retrospective**. We were absent from sprint planning, daily standups, and sprint reviews.

The impact of this was significant. Because we were not in sprint planning, we had no input into what was being built, no visibility into the complexity of features, and no chance to flag testability concerns or ask questions about requirements before a single line of code was written. Because we were not in standups, we had no real-time awareness of what developers were completing day by day, so we could not prepare test cases in advance or flag risks early. The result was that most bugs , including the accessibility violation on the login page and the Posts/Comments disappearing issue , were only caught after the build was deployed to production. Issues that could have been caught in development or staging were instead found in prod, making them more expensive and disruptive to fix.

---

## One Concrete Change for Next Sprint

QA should be included from sprint planning onwards , not just handed a build at the end. Specifically, the team should adopt a rule that **no new build is deployed until the majority of bugs reported in the previous build are fixed and verified.** Right now the pattern is: one bug gets fixed, a new build ships, new bugs appear on top of old ones, and the backlog keeps growing. This creates a situation where QA is always chasing a moving target. If the team commits to clearing most of the known bug backlog before deploying a new build, QA can test on a stable base, findings become more meaningful, and the overall quality of each release improves. This is specific, doable, and directly addresses what slipped in this sprint.