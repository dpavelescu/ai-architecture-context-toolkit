---
name: ai-context-check
description: >-
  Check whether a story, analysis, implementation plan, pull request, diff, or
  solution note aligns with the approved AI Architecture Context and AI Coding
  Guidelines. A planning-time and review-time governance check that catches locally
  reasonable but directionally wrong solutions. Use before implementation when
  possible, or during PR review. Not for first-time context setup (use
  ai-context-bootstrap) or guidance evolution (use ai-guidance-update).
---

# Skill: ai-context-check

## Purpose

Check whether a story, analysis, implementation plan, pull request, diff, or solution
note is aligned with the approved AI Architecture Context and AI Coding Guidelines.
Acts as a planning-time and review-time architecture governance check, preventing
locally reasonable solutions from violating architecture intent.

## When to use

Reviewing a Jira story, Story Artifact, AI-generated analysis, implementation plan,
pull request, code diff, solution note, or proposed reusable learning. Use **before**
implementation when possible; during PR review when needed; before guidance-update
analysis when a repeated issue appears.

## Invocation

```
/ai-context-check work=<story|artifact|plan|pr|diff|solution-note> mode=<interactive|headless|analyze-only>
```

Examples:

```
/ai-context-check work=JIRA-123 mode=analyze-only
/ai-context-check work=docs/plans/payment-events-plan.md mode=interactive
/ai-context-check work=PR-456 mode=analyze-only
```

Optional: `scope=<repository|service|module|bounded-context>`,
`focus=<architecture|coding|brownfield|contracts|security|all>`. If no mode is given,
use `analyze-only`.

## House rules (apply throughout)

1. **Discover first** — never ask the user to paste anything discoverable from the repo.
2. **One blocking question at a time** — non-blocking questions go in the report.
3. **No silent governance** — do not approve architecture exceptions.
4. **Classify evidence** — never treat existing code as approved intent unless an
   approved source confirms it.
5. **Durable output, read-only** — always produce the Context Alignment Report. This
   skill **never edits** the Context or Guidelines; only `ai-guidance-update` writes to
   them (with approval).
6. **Cite every finding** — each finding must name the rule or source it violates and the
   offending location (file:line or contract field). If you can't cite it, don't raise it.
7. **Right-size the review** — match effort to risk. A small, in-scope, low-risk change
   gets a short report (or a one-line "Ready"); skip phases with no impact. Reserve the
   full phase-by-phase check for changes touching boundaries, ownership, data, contracts,
   or security/privacy/audit/compliance. Use repo-relative paths.

## Phase 1 — Discover context

Read and classify: root AI instruction file; context manifest; AI Architecture
Context; AI Coding Guidelines; Brownfield Rule Cards; relevant SAD sections; relevant
ADRs; relevant formal specs; relevant code and tests; relevant solution notes
(supporting memory only).

## Phase 2 — Understand the reviewed work

Identify: the work item; business intent; affected service / module / bounded context;
affected data ownership; affected API / event / UI contracts; affected security /
privacy / audit / compliance behavior; changed or proposed files; the implementation
pattern being used or proposed; current-vs-target implications; relevant Brownfield
Rule Cards. If intent is unclear and risk is material, ask one blocking question
(interactive only).

## Phase 3 — Architecture alignment check

Check against: ownership / service / module / bounded-context boundaries; data
ownership rules; integration rules; allowed coupling; prohibited shortcuts;
current-vs-target guidance; Brownfield Rule Cards; ask-first triggers.

Flag **locally reasonable but directionally wrong** solutions, e.g.:

- adding a synchronous service-to-service call because similar calls exist
- reading another service's database because legacy code does
- duplicating domain logic in the frontend
- bypassing an event contract
- writing audit data directly instead of via the approved mechanism
- copying legacy validation or error-handling that is no longer target

## Phase 4 — Coding guideline alignment check

Check against: repository structure; layering; naming; DTO / mapping / validation /
error-handling conventions; testing expectations; logging & observability; security /
privacy / audit / compliance coding rules; contract-change workflow; scope control.
Flag changes broader than the reviewed scope.

## Phase 5 — Contract & compliance check

Check impact on OpenAPI, AsyncAPI, UI specs, data specs, security, privacy, audit, and
compliance. If a formal spec should change but the work does not mention it, flag the
gap. If the work changes a contract without an approved source, flag a governance
issue.

## Phase 6 — Brownfield risk check

If the solution copies or extends known legacy / a tolerated workaround / a partial
migration / a local exception / suspected drift / current-but-not-target code,
classify the risk: acceptable preservation / risky expansion / migration-needed-but-
not-scoped / target-not-ready / ask-first / architecture-decision-required.

## Phase 7 — Coverage gap check (cross-cutting)

While running the checks above, watch for **coverage gaps**: the reviewed work depends on
a dimension the Context is **silent on**, and no source artifact (SAD, ADR, LLD,
security/privacy requirement, spec) covers it at an actionable level. Sweep the same
dimensions bootstrap uses — architecture style & modularity, ownership & boundaries, data
ownership, integration, API/event contracts, security, privacy, audit, compliance,
current-vs-target.

A coverage gap means the AI had to guess because the guidance was missing — not that the
work is wrong. For each gap, note **what guidance is missing** and **where it belongs**
(Context / SAD / ADR / requirement / spec), and recommend `ai-guidance-update` (plus a
source update when the gap belongs in an upstream artifact). Do not silently fill the gap.

## Phase 8 — Produce output

Produce a Context Alignment Report.

## Output format

```markdown
# Context Alignment Report

## Decision
Choose one: Ready | Ready with risks | Needs clarification | Blocked by architecture
decision | Requires guidance update analysis | Requires formal spec update |
Requires ADR or SAD update

## Reviewed input
- Type: / Reference: / Scope: / Mode:

## Summary
- <short summary>

## Architecture alignment
| Area | Status | Finding | Evidence |
|---|---|---|---|
(Status: aligned | risk | conflict | unclear | not applicable)

## Coding guideline alignment
| Area | Status | Finding | Evidence |
|---|---|---|---|

## Brownfield risks
| Pattern | Classification | Risk | Recommendation |
|---|---|---|---|

## Contract & compliance impact
| Area | Impact | Finding | Action |
|---|---|---|---|

## Source conflicts
| Conflict | Sources | Risk | Recommendation |
|---|---|---|---|

## Coverage gaps
| Dimension | Missing guidance | Where it belongs |
|---|---|---|
(Where it belongs: Context | SAD | ADR | requirement | spec)

## Blocking question
Ask exactly one question only if needed, or write: None.

## Non-blocking open points
- <open point>

## Recommended next action
Choose one: proceed | proceed with noted risks | clarify one blocking question |
update plan | update PR | run ai-guidance-update in analyze-only mode | raise
architecture decision | update formal spec | create or update Brownfield Rule Card
```

## Stop conditions

In `interactive` mode, stop and ask one blocking question (otherwise report the issue)
when:

- the solution requires an architecture decision
- ownership, data ownership, or contract authority is unclear
- security, privacy, audit, or compliance impact is unclear
- current code and target direction conflict
- the reviewed work violates an ask-first trigger
