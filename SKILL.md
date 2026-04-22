---
name: paper-review
description: |
  Use when the user provides a paper PDF for peer review, mentions reviewing a paper,
  references a venue submission form (OpenReview/CMT/HotCRP), or shares a paper path
  with a venue name or target score.
---

# Paper Review

Multi-role academic peer review that produces submission-ready output matching venue form fields.

## Why Multi-Role

A single-pass review has blind spots. Four complementary expert lenses (domain, modeling, experiments, systems) catch different classes of issues. Cross-reviewing then surfaces what's consensus vs. disagreement, so the final review is calibrated rather than one-sided. Every claim in the output must trace to specific evidence in the paper (section, table, figure, equation).

## Phase 1: Read and Extract

Before anything else, read the full paper + supplementary. For PDFs > 10 pages, read in chunks using the `pages` parameter.

Extract and write down:

1. **Metadata**: title, venue, submission number, contribution type (CT) if declared
2. **Core idea**: one sentence
3. **Method pipeline**: 3-5 numbered technical components with specifics (model sizes, loss functions, key hyperparameters)
4. **Experimental setup**: datasets (with sizes), baselines (with years), metrics, ablations
5. **Key numbers**: reproduce important table rows with exact values. These go into subagent prompts since subagents cannot read PDFs.
6. **Supplementary findings**: additional experiments, real-world results, acknowledged limitations

If the user provided venue guidelines or a review form, parse the required fields. See `references/venue-formats.md` for field mappings across OpenReview, CMT, and HotCRP venues. If no venue is specified, ask the user or default to NeurIPS/ICML style.

## Phase 2: Assign 4 Expert Roles

Select roles based on the paper's domain. The goal is non-overlapping analytical coverage.

| Paper Domain | Role A | Role B | Role C | Role D |
|---|---|---|---|---|
| 3D / Rendering | Domain (3D representations) | Paradigm (diffusion/AR/flow) | Evaluation methodology | Compute & scalability |
| NLP / LLM | Task domain | Architecture & training | Benchmark & statistics | Inference & serving |
| RL | Environment & reward | Algorithm & convergence | Reproducibility | Sample efficiency |
| Theory | Mathematical foundations | Proof techniques | Empirical validation | Connections to practice |
| Datasets | Data quality & bias | Annotation methodology | Benchmark utility | Scale & access |

Adapt freely: theory-heavy paper → replace systems with proof reviewer. Dataset paper → replace modeling with annotation quality. The point is 4 distinct lenses.

## Phase 3: Parallel Independent Reviews

Spawn all 4 reviewer subagents in a single message for concurrency. Use `subagent_type: "labmate:domain-expert"` if available; otherwise use the default general-purpose agent.

Each subagent prompt must be fully self-contained. It has no access to PDFs or conversation history. See `references/role-templates.md` for the required prompt structure: identity block, full paper summary with numbers, 4-6 pointed analytical questions, and output format.

**The single most important thing**: the paper summary block in each prompt must include concrete numbers from the extracted tables. A prompt saying "results are competitive" produces hollow reviews. A prompt saying "FID 5.68 vs L3DG's 8.49, 33% improvement" produces grounded ones.

Each reviewer outputs:
- **Strengths** (3): concise, one bold topic + one sentence of evidence each. No inline numbers or parenthetical comparisons.
- **Major Weaknesses** (3-4): issues that drive the score. Each must be something the authors could plausibly address in a 7-day rebuttal (analysis, timing table, extra run). Never demand retraining or new large-scale experiments.
- **Minor Weaknesses** (2-4): fixable in revision. Focus on: typos, notation issues, missing details, and plug-and-play suggestions (e.g., swap sampler, add a table row) that don't require retraining.
- **Score**: on the venue's scale
- **Key Questions** (2-3): what the authors should address in rebuttal

Under Concept & Feasibility CTs, reviewers should not penalize for lacking large-scale experiments. Focus on novelty, soundness, and feasibility validation credibility.

## Phase 4: Cross-Review

After all 4 reviews return, build a consensus matrix:

```
| Issue | A | B | C | D | → Final |
|-------|---|---|---|---|---------|
| [specific issue] | Major | Major | Major | — | 3/4 → Major |
| [specific issue] | Minor | — | Major | Major | 2/4 Major → Final Major |
```

Classify each point:
- **Consensus** (3-4 reviewers): high confidence, include as-is
- **Majority** (2 reviewers): moderate confidence, include with nuance
- **Unique but compelling** (1 reviewer): include if well-argued with evidence
- **Disagreement**: present both sides explicitly in the final review

Record the score distribution (all 4 scores + median).

## Phase 5: Synthesize Final Review

Produce output matching the venue's form fields exactly. The default structure (use when no specific form is provided):

- **Title**: short descriptive review title
- **CT Justification**: "I agree." if using the authors' CT, otherwise brief explanation
- **Paper Summary**: 3-5 sentences. Describe what the paper does and why, at the level of a colleague summarizing over coffee. No method pipeline enumeration, no inline numbers, no parenthetical baselines. The reader should understand the problem, the key idea, and the outcome.
- **Strengths**: 3 items. Each: bold topic + 1-2 sentences of evidence. Keep concise. No specific numbers in strengths (reference table/figure instead of quoting values).
- **Major Weaknesses**: 3-4 items. Each must be something the authors can plausibly address in a 7-day rebuttal window (report timing, add error bars, run 2 extra seeds, state a missing number). Never ask authors to retrain models, collect new data, or run experiments requiring >1 GPU-day. Each weakness stands on its own without requiring cross-reference to other weaknesses.
- **Minor Weaknesses**: 2-4 items. Typos with exact location (Fig/Eq/line), notation inconsistencies, missing but easily-added details, and plug-and-play suggestions that don't require retraining (e.g., swap DDPM sampler for DDIM, add a row to an existing table).
- **Recommendation**: score on venue scale
- **Justification**: 3-5 sentences weighing strengths vs weaknesses. Match tone to score (see Score-Tone Calibration). No new points here, only synthesis of what was already said.
- **Rebuttal Suggestions**: one suggestion per Major Weakness, numbered to match (W1 -> suggestion 1). Each must be completable in 7 days. End with: "If these are adequately addressed, I am willing to raise my score."
- **Confidence**: score + one-sentence justification
- **Confidential AC Comments**: meta-observations only (e.g., reviewer agreement patterns, borderline positioning). No scientific critique.

### Score-Tone Calibration

If the user provided a target score, calibrate tone:

| Range | Lead with | Weakness framing |
|---|---|---|
| Accept (top tier) | Strengths | Constructive suggestions |
| Weak Accept | Balanced | Roughly equal weight |
| Borderline | Weaknesses slightly | Acknowledge potential |
| Reject | Fundamental issues | Strengths are secondary |

### Language

- Review content: academic English (this gets submitted)
- User communication: Chinese (中文)
- Every paragraph earns its place

**Anti-AI-voice rules** (reviewers are human, the review must read like one):
- No em dashes. Use commas, periods, or semicolons.
- No inline number comparisons in parentheses like "(FID 77 vs. 110, 30% better)". Reference "Table 1" or "Fig 3" instead. Numbers belong in Major Weaknesses when making a specific technical point, not scattered everywhere.
- No filler phrases: "it is worth noting", "importantly", "interestingly", "notably", "furthermore", "moreover", "it should be noted that"
- No AI vocabulary: "empirical package", "narrative", "landscape", "robust", "comprehensive", "nuanced", "substantive advance", "holistic"
- Short sentences. Vary length. A 5-word sentence after a 20-word one reads human.
- Write like a tired but conscientious reviewer at 11pm, not like a language model producing "balanced academic prose"

## Phase 6: Iterative Delivery and Verification

Do NOT dump the entire review in one message. Deliver field by field, waiting for user confirmation before moving to the next:

1. **Paper Summary** -> wait for user OK or edits
2. **Strengths** -> wait
3. **Major Weaknesses** -> wait
4. **Minor Weaknesses** -> wait
5. **Recommendation + Justification + Rebuttal Suggestions** -> wait (these three are tightly coupled, deliver together)
6. **Confidence + Confidential AC Comments** -> wait

At each step, output in plain text ready to paste into the venue form (no markdown headers, no extra formatting beyond what the form accepts). Use the user's language (中文) for commentary/questions, but keep review content in academic English.

After all fields are confirmed, suggest: "All fields ready. Run `/review-review` to audit for hallucinations and AI voice before submitting?"

Then run a **completeness check**:
- Verify each form field has content and is in the correct field (e.g., Strengths content is actually about strengths, not weaknesses)
- Verify Rebuttal Suggestions map 1:1 to Major Weaknesses
- Verify no specific numbers appear in Paper Summary or Strengths (reference table/figure instead)
- Verify Minor Weaknesses reference exact locations (Fig X, Eq Y, line Z, Sec N)
- Flag any issues to the user before they submit

## Common Mistakes

**Hollow subagent prompts.** Forgetting to include exact numbers from the paper in subagent prompts produces reviews that say "results are promising" without citing any actual results. Always reproduce key table rows in the prompt.

**Role collapse.** All 4 reviewers saying the same thing means the analytical focus questions were too generic. Each role's focus questions should reference different aspects of the paper. Domain reviewer asks about representation quality, experiments reviewer asks about baseline fairness, etc.

**Overclaiming consensus.** Two reviewers mentioning "speed" in different contexts does not make it a consensus point. Match on the specific technical claim, not keywords.

**Venue format mismatch.** ECCV has Major/Minor weaknesses split; NeurIPS does not. CVPR uses 1-10 scale; ECCV uses 1-6. Always check `references/venue-formats.md` before formatting output.

**Score-tone inconsistency.** A Weak Accept score with language that reads like a Reject ("fundamental issues", "critically flawed") confuses authors and ACs. Calibrate language to match the numeric score.

**Field content mismatch.** The most embarrassing mistake: pasting Minor Weaknesses content into the Strengths field, or vice versa. Phase 6's completeness check exists to catch this. Always verify before submission.

**Unrealistic rebuttal demands.** Asking authors to "retrain with a larger model", "collect a new dataset", or "run experiments at 512×512 resolution" in a 7-day window is unreasonable and signals inexperience. Stick to: reporting existing numbers (timing, F1, grid resolution), running 2-3 extra seeds, adding a table row from existing data, or providing a clarifying analysis.

**Numbers in the wrong places.** Paper Summary and Strengths should not contain specific numeric comparisons. Readers can look at the tables. Save numbers for Major Weaknesses where you need them to make a precise technical argument (e.g., "the 2.28 dB gap to oracle occupancy in Table 2 suggests...").
