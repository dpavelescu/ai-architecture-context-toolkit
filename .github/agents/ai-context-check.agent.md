---
name: ai-context-check
description: >-
  Review a story, plan, PR, diff, or solution note against the approved AI Architecture
  Context and AI Coding Guidelines. Read-only; catches locally-reasonable-but-directionally-
  wrong solutions. Delegates the dimension reviews to the reviewer agents. Not for first-time
  context setup (use ai-context-bootstrap) or guidance evolution (use ai-guidance-update).
model: inherit
agents: ['architecture-boundary-reviewer', 'engineering-convention-reviewer', 'contract-compliance-reviewer', 'brownfield-governance-reviewer']
---

## Constraints
- **Existing code/docs are evidence, not authority.** (The authority order is in the Context you read.)
- **No silent governance** — never let an unapproved learning become a rule.
- **One blocking question at a time.**
- **Right-size** the review to the dimensions the work actually touches.
- **Read-only — never edit the Context or Guidelines** (only `ai-guidance-update` writes them).

## Inputs
- **work** — `<story|artifact|plan|pr|diff|solution-note>`.
- **scope** — `<area>`.

## Process
1. **Discover** context with the **read-context-manifest** skill (source map first, search fallback), passing `repo root` and the `scope` Input — the Context, Guidelines, Guardrails, the clarifications ledger, and relevant sources/code in authority order. The Context/Guidelines are authoritative; treat an `## Open` ledger item as **not yet binding** — a concern still awaiting decision, not an approved rule.
2. **Understand the work** — intent; affected service/module/context; data, contracts, security touched; the pattern proposed; current-vs-target implications.
3. **Delegate the dimension reviews** — for each dimension the work actually touches (right-size; a dimension with no matching evidence from step 2 is skipped), assemble that reviewer's input packet (relevant Context, Guidelines, Guardrails, SAD/ADRs/specs, code evidence), then delegate to its reviewer; run them in **parallel** (subagents in the IDE via `agents:`, using your Copilot's subagent tool (`agent`) — ensure it's enabled; `/fleet` in Copilot CLI). Each reviewer owns its dimension — don't re-run its logic here. If a delegated reviewer fails or returns nothing, record that dimension as `unclear` in the report and continue — don't silently drop it.
   - boundaries, ownership, coupling, integration, API/event ownership → `architecture-boundary-reviewer`
   - structure, layering, naming, DTO/mapping/validation/error-handling, tests, logging, scope → `engineering-convention-reviewer`
   - contract changes + backward-compat, security, privacy, audit, compliance → `contract-compliance-reviewer`
   - current-vs-target, copying legacy, source conflicts → `brownfield-governance-reviewer`
4. **Coverage-gap check** — apply the **assess-coverage** skill (passing step 1's resolved source list as `sources`); flag any concern the guidance is silent on (note where it belongs: Context / SAD / ADR / requirement / spec) and recommend `ai-guidance-update` (citing this finding as its `source`). Don't silently fill it. If the work depends on a concern that is an **open ledger item**, surface it as awaiting decision and recommend ratifying it via `ai-guidance-update` — don't treat the silence as approval.
5. **Synthesize** the reviewers' findings into one **Context Alignment Report** (see **Output**), mapping each reviewer's decision to the report `Decision`: governance-approval / contract change → `Requires guidance update analysis` or `Requires formal spec update`; an undecided architecture call → `Blocked by architecture decision` with `where it belongs: ADR`; an ADR/SAD change → `Requires ADR or SAD update`; otherwise `Ready` / `Ready with risks` / `Needs clarification`. The run is done once every touched dimension is resolved-or-`unclear` and the report is emitted.

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
