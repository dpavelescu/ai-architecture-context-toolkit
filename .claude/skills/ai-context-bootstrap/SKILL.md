---
name: ai-context-bootstrap
description: >-
  Generates a repository's AI Architecture Rules and AI Coding Guidelines from its approved
  sources and representative code, and opens a clarifications ledger holding the decisions still
  unsettled. Covers the whole repo or one service/area, first time or as a re-baseline. Creates the
  guidance; doesn't review or change it.
---

# Skill: ai-context-bootstrap

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
regardless.

## Scope

`scope` is which part of the repo to bootstrap: a path (`services/orders`), a glob (`apps/*`),
several paths, or a name from the source map's `areas:`. Omit it for the whole repo. A path or a
defined `area` is the real selector; a bare free-form phrase is only a weak hint.

Scope bounds what the run examines and writes, not the output path. Output always goes to the
single repo-level Architecture Rules/Guidelines pair, not a per-area copy. Re-running over an
already-covered area reconciles, never regenerates.

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
   changing, or draft proposed text for a human to own. Don't create a second SAD — keep the
   AI-facing guidance thin, and use repo-relative paths everywhere.

## Process

Each phase runs until its **Complete when** holds; don't enter the next phase before it does.

### Phase 1 — Discover

**Goal** — The approved sources, representative code, existing guidance, and ledger relevant to
`scope` are resolved.

**Procedure** — With the **read-source-map** skill, bounded by `scope`, load the sources relevant to
the guidance's concerns (architecture sources, specs, code), plus any existing `guidance` and the
`ledger`. Search and read as often as it takes. If it resolved no `sources` — neither approved
sources nor code — write nothing and stop; report what's missing.

- **If guidance already exists (refresh).** When it resolves existing `guidance` (Architecture Rules,
  Guidelines, or approved Guardrails) or a `ledger`, this run is a refresh: that guidance is the
  approved `baseline`. Per-learning evolution stays with `ai-guidance-update`.

**Complete when** each category — approved sources, representative code, existing guidance, ledger —
is resolved or confirmed absent.

### Phase 2 — Assess

**Goal** — This run knows which concerns it may state as rules and which it must leave to a human.

**Procedure** — Split every concern that matters into what the approved sources already settle and
what still needs a human; the split decides what the clean files may say and what they must stay
silent on. Apply the **assess-coverage** skill; on a refresh it also assesses the existing
`baseline`.

**Complete when** every concern that cleared the relevance gate is classified — a **final rule**, or
a **ledger candidate** marked `raise: live` or `raise: ledger`.

### Phase 3 — Ask the critical few

**Goal** — The decisions too important to defer have been put to a human before anything is written.

**Procedure** — **You** ask, in this conversation, and **before any file is written**. Put each
`raise: live` candidate one at a time, most critical first, preferring multiple choice, each
offering *decide now or defer to the ledger*:

- **Answered** → that answer is the decision: it becomes a final rule.
- **Deferred** → it stays a candidate, and you don't raise it again this run.
- **Nobody is there to answer** (a non-interactive run) → don't block and don't guess: every
  `raise: live` candidate stays a candidate, and the run continues.

Never guess a critical item, never silently defer one, and never start writing while a live question
is outstanding.

**Complete when** every `raise: live` candidate has been answered, deferred, or carried for want of
anyone to answer — and no question is outstanding.

### Phase 4 — Record the open decisions

**Goal** — The ledger carries every open decision this run surfaced.

**Procedure** — Record the still-undecided candidates in the ledger with the
**update-clarifications-ledger** skill; take its count line for the Result.

**Complete when** the ledger holds every still-undecided candidate, with the prior `Settled` list
preserved.

### Phase 5 — Write the selected guidance

**Goal** — The repo has AI-facing guidance that states only decided rules, and a Guardrail wherever
current code would mislead.

**Procedure** — Validate the final rules against the sampled code, and for each current-vs-target
divergence produce a Brownfield Guardrail with the **write-brownfield-guardrail** skill. Then, for
each file `produce` selects — the Architecture Rules, the Guidelines, or both — write it from the
**final rules** with the **write-guidance-file** skill, merging into the existing file on a refresh.

**Complete when** every file `produce` selects is written, carrying only final rules and merged into
its baseline on a refresh.

### Phase 6 — Report

**Goal** — A human is left knowing what this run wrote, what stays open, and in what order an agent
should read it.

**Procedure** — Propose the read order for the repo's agent instruction file (conventionally
`CLAUDE.md` or `AGENTS.md`): read the source map, Architecture Rules, Coding Guidelines, and
clarifications ledger — in that order — before analysing, planning, coding, or reviewing. Carry it
into the Result's Proposals section as a recommendation for a human.

**Complete when** the Result is emitted in the **Output format**, accounting for every file written
and every open clarification.

## Output format

One Result. (If discovery found neither sources nor code, nothing is written: report where you
looked, what's missing, and the Recommended next step instead of a Result.)

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
- <N> in the ledger, most important first — ratify or reject via `ai-guidance-update source=clarification-decision`

## Brownfield Guardrails created
| Topic | Status | Reason |
|---|---|---|

## Proposals — recommendations for a human, nothing written
- <the recommended agent-instruction read order (source map → Architecture Rules → Coding Guidelines → ledger)>

## Refresh summary (refresh runs only)
- Kept / Added / Drift→ledger / Stale / Settled preserved

## Recommended next step
- <next step>
```
