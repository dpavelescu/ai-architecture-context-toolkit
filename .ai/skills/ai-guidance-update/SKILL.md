---
name: ai-guidance-update
description: >-
  Analyze and apply controlled, approved updates to the AI Architecture Context, AI
  Coding Guidelines, and Brownfield Rule Cards — so useful learnings don't die in chat,
  PRs, or solution notes, and unapproved learnings don't silently become governance
  rules. Use when the AI repeatedly makes the same wrong assumption, reviewers keep
  correcting the same issue, a brownfield pattern misleads AI, a target direction
  becomes clear, an ADR/spec changes, or a reusable learning is found. Default mode is
  analyze-only; never applies governance-impacting updates without explicit approval.
---

# Skill: ai-guidance-update

## Purpose

Analyze and apply controlled updates to the AI Architecture Context, AI Coding
Guidelines, and Brownfield Rule Cards. Prevents useful learnings from staying only in
chat history, PR comments, or solution notes — and prevents unapproved learnings from
silently becoming governance rules.

## When to use

- the AI repeatedly makes the same wrong assumption
- reviewers repeatedly correct the same issue
- a brownfield pattern misleads AI
- a target direction becomes clear
- an ADR changes implementation behavior, or a formal spec changes
- a reference implementation becomes approved
- a solution note contains reusable learning
- a current pattern should no longer be copied
- a guidance conflict is detected

Do not use for story-specific details unless the learning affects future AI behavior.

## Invocation

```
# analyze only (default)
/ai-guidance-update source=<learning|solution-note|pr-finding|review-issue|adr|spec-change> mode=analyze-only

# apply an already-approved update
/ai-guidance-update source=<approved-update> mode=apply-approved-update
```

Examples:

```
/ai-guidance-update source=docs/solutions/payment-status-event-versioning.md mode=analyze-only
/ai-guidance-update source=PR-456-review-finding mode=analyze-only
/ai-guidance-update source=approved-guidance-update-2026-06-13 mode=apply-approved-update
```

If no mode is given, use `analyze-only`.

## Required behavior

- never apply governance-impacting updates without explicit approval
- never auto-promote solution notes to approved guidance
- keep changes minimal; touch one thing; do not update unrelated sections
- preserve links to approved sources
- flag conflicts instead of resolving them silently
- always produce a durable report; chat history is never the source of truth
- right-size the change: prefer the smallest edit that captures the learning, and prefer
  "no update needed" over adding guidance that won't change future AI behavior

## Phase 1 — Discover current guidance

Read: root file; context manifest; AI Architecture Context; AI Coding Guidelines;
Brownfield Rule Cards; relevant SAD sections, ADRs, formal specs; the source learning;
relevant code evidence; relevant solution notes (supporting memory only).

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
| Repeated brownfield ambiguity | Brownfield Rule Card |

## Phase 4 — Conflict check

Check the proposed update against: requirements; Story Artifact; formal specs; ADRs;
SAD; AI Architecture Context; AI Coding Guidelines; Brownfield Rule Cards; approved
reference implementation; current code; solution notes. If a conflict exists, do not
apply — produce a conflict finding.

## Phase 5 — Analyze-only output

In `analyze-only` mode, produce a Guidance Update Analysis and modify no files.

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
Choose one: no action | keep as solution note only | request human approval |
create ADR | update SAD | update formal spec | apply approved update |
create or update Brownfield Rule Card
```

## Phase 6 — Apply-approved-update behavior

In `apply-approved-update` mode:

1. Verify explicit approval exists.
2. Verify the approved target artifact.
3. Verify the approved text or intent.
4. Apply the smallest possible change.
5. Do not update unrelated sections.
6. Preserve links to approved sources.
7. Add or update the source reference.
8. Flag any conflict discovered during application.
9. Produce an Applied Guidance Update Report.

If approval is missing, stop and produce an Approval Missing Report.

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

## Stop conditions

Stop when: approval is missing; the target artifact is unclear; the update conflicts
with formal specs, SAD, or ADRs; it would require an architecture decision; it would
require security / privacy / audit / compliance approval; it would change contract
truth; or it is broader than the approved change.
