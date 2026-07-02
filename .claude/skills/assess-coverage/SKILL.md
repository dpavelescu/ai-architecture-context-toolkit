---
name: assess-coverage
description: >-
  Decide, for each relevant concern, whether an approved source already settles it (→ a final rule
  for the clean context) or it needs a human decision (→ a proposal in the clarifications ledger).
  Surfaces only concerns that matter — variation already evidenced, high impact if they vary, or a
  cross-cutting/framework standardization — never a catalog of every existing pattern. Use while
  drafting or refreshing the Context/Guidelines, or checking a work item for coverage gaps.
---

**Relevance gate — surface only what matters.** Do not catalog existing patterns. Keep a concern only
if it clears at least one test:
- **Variation already evidenced** — the code is already inconsistent on it.
- **High impact if it varies** — cross-cutting and costly to get inconsistent (boundaries, contracts, security, data handling, error/result conventions).
- **General / framework-level standardization** — a cross-cutting concern where the framework choice should be locked in (validation, DI, logging, mapping, HTTP/resilience), even if currently uniform.

Drop any concern that is uniform AND low-impact AND local. Add repo/domain-specific concerns that clear
the gate even if unlisted (multi-tenancy & data isolation, performance/latency SLAs, i18n).

For each kept concern, check coverage against the sources in **authority order** (SAD, ADRs, specs,
diagrams; code lowest), and route it to exactly one outcome:

- **Settled by an approved source** — covered at an actionable level → emit a **final rule** for the clean context: a one-line imperative rule + a link to the owning source (if abstract or buried, restate it as a one-line rule and link back). No decision needed.
- **Needs a decision** — no approved source covers it, the source is ambiguous, sources conflict, or it is only code-evidenced → emit a **ledger candidate** (proposal + rationale, below). A code-derived proposal is lowest authority and never self-ratifies; nothing here enters the context until decided.

**Severity** decides where a candidate is raised: **critical** — security, privacy, compliance, data
ownership, or a needed architecture decision → ask live, offering defer-to-ledger; everything else →
ledger only.

**Concern checklist** (a baseline, not a ceiling):
- ownership & boundaries; data ownership & access
- integration (sync/async; allowed/forbidden); API & event contracts
- security; data privacy / PII; audit; compliance
- technology & platform (languages/frameworks/runtimes/datastores; allowed/forbidden)
- architecture style & modularity (modular monolith / microservices-distributed / layered)
- resilience & error handling
- logging & observability — only where architecturally constrained
- current-vs-target (brownfield) divergences

**Two output streams.** Final rules (for the clean Context/Guidelines): `<concern> · <imperative rule> · <source link>`. Ledger candidates (for the clarifications ledger), each:
```
[<concern>] Proposal: <recommended rule>.
why: <evidence of variation / impact / framework standardization>.
raise: <live | ledger>   (critical → live; all others → ledger)
decision:
```
