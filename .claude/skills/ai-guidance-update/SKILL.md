---
name: ai-guidance-update
description: >-
  Decides whether a learning becomes a rule and where, then applies the smallest approved change to
  the AI Architecture Rules, Coding Guidelines, or Brownfield Guardrails. Takes a filled-in
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
  AI-facing layer (Architecture Rules, Guidelines, Guardrails, candidate solution notes). The
  Architecture Rules never originate a decision: for a new direction with no approved source, propose ADR-first,
  or a provisional Brownfield Guardrail via **write-brownfield-guardrail**

## Process

Each phase runs until its **Complete when** holds; don't enter the next phase before it does.

### Phase 1 — Discover current guidance

**Goal** — The run knows whether a baseline exists to update at all, and what it says today.

**Procedure** — Load current guidance and the sources the run works from with the
**read-source-map** skill, bounded by `scope`: the `guidance` (Architecture Rules, Coding
Guidelines, Brownfield Guardrails); the `ledger`; the relevant `sources` (SAD sections, ADRs, formal specs, code evidence); `memory` (solution notes); and
the source learning itself. When `source=clarification-decision`, read the ledger's
`## Open` items and their filled `decision:` lines.

#### When no baseline exists (bootstrap not yet run)

If no AI Architecture Rules or Coding Guidelines are found:

- A learning whose home is an **ADR / formal spec / Jira / Story Artifact** → proceed
  normally: **recommend (and optionally draft)** that target.
- A learning whose home is the **Architecture Rules / Guidelines / Guardrails** → **stop and report
  Decision = `Blocked`, recommending `ai-context-bootstrap` first**; don't fabricate a
  single-rule Architecture Rules file. Optionally record the learning as a **candidate solution note**
  so it isn't lost.

**Complete when** current `guidance`, the `ledger`, and the `sources` the conflict check will need
are each resolved or confirmed absent.

### Phase 2 — Classify the learning

**Goal** — The learning sits at the right altitude.

**Procedure** — One of: story-specific decision / implementation detail / reusable coding
convention / architecture rule / brownfield ambiguity / contract change / security rule / privacy
rule / audit rule / compliance rule / reference implementation / candidate memory /
suspected drift / conflict between sources.

**Complete when** the learning carries exactly one classification from that list — not two
candidates, not a hedge.

### Phase 3 — Decide the target artifact

**Goal** — The learning points at the one artifact that should carry it.

**Procedure** — Recommend one target and follow these routing rules:

| Learning | Target |
|---|---|
| Story-specific decision | Story Artifact / Jira |
| Implementation detail | PR / plan |
| Reusable coding convention | AI Coding Guidelines |
| Architecture rule (changes AI behavior) | AI Architecture Rules |
| Decision rationale | ADR |
| Contract change | Formal spec |
| Candidate / unproven learning | Solution note only |
| Candidate memory | Solution note only |
| Repeated brownfield ambiguity | Brownfield Guardrail |
| Security / privacy / audit / compliance rule | AI Architecture Rules (human approval required) |
| Reference implementation | AI Coding Guidelines (Reference implementations) |
| Suspected drift | Conflict finding, no update |
| Conflict between sources | Conflict finding, no update |

When the target is a Brownfield Guardrail, pass the divergence to the **write-brownfield-guardrail**
skill and route what it returns into the `Suggested minimal update` output; when it's the Architecture Rules or
Guidelines, follow the **write-guidance-file** skill.

**Complete when** exactly one recommended target follows from the classification — not two
candidates, not a hedge.

### Phase 4 — Conflict check

**Goal** — Whether the proposed update can coexist with what is already approved is known before
anything is written.

**Procedure** — Check the proposed update against: requirements; Story Artifact; formal specs;
ADRs; SAD; AI Architecture Rules; AI Coding Guidelines; Brownfield Guardrails; approved
reference implementation; current code; solution notes. If a conflict exists, do not
apply — produce a conflict finding.

**Complete when** the proposed update has been checked against every source in that list, with any
conflict recorded as a finding rather than applied.

### Phase 5 — Analyze-only output

**Goal** — The recommendation stands as a proposal a human can approve or refuse.

**Procedure** — In `analyze-only` mode, produce a Guidance Update Analysis and modify no files.

**Complete when** the Guidance Update Analysis is emitted with a Decision from the analyze-only enum
and no file has been modified.

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

### Phase 6 — Apply-approved-update behavior

**Goal** — The approved change lands in the guidance, and nothing beyond it does.

**Procedure** — Do not apply — stop and report — when: approval is missing (report Decision =
`Approval missing`); the target artifact is unclear; the update conflicts with formal specs, SAD, or ADRs;
it would require an architecture decision, or security / privacy / audit / compliance approval;
it would change contract truth; it is broader than the approved change; or the target is the
Architecture Rules/Guidelines but no baseline exists (recommend `ai-context-bootstrap`). Otherwise, in
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
each **accepted/edited** rule from the accepted text into the clean Architecture Rules or Guidelines (per the
classification) with **write-guidance-file**, merging into the existing file; then retire every
decided item with the **update-clarifications-ledger** skill (`resolve-decisions`). A
**code-vs-stale-source** conflict resolved in code's favour → write the corrected rule and flag the
upstream SAD/ADR/spec as stale (never rewrite it).

**Complete when** the approved change is applied at its smallest scope, every decided ledger item is
retired, and the Applied Guidance Update Report is emitted — or the run stopped and reported why.

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
