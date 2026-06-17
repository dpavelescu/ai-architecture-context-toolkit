---
name: assess-coverage
description: >-
  Decide, per dimension, how the AI Architecture Context covers it — point to an existing
  artifact, restate it actionably, or fill-and-flag a gap. Use when drafting or refreshing
  the Context, or when checking a work item for coverage gaps.
---

For each dimension that is relevant to the work **and** could be misinterpreted by an AI,
choose one — sized against the existing artifacts (SAD, ADRs, LLD, security/privacy
requirements, specs):

- **Point** — an artifact covers it at an actionable level → reference it (must-read + a one-line operational pointer); don't restate.
- **Restate actionably** — covered but too abstract or buried → add a thin operational rule and link back.
- **Fill and flag** — not covered anywhere (or only in code) → capture the operational rule and record a gap (the SAD/ADR/requirement may need creating or updating; never decide it silently).

Dimensions (all equal — don't over-weight any one):

- ownership & boundaries; data ownership & access
- integration (sync/async; allowed/forbidden); API & event contracts
- security; data privacy / PII; audit; compliance
- architecture style & modularity (modular monolith / microservices-distributed / layered)
- error handling / resilience, logging / observability — only where architecturally constrained
- current-vs-target (brownfield) divergences

**Thin ≠ narrow:** cover every relevant dimension, but where an artifact already covers one
well, shrink to a pointer. Skip a dimension only when it's genuinely irrelevant — never
because it's "not architecture."
