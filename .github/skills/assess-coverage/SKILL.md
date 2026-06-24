---
name: assess-coverage
description: >-
  Decide how the AI Architecture Context should cover each relevant concern — point to an
  existing artifact, restate it actionably, flag it for clarification when the source is
  ambiguous, or fill-and-flag when nothing covers it. Use while drafting or refreshing the
  Context, or checking a work item for coverage gaps. Never resolve ambiguity by guessing — flag it.
---

Run over the **checklist below** (a baseline, not a ceiling) — checking what's **missing** as much
as what the sources mention. For each relevant concern that could be misinterpreted by an AI, choose
one — sized against the existing artifacts (SAD, ADRs, LLD, security/privacy requirements, specs):

- **Point** — an artifact covers it at an actionable level → reference it (must-read + a one-line operational pointer); don't restate.
- **Restate actionably** — covered but too abstract or buried → add a thin operational rule and link back.
- **Flag for clarification** — covered, but the source is ambiguous or admits more than one valid reading on something that matters → don't pick a reading; flag it for a human to state explicitly.
- **Flag or fill** — nothing covers it. If it's **architecturally significant**, flag it for clarification — don't invent a rule. For lower-risk concerns you may add a *proposed* rule (marked proposed/TBD). Either way record the gap (the SAD/ADR/requirement may need creating; never decide it silently).

Concerns (all equal — don't over-weight any one):

- ownership & boundaries; data ownership & access
- integration (sync/async; allowed/forbidden); API & event contracts
- security; data privacy / PII; audit; compliance
- technology & platform (languages/frameworks/runtimes/datastores; allowed/forbidden)
- architecture style & modularity (modular monolith / microservices-distributed / layered)
- resilience & error handling
- logging & observability — only where architecturally constrained
- current-vs-target (brownfield) divergences

**The list is a baseline, not a ceiling.** Also surface and cover any concern specific to this repo
or domain that could mislead an AI even if it isn't listed — e.g. performance/latency SLAs,
scalability, cost, multi-tenancy & data isolation, internationalization/accessibility, caching,
concurrency. Cover it as its own concern; never drop a real concern for being "off-list."

**Thin ≠ narrow:** cover every relevant concern, but where an artifact already covers one
well, shrink to a pointer. Skip a concern only when it's genuinely irrelevant — never
because it's "not architecture."
