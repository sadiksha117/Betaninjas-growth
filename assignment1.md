# Assignment 1 — The Bug Trail
**Name:** Sadiksha Gyawali 
**App:** Betaninjas SEO LMS  
**Date:** June 16, 2026  

---

## The Bug - What It Is and How to Reproduce It

On the LINK type task, the app accepts and successfully submits clearly invalid URLs without showing any error. When I typed `https://a` into the submission field and clicked "Submit Link", the app responded with a green tick and "Submitted!" , exactly the same success state as a real, valid URL. The field even highlighted green as if the input was correct. According to the spec, an invalid URL should be blocked with an inline error shown next to the field. Neither happened. The submission went through, the success state was shown, and the task was marked as done - all from a URL that is not valid by any standard.

**Steps to reproduce:**
1. Log in as a member
2. Navigate to any LINK type task
3. Type `https://a` in the URL input field
4. Click "Submit Link"
5. Observe: green tick appears, "Submitted!" shown - no error, no block

---

## Which Principle It Relates To and Why

This bug relates to **Principle 2 - Exhaustive testing is impossible.** The developer who built the URL validation likely tested it with a proper, realistic URL like `https://google.com` or `https://hubspot.com/certificate/abc`. It worked correctly, so they moved on. The problem is that no one can test every possible URL input as there are infinite variations. As a result, boundary cases like `https://a` , which has a valid prefix but no real domain or TLD , were never tried. The bug exists not because the developer was careless, but because exhaustive testing of every edge case is not realistic. This is exactly what Principle 2 warns us about: the absence of found errors does not mean the software is correct.

---

## Which SDLC Stage Could Have Caught It and How

The **Development stage** could have caught this before it ever reached the live app. If the developer had written unit tests specifically for the URL validator function testing inputs like `https://a`, `https://`, `abc`, an empty string, and a URL with no TLD . the failure would have appeared immediately during development. This did not require a QA engineer or a live testing session. A simple set of boundary-value test cases on the validation logic alone would have been enough. The bug reached production because the validator was likely tested only with valid inputs, never with deliberately broken ones.

---

## Static or Dynamic - Which Would Have Found It First

**Dynamic testing** finds this bug. Reading the spec tells you that URL validation should exist and that invalid URLs should be blocked but it gives no indication that the validation is missing or broken. You only discover the problem by actually opening the app, typing `https://a` into the field, and clicking submit. The moment you see "Submitted!" appear with a green tick instead of an error message, the bug is right in front of you. No static review of the requirements document, the feature spec, or even the source code would make this obvious without running the application and testing the input directly.