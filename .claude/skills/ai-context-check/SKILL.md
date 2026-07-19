---
name: ai-context-check
description: >-
  Reviews a story, plan, PR, diff, or solution note against the approved AI Architecture Rules and
  AI Coding Guidelines, and reports each divergence — including the locally reasonable solution that
  runs against the architecture's direction. Delegates each touched dimension to a reviewer agent and
  synthesizes one alignment report. Runs at planning time or on a PR. Read-only — judges work against
  the guidance, doesn't create or change it.
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

Optional: `scope=<area>` — a path, paths/glob, or a source map `areas:` name; omit for the whole
repo.

## Constraints

1. **Discover first** — never ask the user to paste anything discoverable from the repo.
2. **No silent governance** — do not approve architecture exceptions.
3. **Classify evidence** — never treat existing code as approved intent unless an
   approved source confirms it.
4. **Read-only** — this skill **never edits** the Architecture Rules or Guidelines; only
   `ai-guidance-update` writes to them (with approval).
5. **Right-size the review** — a small, in-scope, low-risk change gets a short report (or
   a one-line "Ready"). Use repo-relative paths. Preserve each reviewer's cited evidence
   in the report; add no uncited findings.

## Process

Each phase runs until its **Complete when** holds; don't enter the next phase before it does.

### Phase 1 — Discover context

**Goal** — This review holds the guidance it judges against, with what is binding separated from what
is still open.

**Procedure** — Load the context this review works from with the **read-source-map** skill, bounded
by `scope`: its `guidance`, `ledger`, `sources`, and `memory`. The Architecture Rules/Guidelines are
authoritative; treat an `## Open` ledger item as **not yet binding** — a concern still awaiting
decision, not an approved rule.

**Complete when** each category — `guidance`, `ledger`, `sources` — is resolved or confirmed absent.

### Phase 2 — Understand the reviewed work

**Goal** — What the work is trying to do, and what it puts at risk, is understood.

**Procedure** — Identify: the work item; business intent; affected service / module / bounded context;
affected data ownership; affected API / event / UI contracts; affected security /
privacy / audit / compliance behavior; changed or proposed files; the implementation
pattern being used or proposed; current-vs-target implications; relevant Brownfield
Guardrails. Ask one blocking question (otherwise record it in the report) when intent is unclear
and risk is material, or when: the solution requires an architecture
decision; ownership, data ownership, or contract authority is unclear; security, privacy,
audit, or compliance impact is unclear; current code and target direction conflict; or the
reviewed work violates an ask-first trigger.

**Complete when** the affected service / module / bounded context, data ownership, contracts, and
security / privacy surface are each identified or explicitly established as not touched — that split
is what decides which dimensions get delegated.

### Phase 3 — Delegate the dimension reviews

**Goal** — Each dimension the work touches is judged by the reviewer that owns it, not by this run.

**Procedure** — For each dimension the work actually touches (right-size — skip the rest):
**assemble that reviewer's input packet** — the relevant Architecture Rules, Guidelines, Guardrails,
SAD/ADRs/specs, and code evidence — then delegate to its reviewer sub-agent; run them in parallel. Each reviewer owns
its dimension's checks and returns cited findings — do **not** re-run their logic here. If a delegated
reviewer fails or returns nothing, record that dimension as `unclear` in the report and continue —
don't silently drop it.

| Dimension the work touches | Reviewer |
|---|---|
| boundaries, ownership, data ownership, coupling, sync-vs-async, API/event ownership, dependency direction | `architecture-boundary-reviewer` |
| repo structure, layering, naming, DTO/mapping/validation/error-handling, tests, logging/observability, scope control | `engineering-convention-reviewer` |
| API/event/data/UI contract changes & backward-compat, security, privacy, audit, compliance | `contract-compliance-reviewer` |
| current-vs-target divergence, copying/extending legacy, conflicts between sources | `brownfield-governance-reviewer` |

If you're not running sub-agents, apply that reviewer file's criteria inline.

**Complete when** every dimension the work touches has either returned findings or been recorded
`unclear` — none left pending or dropped.

### Phase 4 — Coverage gap check

**Goal** — Concerns this work depends on that no approved source settles are surfaced rather than
answered here.

**Procedure** — Apply the **assess-coverage** skill against the existing `guidance` as baseline;
each concern it routes as **needs a decision** is a coverage gap for this report. If the work
depends on a concern that is an **open ledger item**, surface it as awaiting decision
(recommend ratifying it via `ai-guidance-update`) — don't treat the silence as approval.

For each gap, note **what guidance is missing** and **where it belongs**
(Architecture Rules / SAD / ADR / requirement / spec), and recommend `ai-guidance-update` (citing the
specific finding as its `source`; plus a source update when the gap belongs in an upstream
artifact). Do not silently fill the gap.

**Complete when** every concern the work depends on is either covered by approved guidance or
recorded as a gap with where it belongs — none silently filled.

### Phase 5 — Produce output

**Goal** — The reader gets one verdict on the work, backed by the reviewers' cited findings.

**Procedure** — Synthesize the reviewers' findings into one **Context Alignment Report** in the
**Output format**, mapping each reviewer's decision to the report `Decision`: governance-approval /
contract change → `Requires guidance update analysis` or `Requires formal spec update`; an undecided
architecture call
→ `Blocked by architecture decision` with `where it belongs: ADR`; an ADR/SAD change → `Requires ADR
or SAD update`; otherwise `Ready` / `Ready with risks` / `Needs clarification`.

**Complete when** one Context Alignment Report is emitted carrying a single `Decision`, with every
touched dimension represented in it.

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
(Where it belongs: Architecture Rules | SAD | ADR | requirement | spec)

## Blocking question
Ask exactly one question only if needed, or write: None.

## Non-blocking open points
- <open point>

## Recommended next action
Choose one: proceed | proceed with noted risks | clarify one blocking question |
update plan | update PR | run ai-guidance-update in analyze-only mode | raise
architecture decision | update formal spec | create or update Brownfield Guardrail
```
