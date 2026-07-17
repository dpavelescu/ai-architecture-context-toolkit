---
name: ai-guidance-update
description: >-
  Decides whether a learning becomes a rule and where, then applies the smallest approved change to
  the AI Architecture Context, Coding Guidelines, or Brownfield Guardrails. Takes a filled-in
  clarifications ledger, a review finding, a wrong AI assumption, or a changed ADR/spec.
  Analyze-only by default; writes only on explicit approval, and never approves on its own.
---

# Skill: ai-guidance-update

## Invocation

```
# analyze only (default)
/ai-guidance-update source=<clarification-decision|learning|solution-note|pr-finding|review-issue|adr|spec-change> mode=analyze-only

# apply an already-approved update (a filled-in clarifications ledger counts as approval)
/ai-guidance-update source=<clarification-decision|approved-update> mode=apply-approved-update
```

Examples:

```
/ai-guidance-update source=docs/solutions/payment-status-event-versioning.md mode=analyze-only
/ai-guidance-update source=PR-456-review-finding mode=analyze-only
/ai-guidance-update source=approved-guidance-update-2026-06-13 mode=apply-approved-update
/ai-guidance-update source=docs/architecture/ai-clarifications.md mode=apply-approved-update
```

If no mode is given, use `analyze-only`.

## Constraints

- never auto-promote solution notes to approved guidance
- prefer "no update needed" over adding guidance that won't change future AI behavior
- never copy a source's detail into a Guardrail or solution note; link to it
- never write into a SAD / ADR / spec — or a tracker item (Jira / Story Artifacts): flag that
  one needs changing, or draft proposed text for a human to own. This skill writes only the
  AI-facing layer (Context, Guidelines, Guardrails, candidate solution notes). The Context
  never originates a decision: for a new direction with no approved source, propose ADR-first,
  or a provisional Brownfield Guardrail via **write-brownfield-guardrail**

## Phase 1 — Discover current guidance

Locate current guidance with the **read-source-map** skill (`repo root`, `scope`); its `sources` feed
the conflict check. Then read: the `guidance` (Context, Coding Guidelines, Brownfield Guardrails);
the `ledger`; the relevant `sources` (SAD sections, ADRs, formal specs, code evidence); `memory`
(solution notes); and the source learning itself. When `source=clarification-decision`, read the ledger's
`## Open` items and their filled `decision:` lines — each filled decision is the approval.

### When no baseline exists (bootstrap not yet run)

If no AI Architecture Context or Coding Guidelines are found:

- A learning whose home is an **ADR / formal spec / Jira / Story Artifact** → proceed
  normally: **recommend (and optionally draft)** that target.
- A learning whose home is the **Context / Guidelines / Guardrails** → **stop and report
  Decision = `Blocked`, recommending `ai-context-bootstrap` first**; don't fabricate a
  single-rule Context. Optionally record the learning as a **candidate solution note** so it
  isn't lost.

## Phase 2 — Classify the learning

One of: story-specific decision / implementation detail / reusable coding convention /
architecture rule / brownfield ambiguity / contract change / security rule / privacy
rule / audit rule / compliance rule / reference implementation / candidate memory /
suspected drift / conflict between sources.

## Phase 3 — Decide the target artifact

Recommend one target and follow these routing rules:

| Learning | Target |
|---|---|
| Story-specific decision | Story Artifact / Jira |
| Implementation detail | PR / plan |
| Reusable coding convention | AI Coding Guidelines |
| Architecture rule (changes AI behavior) | AI Architecture Context |
| Decision rationale | ADR |
| Contract change | Formal spec |
| Candidate / unproven learning | Solution note only |
| Candidate memory | Solution note only |
| Repeated brownfield ambiguity | Brownfield Guardrail |
| Security / privacy / audit / compliance rule | AI Architecture Context (human approval required) |
| Reference implementation | AI Coding Guidelines (Reference implementations) |
| Suspected drift | Conflict finding, no update |
| Conflict between sources | Conflict finding, no update |

When the target is a Brownfield Guardrail, apply the **write-brownfield-guardrail** skill
(`trigger` = the divergence) and route the returned Guardrail into the `Suggested minimal
update` output; when it's the Context or Guidelines, follow the **write-guidance-file** skill.

## Phase 4 — Conflict check

Check the proposed update against: requirements; Story Artifact; formal specs; ADRs;
SAD; AI Architecture Context; AI Coding Guidelines; Brownfield Guardrails; approved
reference implementation; current code; solution notes. If a conflict exists, do not
apply — produce a conflict finding.

## Phase 5 — Analyze-only output

In `analyze-only` mode, produce a Guidance Update Analysis and modify no files.

```markdown
# Guidance Update Analysis

## Decision
Choose one: No update needed | Candidate update | Human approval required |
Conflict detected | Apply-ready, approval already explicit | Blocked

## Source
- Type: / Reference: / Summary:

## Learning classification
- <classification>

## Recommended target
- <target artifact>

## Reason
- <reason>

## Existing guidance affected
- <file and section>

## Conflict check
| Source | Conflict? | Finding |
|---|---|---|

## Human approval required
Yes / No — required owner: Architect | Tech Lead | Product Owner | Security | Privacy |
Compliance | QA | Other

## Suggested minimal update
<the smallest possible text change>

## Not included
<related but intentionally excluded changes>

## Recommended next action
Choose one: no action | keep as solution note only | request human approval |
create ADR | update SAD | update formal spec | apply approved update |
create or update Brownfield Guardrail
```

## Phase 6 — Apply-approved-update behavior

Do not apply — stop and report — when: approval is missing (report Decision = `Approval
missing`); the target artifact is unclear; the update conflicts with formal specs, SAD, or ADRs;
it would require an architecture decision, or security / privacy / audit / compliance approval;
it would change contract truth; it is broader than the approved change; or the target is the
Context/Guidelines but no baseline exists (recommend `ai-context-bootstrap`). Otherwise, in
`apply-approved-update` mode:

1. Verify explicit approval exists.
2. Verify the approved target artifact.
3. Verify the approved text or intent.
4. Apply the smallest possible change.
5. Do not update unrelated sections.
6. Preserve links to approved sources.
7. Add or update the source reference.
8. Flag any conflict discovered during application.
9. Produce an Applied Guidance Update Report.

**From the ledger** (`source=clarification-decision`): each filled `decision:` is the approval. Fold
each **accepted/edited** rule into the clean Context or Guidelines via **write-guidance-file**
(passing `target-file` = Context or Guidelines per the Phase 2 classification,
`coverage-decisions` = the accepted rule, `sources` = Phase 1's `sources`, `baseline` = the existing
file), then retire every decided item with the **update-clarifications-ledger** skill (`operation` =
`resolve-decisions`, `decisions` = the decided entries, `ledger-path` from Phase 1). A
**code-vs-stale-source** conflict resolved in code's favour → write the corrected rule and flag the
upstream SAD/ADR/spec as stale (never rewrite it).

```markdown
# Applied Guidance Update Report

## Decision
Choose one: Applied | Not applied | Partially applied | Approval missing | Blocked

## Files changed
| File | Section | Change |
|---|---|---|

## Approval source
- <approval reference>

## Summary of applied change
- <summary>

## Conflicts detected during application
- <conflict or none>

## Follow-up recommended
- <follow-up or none>
```
