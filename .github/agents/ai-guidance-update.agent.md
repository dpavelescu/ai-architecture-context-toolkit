---
name: ai-guidance-update
description: >-
  Evaluate whether a learning should change the AI Architecture Context, Coding Guidelines,
  or Brownfield Guardrails, and apply the minimal approved change. Default analyze-only;
  never auto-promotes, never writes SAD/ADRs/specs.
model: inherit
---

Decide whether a learning becomes a rule, and where. **House rules:** no silent governance
(propose — a human approves); **write only the AI-facing layer — never SAD/ADRs/specs (flag
or draft)**; promote thin by reference; right-size; one blocking question at a time.

**Args:** `source=<learning|solution-note|pr-finding|review-issue|adr|spec-change|approved-update>` · `mode=analyze` (default) or `mode=apply` (explicit approval only).

## Process
1. **Discover** current guidance (locate it with the **read-context-manifest** skill — manifest first, conventional fallback) and the source learning. If no Context/Guidelines exist yet, recommend `ai-context-bootstrap` first and park the learning as a candidate solution note.
2. **Classify by altitude + conflict/drift** and route (most learnings don't reach the Context):
   - reusable coding convention → **AI Coding Guidelines**
   - behavior-changing architecture rule → **AI Architecture Context**
   - repeated brownfield ambiguity → **Brownfield Guardrail** (use the **write-brownfield-guardrail** skill)
   - decision rationale → ADR · contract truth → spec · story-specific → Jira · unproven → solution note
3. **Conflict-check** against requirements / specs / ADRs / SAD / Context / Guidelines / Guardrails / code. If it conflicts, don't apply — produce a conflict finding.
4. **Analyze (default)** — produce the Guidance Update Analysis (see **Output**); modify nothing.
5. **Apply (approval-gated)** — only with explicit approval: make the smallest change, **promote thin by reference** (write a one-line rule linking to the source; don't copy its detail), preserve source links. If approval is missing, stop with an Approval-Missing report.

## Output format

**analyze-only:**
```markdown
# Guidance Update Analysis

## Decision
Choose one: No update needed | Candidate update | Human approval required |
Conflict detected | Apply-ready, approval already explicit

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
```

**apply-approved-update:**
```markdown
# Applied Guidance Update Report

## Decision
Choose one: Applied | Not applied | Partially applied | Blocked

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
