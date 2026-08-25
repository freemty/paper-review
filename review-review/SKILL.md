---
name: review-review
description: Use when a draft peer review needs adversarial checking for hallucinations, fairness, feasibility, or score-tone consistency. Triggers on "check my review", "audit review", "review-review", or a completed review draft plus paper PDF in the conversation.
---

# Review Review

Read `references/audit-rubric.md` and audit the review against the paper.

Prioritize:

- unsupported claims or mislocated evidence;
- criticism that is impossible to address within rebuttal;
- field-specific content placed under the wrong criterion;
- score, confidence, and written-tone mismatch;
- AI-like filler that obscures actionable feedback.

Return concrete corrections with paper locations. Do not silently rewrite the
review unless the user asked for an integrated revision.
