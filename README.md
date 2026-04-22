# paper-review

Multi-role academic peer review that produces submission-ready output matching venue form fields. Four expert reviewers cross-review independently, then a consensus synthesis produces calibrated, evidence-grounded reviews.

## When to Use

- You have a paper PDF and want a thorough mock review before submission
- Preparing for a venue deadline and want to stress-test weaknesses
- Acting as reviewer and want structured, multi-perspective analysis
- Need review output formatted for a specific venue (OpenReview/CMT/HotCRP)

## Usage

```
/paper-review                        # auto-detect from conversation
/paper-review path/to/paper.pdf      # review a specific PDF
/paper-review paper.pdf NeurIPS      # specify venue format
/paper-review paper.pdf ECCV 5       # specify venue + target score
```

## How it works

1. **Read & Extract** — full paper + supplementary, extract metadata, method pipeline, key numbers
2. **Assign 4 Expert Roles** — domain, modeling, experiments, systems (adapted per paper type)
3. **Parallel Independent Reviews** — 4 subagents review concurrently with self-contained prompts
4. **Cross-Review** — consensus matrix: what's agreed, majority, unique, or disagreed
5. **Synthesize** — venue-formatted final review with score-tone calibration
6. **Iterative Delivery** — field-by-field output with user confirmation at each step

## Supported Venues

| Platform | Venues | Score Scale |
|----------|--------|-------------|
| OpenReview | ECCV, NeurIPS, ICLR, EMNLP | 1-6 / 1-10 (varies) |
| CMT | CVPR, AAAI | 1-10 |
| HotCRP | ACL, NAACL | 1-5 |

Custom venue forms are also supported — paste the form and it adapts.

## Install

### Via skills.sh (recommended)

```bash
npx skills add freemty/paper-review
```

Works with Claude Code, Cursor, Codex, Windsurf, and [15+ other agents](https://skills.sh).

### Manual

```bash
git clone https://github.com/freemty/paper-review.git ~/.claude/skills/paper-review
```

## License

MIT
