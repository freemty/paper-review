# Reviewer Role Templates

## Role contract

Supply paper title, venue/contribution type if known, accessible paper/supplement
paths, coverage limitations, and the question this lens should answer. A reviewer
can use available native readers or text/page artifacts directly. Do not assume
it cannot open a PDF, and do not replace source access with a huge summary.

Return supported strengths, material weaknesses, precise locators, unresolved
questions and confidence. Role count and number of findings depend on the paper.
Do not manufacture issues to fill a quota. A role may run in the main thread when
independent delegation is unavailable. No role submits or archives the review.

## Role-Specific Focus Areas

### Domain Expert (Role A)
- Is the representation/formulation well-designed for this domain?
- How does it compare to state-of-the-art approaches in the specific subfield?
- Are the qualitative results convincing from a domain perspective?
- Does the approach have practical potential in the domain?

### Modeling/Algorithm Expert (Role B)
- Is the core algorithmic contribution novel or incremental?
- Are the design choices (architecture, loss, training strategy) well-motivated?
- How does it relate to concurrent/recent works in the same paradigm?
- Are there fundamental limitations in the formulation?

### Experimental Methodology Expert (Role C)
- Are the baselines appropriate and up-to-date?
- Are the metrics comprehensive and correctly computed?
- Is there statistical rigor (error bars, multiple runs, sufficient samples)?
- Are there missing experiments that would strengthen the claims?
- Is there a gap between what the paper claims and what the evidence supports?

### Systems/Scalability Expert (Role D)
- What are the computational costs (training and inference)?
- How does efficiency compare to alternatives?
- Are there scalability bottlenecks?
- Is the approach practical for real-world deployment?
- Are there obvious optimization paths the authors haven't discussed?

## Contribution Type Adjustments

### Concept & Feasibility
- Do NOT penalize for: lacking large-scale experiments, not achieving SOTA, missing exhaustive ablations, limited deployment
- DO evaluate: novelty/vision, correctness/soundness, claim-evidence alignment, feasibility validation quality, clarity/framing

### Algorithms / Systems
- Full experimental rigor expected
- Baselines should be comprehensive and current
- Ablations should isolate key design choices

### Datasets / Benchmarks
- Focus on: annotation quality, diversity, bias analysis, utility for the community
- Less emphasis on: novel methods built on top of the dataset
