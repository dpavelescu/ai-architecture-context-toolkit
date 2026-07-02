---
name: ai-context-bootstrap
description: >-
  Create or refresh the minimum AI-facing guidance (AI Architecture Context, AI
  Coding Guidelines, Brownfield Guardrails, source-map/root-file proposals) for safe
  AI-assisted delivery in a repository. Use when starting AI delivery in a repo,
  onboarding a new service/module/bounded-context/team, creating the first Context
  or Guidelines, or checking whether existing guidance is usable. Not for
  story-specific planning (use ai-context-check) or guidance evolution (use
  ai-guidance-update).
---

# Skill: ai-context-bootstrap

Runs in the main conversation so it can ask blocking questions interactively. It calls the
shared skills **read-context-manifest**, **assess-coverage**, **write-guidance-file**, and
**write-brownfield-guardrail** for the reusable capabilities — invoke each where the Process
names it rather than re-deriving its logic here.

## Invocation

```
/ai-context-bootstrap [scope=<area>] [produce=<context|guidelines|both>]
```

Examples:

```
/ai-context-bootstrap                               # whole repo
/ai-context-bootstrap scope=services/order-service  # focus one service
/ai-context-bootstrap scope=libs/payments           # focus an area
```

`produce` (default `both`) selects which file(s) to draft — `context` runs step 3, `guidelines`
runs step 4, `both` runs both; discovery and assessment (steps 1–2) always run, because the
Coding Guidelines apply the Architecture Context and must read it either way. The skill asks only
**critical** items live, one at a time (offering *decide now or defer to the ledger*); everything
else becomes a ledger candidate. It always writes the clean files plus the clarifications ledger —
it never blocks on open decisions.

## Scope

`scope` is which part of the repo to bootstrap: a path (`services/orders`), a glob (`apps/*`),
several paths, or a name from the source map's `areas:`. Omit it for the whole repo. A path or a
defined `area` is the real selector; a bare free-form phrase is only a weak hint.

Scope bounds what the run examines and drafts, not the output path. Output always goes to the
single repo-level pair (`docs/architecture/ai-context.md`,
`docs/engineering/ai-coding-guidelines.md`); the Context's *Purpose & scope* section records the
covered areas. Re-running over an already-covered area reconciles, never overwrites — the
**Discover** step detects an existing baseline and runs a refresh.

For multiple repos, run the toolkit in each. For cross-repo architecture, keep a shared
system-level Context and have each repo link up to it rather than duplicating.

## Constraints

1. **Discover first** — never ask the user to paste anything discoverable from the repo.
2. **Ask only critical items live** — one at a time, prefer multiple-choice, offering
   *decide now or defer to the ledger*. Everything else becomes a ledger candidate, not a
   live question.
3. **Never invent an answer** — code is a source for *proposals* (lowest authority), never
   self-ratifying. Critical concerns (security, privacy, compliance, data ownership, a needed
   architecture decision) are asked live; everything else is proposed in the ledger for a human
   decision. Nothing undecided enters the clean files.
4. **Classify evidence** — current code is never "approved architecture" unless an
   approved source confirms it.
5. **No silent governance** — propose, never silently approve, governance-significant
   changes.
6. **Durable output** — always emit a file or report; chat history is never the source
   of truth.
7. **Right-size the work** — match ceremony to the size and clarity of the repo. A small
   or already-aligned codebase gets a compact pass: a short Context, a short Guidelines,
   and few or no Guardrails. Reserve the full treatment for large, ambiguous, or
   high-risk repos. Don't manufacture Guardrails, sections, or questions the situation
   doesn't need.

Additional constraints: the Context and Guidelines are **clean and final** — only decided rules;
every open decision lives in the clarifications ledger, never in those files. Do not create a
second SAD; do not copy long architecture rationale; keep the AI-facing guidance thin. Use
repo-relative paths everywhere; never absolute paths. Write only the AI-facing layer (Context,
Guidelines, Guardrails, clarifications ledger, source-map/root-file, candidate solution notes);
never write SAD/ADRs/specs/tracker items — flag or draft those for a human.

## Process

1. **Discover** — resolve inputs with the **read-context-manifest** skill (source map first, search
   fallback, bounded by `scope`); take the structured source list in authority order. Sample
   representative code — entry points and public APIs, the in-scope modules/services, the largest or
   most-recently-changed areas, and their tests; read excerpts, not whole trees. If discovery finds
   neither sources nor code, write nothing and report what's missing instead.
   - **If guidance already exists (refresh).** When discovery finds an existing `ai-context.md` /
     `ai-coding-guidelines.md` / clarifications ledger (or approved Guardrails), this run is a refresh —
     a health-check and re-baseline, never a regeneration: treat the existing files as the approved
     baseline (preserve human edits, approved entries, and the ledger's `Settled` list); propose drift,
     new gaps, and stale entries as new ledger candidates; never delete or rewrite a rule without
     approval. Per-learning evolution belongs to `ai-guidance-update`.
2. **Assess** — apply the **assess-coverage** skill: the relevance gate surfaces only concerns that
   matter (variation already evidenced, high impact if they vary, or framework standardization), then
   routes each kept concern to either a **final rule** (an approved source settles it) or a **ledger
   candidate** (needs a decision — proposal + rationale; code-derived proposals are lowest authority).
   - **Critical candidates** (security, privacy, compliance, data ownership, or a needed architecture
     decision) — ask one at a time, most critical first, offering *decide now or defer to the ledger*.
     Never guess, never silently defer.
   - All other candidates go to the ledger with no live question.
3. **Draft the Context (clean)** — when `produce ∈ {context, both}`, write `docs/architecture/ai-context.md`
   per the **write-guidance-file** skill from the **final rules only** — no candidates, no statuses, no
   TBDs; if there are no final rules yet, write only the provenance header and note it under Recommended
   next step. Validate against representative code and add a Guardrail (the **write-brownfield-guardrail**
   skill) only where current implementation and target direction differ enough to mislead the AI.
4. **Draft the Guidelines (clean)** — when `produce ∈ {guidelines, both}`, write
   `docs/engineering/ai-coding-guidelines.md` per the **write-guidance-file** skill from the final rules
   (none → provenance header only). Don't redefine architecture; link to the Context.
5. **Write the clarifications ledger** — `docs/architecture/ai-clarifications.md`: every open candidate
   (proposal + rationale + empty `decision:`), most important first, under `## Open`; carry forward any
   prior `## Settled — won't re-propose` list and don't re-raise settled items. Candidates never appear
   in the Context or Guidelines.
6. **Propose** — if missing, propose the source map (**read-context-manifest** skill) and the repo's
   root instruction file.
7. **Produce output** — emit the Result (see **Output format**): the clean Context/Guidelines, the
   ledger, the source-map/root-instruction proposals (if missing), Guardrails (only where needed), and
   the report.

## Output format

One Result — the clean files plus the ledger always travel together. (If discovery found neither
sources nor code, write nothing and report what's missing under **Recommended next step** instead.)

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
