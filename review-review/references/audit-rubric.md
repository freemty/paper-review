# Review Review

Adversarial audit of a draft peer review against the source paper. Catches hallucinations, AI voice, field errors, and unrealistic demands before submission.

## When to Use

- Draft review is written (all form fields present)
- User wants a final check before submitting to OpenReview/CMT/HotCRP
- Can also be called mid-draft on individual fields

## How It Works

Audit the draft against accessible original paper/supplement artifacts. When
independent delegation is appropriate and available, pass those paths and the
draft to a read-only reviewer; otherwise perform the checks in the main thread.
Apply the relevant dimensions and report unverified claims when source coverage
is missing.

## The 7 Checks

### 1. Hallucination Audit

For every factual claim in the review (numbers, method details, what the paper does/doesn't do), verify against the paper:

- Quoted numbers match the paper's tables/figures exactly
- Claims about what the paper "does not report" are actually missing
- Method descriptions match what the paper actually proposes
- Cited sections/figures/equations exist and contain what the review says

Output: `VERIFIED`, `CONTRADICTED` or `UNVERIFIED`, with the supporting/missing locator.

### 2. AI Voice Detection

Flag any of:
- Em dashes (" -- " is fine, " --- " is not)
- Filler: "it is worth noting", "importantly", "interestingly", "notably", "furthermore", "moreover"
- AI vocabulary: "empirical package", "narrative", "landscape", "robust", "comprehensive", "nuanced", "substantive advance", "holistic"
- Parenthetical number comparisons in Summary or Strengths: "(FID 77 vs. 110, 30% better)"
- Overly uniform sentence length (all sentences 15-25 words)

Output: flagged phrases with line location and suggested rewrites

### 3. Field Content Verification

Check that content matches its field:
- Strengths field contains only positive points
- Weaknesses field contains only negative points
- Summary does not contain opinions or recommendations
- Justification does not introduce new points
- Confidential AC Comments contain no scientific critique

Output: `PASS` or `MISMATCH: [field] contains [wrong content type]`

### 4. Rebuttal Feasibility

For each suggestion in "Suggestions for Rebuttal":
- Can it be done within the actual venue's rebuttal window and realistic resources?
- Does it require new data collection? Flag.
- Does it require substantial compute? Distinguish a necessary validity concern from a requested rebuttal action.
- Is it a clarification, analysis, or small experiment? Pass.

Feasible examples: report timing, add error bars, run 3 seeds, state grid resolution, plot a curve from existing data
Infeasible examples: retrain at higher resolution, collect new dataset, run on 3 more benchmarks, implement a new baseline

Output: per-suggestion `FEASIBLE` or `INFEASIBLE: [reason + suggested alternative]`

### 5. Number Placement Check

- Paper Summary: use numbers only where necessary to explain the result, with a locator.
- Strengths: exact numbers are useful when central to the strength; keep the claim sourced.
- Major Weaknesses: numbers OK when making a precise technical argument.
- Minor Weaknesses: numbers OK for pinpointing (e.g., "Eq 6, line 292").

Output: misplaced numbers with location and fix suggestion

### 6. Minor Weakness Location Check

Each Minor Weakness must reference a specific location:
- Figure number (Fig X)
- Equation number (Eq Y)
- Section number (Sec N)
- Line number (line Z)
- Table number (Table N)

Output: items missing precise locations

### 7. Score-Tone Consistency

Compare the numeric score against the language used:
- Accept/Weak Accept score with reject language ("fundamental issues", "critically flawed", "major concerns that cannot be addressed") = inconsistent
- Reject score with accept language ("strong contribution", "well-executed", "impressive results") = inconsistent
- Borderline score should have balanced language

Output: `CONSISTENT` or `INCONSISTENT: score is [X] but language reads as [Y]`

## Subagent Prompt Template

When spawning the audit subagent, include:

1. The complete draft review (all fields, verbatim)
2. Accessible paper/supplement paths and evidence locators; declare excerpt-only limits if paths cannot be read
3. The 7 checks listed above with their output format
4. Return evidence-supported corrections and unknowns; do not fabricate failure findings.

Use any appropriate available read-only reviewer; a named LabMate agent is optional.
If delegation is disabled, the same rubric applies in the main thread.

## Output Format

Present results to the user in Chinese, with the specific flagged English text quoted inline:

```
## Review Audit Results

| Check | Result | Issues |
|-------|--------|--------|
| Hallucination | PASS/FAIL | N claims flagged |
| AI Voice | PASS/FAIL | N phrases flagged |
| Field Content | PASS/FAIL | N mismatches |
| Rebuttal Feasibility | PASS/FAIL | N infeasible items |
| Number Placement | PASS/FAIL | N misplaced |
| Minor Location | PASS/FAIL | N missing locations |
| Score-Tone | PASS/FAIL | consistent/inconsistent |

[Detailed findings per failed check]
```

For each FAIL, provide the exact fix (replacement text, not just "fix this").

## Integration with paper-review

This skill is standalone but designed to work with `paper-review`:
- After Phase 6 delivery is complete, user can call `/review-review` for a final audit
- Paper data from Phase 1 extraction is reusable as context
- Fix suggestions can be applied and re-audited iteratively
