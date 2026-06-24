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

## Invocation

```
/ai-context-check work=<story|artifact|plan|pr|diff|solution-note> [scope=<area>]
```

Examples:

```
/ai-context-check work=JIRA-123
/ai-context-check work=docs/plans/payment-events-plan.md
/ai-context-check work=PR-456
```

Optional: `scope=<area>` — a path, paths/glob, or a manifest `areas:` name; omit for the whole
repo. The check is read-only and interactive: it asks one blocking question only when something
genuinely needs a human, otherwise it records the question in its report.

## Constraints

1. **Discover first** — never ask the user to paste anything discoverable from the repo.
2. **One blocking question at a time** — non-blocking questions go in the report.
3. **No silent governance** — do not approve architecture exceptions.
4. **Classify evidence** — never treat existing code as approved intent unless an
   approved source confirms it.
5. **Durable output, read-only** — always produce the Context Alignment Report. This
   skill **never edits** the Context or Guidelines; only `ai-guidance-update` writes to
   them (with approval).
6. **Right-size the review** — delegate only the dimensions the work actually touches; a
   small, in-scope, low-risk change gets a short report (or a one-line "Ready"). Use
   repo-relative paths. (Per-finding citation is each reviewer's job — preserve their
   cited evidence in the report; don't add uncited findings.)

## Phase 1 — Discover context

Locate inputs with the **read-context-manifest** skill (manifest first, conventional
fallback), then read and classify: the AI Architecture Context; AI Coding Guidelines;
Brownfield Guardrails; relevant SAD sections, ADRs, and formal specs; relevant code and
tests; relevant solution notes (supporting memory only).

## Phase 2 — Understand the reviewed work

Identify: the work item; business intent; affected service / module / bounded context;
affected data ownership; affected API / event / UI contracts; affected security /
privacy / audit / compliance behavior; changed or proposed files; the implementation
pattern being used or proposed; current-vs-target implications; relevant Brownfield
Guardrails. Ask one blocking question (otherwise record it in the report) when intent is unclear
and risk is material, or when: the solution requires an architecture
decision; ownership, data ownership, or contract authority is unclear; security, privacy,
audit, or compliance impact is unclear; current code and target direction conflict; or the
reviewed work violates an ask-first trigger.

## Phase 3 — Delegate the dimension reviews

For each dimension the work actually touches (right-size — skip the rest): **assemble that
reviewer's input packet** — the relevant Context, Guidelines, Guardrails, SAD/ADRs/specs, and
code evidence — then delegate to its reviewer sub-agent; run them in parallel. Each reviewer owns
its dimension's checks and returns cited findings — do **not** re-run their logic here.

| Dimension the work touches | Reviewer |
|---|---|
| boundaries, ownership, data ownership, coupling, sync-vs-async, API/event ownership, dependency direction | `architecture-boundary-reviewer` |
| repo structure, layering, naming, DTO/mapping/validation/error-handling, tests, logging/observability, scope control | `engineering-convention-reviewer` |
| API/event/data/UI contract changes & backward-compat, security, privacy, audit, compliance | `contract-compliance-reviewer` |
| current-vs-target divergence, copying/extending legacy, conflicts between sources | `brownfield-governance-reviewer` |

If you're not running sub-agents (lighter pilot), apply that reviewer file's criteria
inline — the reviewer file is the single source for the dimension's checks either way.

## Phase 4 — Coverage gap check (cross-cutting)

While running the checks above, apply the **assess-coverage** skill over the full concern
checklist to watch for **coverage gaps**: the reviewed work depends on a concern the Context is
**silent on**, and no source artifact (SAD, ADR, LLD, security/privacy requirement, spec)
covers it at an actionable level.

A coverage gap means the AI had to guess because the guidance was missing — not that the
work is wrong. For each gap, note **what guidance is missing** and **where it belongs**
(Context / SAD / ADR / requirement / spec), and recommend `ai-guidance-update` (citing the
specific finding as its `source`; plus a source update when the gap belongs in an upstream
artifact). Do not silently fill the gap.

## Phase 5 — Produce output

Synthesize the reviewers' findings into one Context Alignment Report — each section below
is populated from the matching reviewer.

## Output format

```markdown
# Context Alignment Report

## Decision
Choose one: Ready | Ready with risks | Needs clarification | Blocked by architecture
decision | Requires guidance update analysis | Requires formal spec update |
Requires ADR or SAD update

## Reviewed input
- Type: / Reference: / Scope:

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
| Concern | Missing guidance | Where it belongs |
|---|---|---|
(Where it belongs: Context | SAD | ADR | requirement | spec)

## Blocking question
Ask exactly one question only if needed, or write: None.

## Non-blocking open points
- <open point>

## Recommended next action
Choose one: proceed | proceed with noted risks | clarify one blocking question |
update plan | update PR | run ai-guidance-update in analyze-only mode | raise
architecture decision | update formal spec | create or update Brownfield Guardrail
```
