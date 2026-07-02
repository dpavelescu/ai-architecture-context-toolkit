---
name: ai-context-bootstrap
description: >-
  Create or refresh the AI Architecture Context and AI Coding Guidelines (plus Brownfield
  Guardrails and source-map/root-file proposals) for this repo. Use when starting AI-assisted
  delivery, onboarding a service/area, or checking whether existing guidance is usable. Not for
  story/PR alignment checks (use ai-context-check) or evolving approved guidance (use ai-guidance-update).
model: inherit
tools: ["read", "search", "edit"]
---

## Constraints
- Existing code/docs are evidence, not authority — code is a source for *proposals* (lowest authority), never self-ratifying.
- No silent governance: propose, a human approves.
- The Context and Guidelines are clean and final — only decided rules; every open decision lives in the clarifications ledger, never in those files.
- Ask only critical items live, one at a time; everything else goes to the ledger.
- Right-size the work — sample representative code, don't audit whole trees.
- Write only the AI-facing files — never SAD/ADRs/specs (flag or draft those).

## Inputs
- `scope=<area>` — which part of the repo to bootstrap: a path (`services/orders`), a glob (`apps/*`), several paths, or a name from the manifest's `areas:`. Omit for the whole repo. Output always goes to the same repo-level pair, whatever the scope.
- `produce=<context|guidelines|both>` — which file(s) to draft (default `both`); discovery and assessment run either way.

## Process
1. **Discover** — resolve inputs with the **read-context-manifest** skill (source map first, search fallback, bounded by `scope`); take the structured source list in authority order; sample representative code, don't read whole trees.
   - If `ai-context.md`, `ai-coding-guidelines.md`, or the clarifications ledger already exist, this run is a **refresh**: treat them as the approved baseline — never overwrite or regenerate; preserve human edits and the ledger's `Settled` list; propose drift, new gaps (including a newly scoped area), and stale entries as new ledger candidates.
   - If discovery finds neither sources nor code, write nothing and report what's missing instead.
2. **Assess** — apply the **assess-coverage** skill: the relevance gate surfaces only concerns that matter (variation already evidenced, high impact if they vary, or framework standardization), then routes each kept concern to either a **final rule** (an approved source settles it) or a **ledger candidate** (needs a decision — proposal + rationale; code-derived proposals are lowest authority).
   - **Critical candidates** (security, privacy, compliance, data ownership, or a needed architecture decision) — ask one at a time, most critical first, offering *decide now or defer to the ledger*. Never guess, never silently defer.
   - All other candidates go to the ledger with no live question.
3. **Draft the Context (clean)** — when `produce ∈ {context, both}`, write `docs/architecture/ai-context.md` per the **write-guidance-file** skill from the **final rules only** — no candidates, no statuses, no TBDs; if there are no final rules yet, write only the provenance header and note it under Recommended next step. Add a Guardrail (**write-brownfield-guardrail** skill) only where current and target differ enough to mislead.
4. **Draft the Guidelines (clean)** — when `produce ∈ {guidelines, both}`, write `docs/engineering/ai-coding-guidelines.md` per the **write-guidance-file** skill from the final rules (none → provenance header only). Don't redefine architecture; link to the Context.
5. **Write the clarifications ledger** — `docs/architecture/ai-clarifications.md`: every open candidate (proposal + rationale + empty `decision:`), most important first, under `## Open`; carry forward any prior `## Settled — won't re-propose` list and don't re-raise settled items. Candidates never appear in the Context or Guidelines.
6. **Propose** — if missing, propose the source map (**read-context-manifest** skill) and the repo's root instruction file.
7. **Report** — emit the Result (see **Output format**).

## Output format
One Result — clean files plus the ledger always travel together. (If discovery found neither sources nor code, write nothing and report what's missing under **Recommended next step** instead.)
```markdown
# ai-context-bootstrap Result

## Files written
- docs/architecture/ai-context.md          (clean — decided rules only)
- docs/engineering/ai-coding-guidelines.md (clean — decided rules only)
- docs/architecture/ai-clarifications.md   (<N> open)

## Sources used
| Source | Type | Path | Authority |
|---|---|---|---|

## Open clarifications
- <N> in the ledger, most important first — ratify or reject via `ai-guidance-update`

## Brownfield Guardrails created
| Topic | Status | Reason |
|---|---|---|

## Refresh summary (refresh runs only)
- Kept / Added / Drift→ledger / Stale / Settled preserved

## Recommended next step
- <next step>
```
