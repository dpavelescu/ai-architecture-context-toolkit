---
name: assess-coverage
description: >-
  Decide, for each relevant concern, whether an approved source already settles it (→ a final rule
  for the clean context) or it needs a human decision (→ a proposal in the clarifications ledger).
  Surfaces only concerns that matter — variation already evidenced, high impact if they vary, or a
  cross-cutting/framework standardization — never a catalog of every existing pattern. Runs while the
  Architecture Rules/Guidelines are being written or refreshed, and while a work item is checked for coverage
  gaps. Decides nothing itself: every candidate it emits waits on a human.
---

## Inputs

- **sources** — the sources `read-source-map` already selected as relevant to these concerns, in authority order: SAD, ADRs, specs, diagrams; plus code evidence (lowest authority). Per concern, decide whether a source settles it (→ final rule) or leaves it open (→ candidate).
- **baseline** — the existing approved rules, when guidance already exists. Absent on a first pass.

## Procedure

1. **Relevance gate — surface only what matters.** Do not catalog existing patterns. Walk the concern checklist. Keep a concern only if it clears at least one test:
   - **Variation already evidenced** — the code is already inconsistent on it.
   - **High impact if it varies** — cross-cutting and costly to get inconsistent (boundaries, contracts, security, data handling, error/result conventions).
   - **General / framework-level standardization** — a cross-cutting concern where the framework choice should be locked in (validation, DI, logging, mapping, HTTP/resilience), even if currently uniform.

   Drop any concern that is uniform AND low-impact AND local.
2. **Add repo/domain-specific concerns** that clear the gate even if unlisted (e.g. multi-tenancy & data isolation, performance/latency SLAs, i18n).
3. For each kept concern, check coverage against the sources in authority order.
4. **Route each kept concern to exactly one outcome:**
   - **Settled by an approved source** — covered at an actionable level → emit a **final rule** for the clean context: a one-line imperative rule + a link to the owning source. If covered but abstract or buried, restate it as a one-line imperative rule and link the owning source. No decision needed.
   - **Needs a decision** — no approved source covers it, the source is ambiguous, sources conflict, or it is only code-evidenced → emit a **ledger candidate** in the shape the Output gives (proposal + rationale). Authority travels with it: a code-derived proposal is lowest authority and never self-ratifies. Nothing here enters the context until decided.
5. **With a `baseline`, assess it too.** An existing rule is a concern in its own right where a source now contradicts it, no source supports it any more, or the code has drifted from it → emit a **ledger candidate** proposing the correction or the retirement. Never silently drop or rewrite an approved rule; a baseline rule its source still supports stays a **final rule** and needs no decision.
6. **Severity of each candidate** decides where it is raised: **critical** — security, privacy, compliance, data ownership, or a needed architecture decision → ask live, offering defer-to-ledger; everything else → ledger only.

## Concern checklist

- ownership & boundaries; data ownership & access
- integration (sync/async; allowed/forbidden); API & event contracts
- security; data privacy / PII; audit; compliance
- technology & platform (languages/frameworks/runtimes/datastores; allowed/forbidden)
- architecture style & modularity (modular monolith / microservices-distributed / layered)
- resilience & error handling
- logging & observability — only where architecturally constrained
- current-vs-target (brownfield) divergences

## Output

Two streams.

**Final rules** (settled from an approved source) — for the clean Architecture Rules/Guidelines:

```
<concern> · <imperative rule> · <source link>
```

**Ledger candidates** (need a decision) — for the clarifications ledger, each:

```
[<concern>] Proposal: <recommended rule>.
why: <evidence of variation / impact / framework standardization>.
raise: <live | ledger>   (critical → live; all others → ledger)
decision:
```
