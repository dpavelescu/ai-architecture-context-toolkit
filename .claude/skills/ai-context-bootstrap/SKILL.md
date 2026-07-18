---
name: ai-context-bootstrap
description: >-
  Generates a repository's AI Architecture Rules and AI Coding Guidelines from its approved
  sources and representative code, and opens a clarifications ledger holding the decisions still
  unsettled. Covers the whole repo or one service/area, first time or as a re-baseline of existing
  guidance. The siblings own the rest of the loop: ai-context-check reviews work against the
  guidance, ai-guidance-update changes it.
---

# Skill: ai-context-bootstrap

Runs in the main conversation so it can ask blocking questions interactively. It calls the shared
skills **read-source-map**, **assess-coverage**, **write-guidance-file**,
**write-brownfield-guardrail**, and **update-clarifications-ledger** for the reusable capabilities —
invoke each where the Process names it rather than re-deriving its logic here.

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

`produce` (default `both`) selects which file(s) get written — `context` the Architecture Rules, `guidelines`
the Guidelines, `both` both; discovery, assessment, the critical questions, and the ledger run
regardless, because the Coding Guidelines apply the Architecture Rules and must read it either way.
The skill asks only **critical** items live, one at a time (offering *decide now or
defer to the ledger*); everything else becomes a ledger candidate. It always writes the clean file(s)
plus the clarifications ledger — it never blocks on open decisions, and with no one there to answer,
the critical items land in the ledger too.

## Scope

`scope` is which part of the repo to bootstrap: a path (`services/orders`), a glob (`apps/*`),
several paths, or a name from the source map's `areas:`. Omit it for the whole repo. A path or a
defined `area` is the real selector; a bare free-form phrase is only a weak hint.

Scope bounds what the run examines and writes, not the output path. Output always goes to the
single repo-level Architecture Rules/Guidelines pair, not a per-area copy; the Architecture Rules' *Purpose & scope*
section records the covered areas. Re-running over an already-covered area reconciles, never regenerates — the
**Discover** step detects an existing baseline and runs a refresh.

For multiple repos, run the toolkit in each. For cross-repo architecture, keep a shared
system-level Architecture Rules and have each repo link up to it rather than duplicating.

## Constraints

1. **Discover first** — never ask the user to paste anything discoverable from the repo.
2. **Never invent an answer** — existing implementation is evidence, not authority: code only
   *proposes*, never self-ratifies, and a document carries authority only where the authority order
   ranks it an approved source.
3. **No silent governance** — propose; a human approves. Nothing this run works out for itself
   becomes a rule.
4. **Durable output** — always emit a file or report; chat history is never the source
   of truth.
5. **Right-size the work** — match ceremony to the size and clarity of the repo. A small
   or already-aligned codebase gets a compact pass: a short Architecture Rules, a short Guidelines,
   and few or no Guardrails. Reserve the full treatment for large, ambiguous, or
   high-risk repos. Don't manufacture Guardrails, sections, or questions the situation
   doesn't need.
6. **Write only the AI-facing layer** — the Architecture Rules, Guidelines, Guardrails, the clarifications
   ledger, the root-file proposal, and candidate solution notes. Never author the source map; it is
   optional and human-owned. Never write into a SAD, ADR, spec, or tracker item: flag that one needs
   changing, or draft proposed text for a human to own. Don't create a second SAD or copy long
   architecture rationale — keep the AI-facing guidance thin, and use repo-relative paths everywhere.

## Process

1. **Discover** — load the repo's AI-context inputs with the **read-source-map** skill, bounded by
   `scope`: the approved architecture sources, formal specs, and representative code, plus any existing
   `guidance` and the `ledger`. These are what the later steps draw on; **Assess** decides which of them
   bear on each concern. If it resolved no `sources` — neither approved sources nor code — write
   nothing and stop; report what's missing.
   - **If guidance already exists (refresh).** When it resolves existing `guidance` (Architecture Rules,
     Guidelines, or approved Guardrails) or a `ledger`, this run is a refresh: that guidance is the
     approved `baseline`. Per-learning evolution stays with `ai-guidance-update`.
2. **Assess** — split every concern that matters into what the approved sources already settle and what
   still needs a human; the split decides what the clean files may say and what they must stay silent
   on. Apply the **assess-coverage** skill; on a refresh it also assesses the existing `baseline`. It
   returns **final rules** and **ledger candidates**, each candidate marked `raise: live` or
   `raise: ledger`.
3. **Ask the critical few** — **you** ask, in this conversation, and **before any file is written**. Put
   each `raise: live` candidate one at a time, most critical first, preferring multiple choice, each
   offering *decide now or defer to the ledger*:
   - **Answered** → that answer is the decision: it becomes a final rule.
   - **Deferred** → it stays a candidate, and you don't raise it again this run.
   - **Nobody is there to answer** (a non-interactive run) → don't block and don't guess: every
     `raise: live` candidate stays a candidate, and the run continues.

   Never guess a critical item, never silently defer one, and never start writing while a live question
   is outstanding.
4. **Record the open decisions** — record the still-undecided candidates in the ledger with the
   **update-clarifications-ledger** skill; take its count line for the Result.
5. **Write the selected guidance** — validate the final rules against the sampled code, and for each
   current-vs-target divergence that would mislead the AI, produce a Brownfield Guardrail with the
   **write-brownfield-guardrail** skill. Then, for each file `produce` selects — the Architecture Rules, the
   Guidelines, or both — write it from the **final rules** with the **write-guidance-file** skill,
   merging into the existing file on a refresh.
6. **Report** — if the repo's agent instruction file (conventionally `CLAUDE.md` or `AGENTS.md`) is
   missing, or exists but doesn't already send the agent to the source map, Architecture Rules, Coding
   Guidelines, and clarifications ledger — in that order — before it analyses, plans, codes, or reviews,
   draft that read order and carry it into the Result's Proposals section as a proposal for a human.
   Emit the Result in the **Output format**.

## Output format

One Result — the file(s) `produce` selected and the ledger are always written together. (If discovery
found neither sources nor code, nothing is written: report where you looked, what's missing, and the
Recommended next step instead of a Result.)

```markdown
# ai-context-bootstrap Result

## Files written
(only the files this run actually wrote)
- docs/architecture/ai-architecture-rules.md (clean — decided rules only)
- docs/engineering/ai-coding-guidelines.md   (clean — decided rules only)
- docs/architecture/ai-clarifications.md     (<N> open)

## Sources used
| Source | Type | Path | Authority |
|---|---|---|---|

## Open clarifications
- <N> in the ledger, most important first — ratify or reject via `ai-guidance-update`

## Brownfield Guardrails created
| Topic | Status | Reason |
|---|---|---|

## Proposals — pending approval, not written
- <the drafted root instruction file, or omit the section>

## Refresh summary (refresh runs only)
- Kept / Added / Drift→ledger / Stale / Settled preserved

## Recommended next step
- <next step>
```
