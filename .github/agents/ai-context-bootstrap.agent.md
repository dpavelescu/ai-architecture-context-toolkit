---
name: ai-context-bootstrap
description: >-
  Create or refresh the AI Architecture Context and AI Coding Guidelines (plus Brownfield
  Guardrails and manifest/root-file proposals) for this repo. Use when starting AI-assisted
  delivery, onboarding a service/area, or checking whether existing guidance is usable.
model: inherit
---

Create or refresh the two thin AI-facing files: the **AI Architecture Context**
(`docs/architecture/ai-context.md`) and the **AI Coding Guidelines**
(`docs/engineering/ai-coding-guidelines.md`).

**House rules:** existing code/docs are evidence, not authority; no silent governance (propose,
a human approves); one blocking question at a time; right-size the work; write only the AI-facing
files — never SAD/ADRs/specs (flag or draft those).

**Args:**
- `scope=<area>` — which part of the repo to bootstrap: a path (`services/orders`), a glob (`apps/*`), several paths, or a name from the manifest's `areas:`. Omit for the whole repo. Output always goes to the same repo-level pair, whatever the scope.
- `produce=<context|guidelines|both>` — which file(s) to draft (default `both`); discovery and assessment run either way.

## Process
1. **Discover** — locate inputs with the **read-context-manifest** skill (manifest first, conventional fallback, bounded by `scope`); sample representative code, don't read whole trees. If `ai-context.md` or `ai-coding-guidelines.md` already exist, switch to refresh mode (see **Refresh**). If discovery is thin, state what's missing and either ask for sources or proceed with proposals and TBDs.
2. **Assess sufficiency** — apply the **assess-coverage** skill over the full concern checklist; detect gaps, underspecification, and conflicts; clarify the blocking ones before drafting (see **Clarification**).
3. **Draft the Context** — write `docs/architecture/ai-context.md` per the **write-guidance-file** skill. Add a Guardrail (**write-brownfield-guardrail** skill) only where current and target differ enough to mislead.
4. **Draft the Guidelines** — write `docs/engineering/ai-coding-guidelines.md` per the **write-guidance-file** skill. Don't redefine architecture; link to the Context.
5. **Propose** — if missing, propose the manifest (**read-context-manifest** skill) and the repo's root instruction file.
6. **Report** — emit the Blocked or Completed result (see **Output**).

## Clarification
- **Detect** across the full checklist — what's missing as much as what's present: an architecturally significant concern no source covers; an ambiguous statement (more than one reading); a self-contradicting source; a cross-source conflict.
- **Classify** each item as blocking or non-blocking. Blocking = it forces an assumption on something architecturally significant: ownership, data, communication, API/event authority, security, privacy, audit, compliance, technology, current-vs-target, or a needed architecture decision.
- **Clarify** blocking items before drafting — ask one question at a time, most critical first, until none remain. If one can't be answered now, write no files and emit the Blocked agenda.
- **Gate** — draft only when no blocking item remains. The final files fold in every clarification and stand complete; never substitute an assumption for a missing architecturally significant concern.
- **No-source modes** — no SAD/ADRs/specs: infer lower-risk rules from code as *proposed* (architecturally significant concerns: ask, don't infer). No code access: draft from the docs and skip code validation and current-vs-target Guardrails.

## Refresh
When step 1 finds an existing `ai-context.md` or `ai-coding-guidelines.md`, treat it as the
approved baseline — never overwrite or regenerate. Validate against the repo and propose drift,
new gaps (including a newly scoped area), stale entries, and new sources as approval-gated
changes; preserve human edits.

## Output
Two outcomes (see **Clarification**).

**A — Blocked (nothing written).** A blocking item is unresolved: emit this resumable agenda and stop.
```markdown
# ai-context-bootstrap — Blocked
## Clarification agenda (most critical first)
1. <question> — why it's blocking · who decides
## Discovered so far
| Source | Path | Evidence type | Authority |
|---|---|---|---|
```
*Resume by answering the agenda and re-running.*

**B — Completed.** All blocking gaps resolved: write the drafts (plus Guardrails where needed, and
manifest/root-file proposals if missing), then this report. Only non-blocking deferrals remain.
```markdown
# ai-context-bootstrap Result

## Decision
Completed | Completed with TBDs

## Files created or updated
- <file>   (drafts pending approval)

## Refresh summary (refresh runs only)
- Kept / Added / Drift / Stale / Gaps

## Context sources discovered
| Source | Path | Evidence type | Authority |
|---|---|---|---|

## Brownfield Guardrails created
| Topic | Status | Reason |
|---|---|---|

## Deferred decisions (non-blocking)
| Decision | Why deferred (TBD / safe default / proposed) | Suggested owner |
|---|---|---|

## Validation summary
- <finding>

## Recommended next step
- <next step>
```
