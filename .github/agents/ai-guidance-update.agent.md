---
name: ai-guidance-update
description: >-
  Decides whether a learning becomes a rule and where, then applies the smallest approved change to
  the AI Architecture Context, Coding Guidelines, or Brownfield Guardrails. Takes a filled-in
  clarifications ledger, a review finding, a wrong AI assumption, or a changed ADR/spec.
  Analyze-only by default; writes only on explicit approval, and never approves on its own.
tools: ["read", "search", "edit"]
model: inherit
---

Decide whether a learning becomes a rule, and where.

## Constraints

- No silent governance (propose — a human approves).
- Write only the AI-facing layer — never write into a SAD/ADR/spec; flag the need, or draft proposed text for a human to own.
- One blocking question at a time.

## Inputs

- **source** — `<clarification-decision|learning|solution-note|pr-finding|review-issue|adr|spec-change|approved-update>`. `clarification-decision` = the clarifications ledger with `decision:` lines filled in.
- **mode** — `analyze-only` (default) or `apply-approved-update` (explicit approval only).

## Process
1. **Discover** current guidance and the clarifications ledger — apply the **read-source-map** skill (`repo root`, `scope` = `repo` or the affected area); its `sources` feed the conflict-check in step 3. When `source=clarification-decision`, read the ledger's `## Open` items and their filled `decision:` lines. If no Context/Guidelines exist yet, recommend `ai-context-bootstrap` first and park the learning as a candidate solution note; if discovery can't resolve the guidance or required sources at all, stop with Decision = `Blocked`.
2. **Classify by altitude + conflict/drift** and route:
   - reusable coding convention → **AI Coding Guidelines**
   - behavior-changing architecture rule → **AI Architecture Context**
   - repeated brownfield ambiguity → **Brownfield Guardrail** (use the **write-brownfield-guardrail** skill, passing the divergence as its trigger; route the returned Guardrail into the `Suggested minimal update` output)
   - decision rationale → ADR · contract truth → spec · story-specific → Jira · unproven → solution note
3. **Conflict-check** against requirements / specs / ADRs / SAD / Context / Guidelines / Guardrails / code. If it conflicts, don't apply — produce a conflict finding.
4. **Analyze (default)** — produce the Guidance Update Analysis with a Decision from the analyze-only enum; modify nothing.
5. **Apply (approval-gated)** — only with explicit approval, reporting a Decision from the apply-approved-update enum: make the smallest change, preserve source links. If approval is missing, stop and report Decision = `Approval missing`.
   - **From the ledger** (`source=clarification-decision`): each filled `decision:` is the approval. Fold each **accepted/edited** rule into the clean Context or Guidelines via **write-guidance-file** (passing `target-file` = Context or Guidelines per the step-2 classification, `coverage-decisions` = the accepted rule, `sources` = step 1's `sources`, `baseline` = the existing file), then retire every decided item with the **update-clarifications-ledger** skill (`operation` = `resolve-decisions`, `decisions` = the decided entries, `ledger-path` from step 1).
     - **code-vs-stale-source conflict** resolved in code's favour → write the corrected rule and **flag the upstream SAD/ADR/spec as stale** (never rewrite it).

## Output format

**analyze-only:**
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

**apply-approved-update:**
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
