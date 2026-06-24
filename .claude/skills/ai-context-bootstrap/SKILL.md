---
name: ai-context-bootstrap
description: >-
  Create or refresh the minimum AI-facing guidance (AI Architecture Context, AI
  Coding Guidelines, Brownfield Guardrails, manifest/root-file proposals) for safe
  AI-assisted delivery in a repository. Use when starting AI delivery in a repo,
  onboarding a new service/module/bounded-context/team, creating the first Context
  or Guidelines, or checking whether existing guidance is usable. Not for
  story-specific planning (use ai-context-check) or guidance evolution (use
  ai-guidance-update).
---

# Skill: ai-context-bootstrap

## Invocation

```
/ai-context-bootstrap [scope=<area>] mode=<interactive|headless>
```

Examples:

```
/ai-context-bootstrap mode=interactive                              # whole repo
/ai-context-bootstrap scope=services/order-service mode=interactive # focus one service
/ai-context-bootstrap scope=libs/payments mode=headless            # focus an area
```

Optional: `produce=<context|guidelines|both>` (default `both`),
`source_override=<path-or-reference>`, `representative_code_override=<path>`,
`target_output_dir=<path>`. If no mode is given, use `interactive`.

`produce` selects which artifact(s) to draft: `context` runs Phase 4, `guidelines` runs
Phase 5, `both` runs both. Discovery and assessment (Phases 1–2) always run, because the
Coding Guidelines apply the Architecture Context and must read it either way.

## Scope

`scope` is which part of the repo to bootstrap: a path (`services/orders`), a glob (`apps/*`),
several paths, or a name from the manifest's `areas:`. Omit it for the whole repo. A path or a
defined `area` is the real selector; a bare free-form phrase is only a weak hint.

Scope bounds what the run examines and drafts, not the output path. Output always goes to the
single repo-level pair (`docs/architecture/ai-context.md`,
`docs/engineering/ai-coding-guidelines.md`); the Context's *Purpose & scope* section records the
covered areas. Re-running over an already-covered area reconciles, never overwrites — Phase 1
detects an existing baseline and runs a refresh.

For multiple repos, run the toolkit in each. For cross-repo architecture, keep a shared
system-level Context and have each repo link up to it rather than duplicating.

## Constraints

1. **Discover first** — never ask the user to paste anything discoverable from the repo.
2. **One blocking question at a time** — classify gaps as blocking / non-blocking /
   clarify-later; only blocking gaps may interrupt; prefer multiple-choice; put
   non-blocking questions in the report.
3. **Safe defaults** — if a missing decision touches architecture, ownership, data,
   contracts, or security/privacy/audit/compliance, never invent the answer: ask one
   blocking question, mark `TBD` or `Ask first`, recommend a decision, stop with a
   blocking finding, or produce an analyze-only report.
4. **Classify evidence** — current code is never "approved architecture" unless an
   approved source confirms it.
5. **No silent governance** — propose, never silently approve, governance-significant
   changes.
6. **Durable output** — always emit a file or report; chat history is never the source
   of truth.
7. **Right-size the work** — match ceremony to the size and clarity of the repo. A small
   or already-aligned codebase gets a compact pass: a short Context, a short Guidelines,
   and few or no Guardrails. Reserve the full multi-phase treatment for large, ambiguous,
   or high-risk repos. Don't manufacture Guardrails, sections, or questions the situation
   doesn't need.

Additional constraints: do not create a second SAD; do not copy long architecture
rationale; keep the AI-facing guidance thin. Use repo-relative paths everywhere; never
absolute paths. Write only the AI-facing layer (Context, Guidelines, Guardrails,
manifest/root-file, candidate solution notes); never write SAD/ADRs/specs/tracker items —
flag or draft those for a human.

## Phase 1 — Discover context

**Discovery strategy (don't blind-scan the whole repo):**

1. If a **context manifest** exists, treat it as the authoritative map of inputs — read
   what it lists; don't go hunting.
2. Else, if `source_override` / `representative_code_override` are given, use those.
3. Else **discover by convention, bounded by `scope`** — the standard locations listed
   below, plus the code under the scope path.

**Sampling representative code** (you can't read everything): prefer the manifest's
`code.representative`; otherwise sample within `scope` — entry points and public APIs, the
modules/services in scope, the largest or most recently-changed areas, and their tests.
Read excerpts, not whole trees.

**If discovery comes up thin** — few or no architecture sources, specs, or recognizable
representative code — **do not silently produce a thin draft.** State what's missing and
either ask the user to point at sources (interactive) or record an insufficiency note and
proceed in no-source mode with proposals + TBDs (headless).

Inspect the repository for, and classify each, as approved source / formal spec / AI
guidance / implementation evidence / known legacy / supporting memory / unknown
authority:

1. Root AI instruction file: `AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`
2. Context manifest: `ai-enablement/context-manifest.{yaml,yml,md}`
3. Existing AI guidance: `docs/architecture/ai-context.md`, `docs/engineering/ai-coding-guidelines.md`
4. Architecture sources: SAD, ADRs, diagrams, decision logs
5. Formal specs: OpenAPI, AsyncAPI, UI, data, security, privacy, audit, compliance
6. Code evidence: representative services, modules, tests, CI checks, reference implementations
7. Supporting memory: `docs/solutions/`, previous reports

### If guidance already exists (refresh)

If discovery finds existing `ai-context.md` / `ai-coding-guidelines.md` (or approved
Guardrails), this run is a **refresh** — a health-check and re-baseline, **never a
regeneration**:

- **Treat the existing files as the approved baseline.** Never overwrite or regenerate them
  wholesale; preserve all human edits, filled-in TBDs, and approved entries.
- **Validate them against the current repo** and report changes as approval-gated proposals:
  drift (a source moved away from a stated rule), new gaps (a now-uncovered concern or area),
  stale entries (a rule whose source changed or was removed), new sources (newly-found
  SAD/ADRs/specs/code to link).
- **Apply only minimal, approved additions** (same human gate as a first run). Never delete or
  rewrite an existing rule without explicit approval; if a rule looks wrong, **flag it, don't
  silently change it.**
- **Boundary:** refresh is for re-baselining, large drift, or onboarding a new area;
  incremental, per-learning evolution belongs to `ai-guidance-update`.

## Phase 2 — Assess sufficiency (detect → clarify → gate)

Run the **full concern checklist** (Phase 4) — checking what's **missing** as much as what's
**present** — and surface four things:

- an **architecturally significant concern no source covers** (don't fill it with an assumption)
- a statement that's **ambiguous or open to more than one reading** (underspecification)
- a source that **contradicts itself** (intra-source conflict)
- **sources that disagree** (cross-source conflict)

Classify each as **blocking** (the AI would otherwise assume something architecturally significant) or
**non-blocking** (a genuinely minor deferral). **Resolve every blocking item before generating
anything:**
- `interactive` — run a **human-in-the-loop clarification loop**: ask the **single most critical**
  open question (use the IDE/agent's native prompt if it has one), wait for the answer, then the
next, **in order of criticality, until none remain.**
  Ask only what genuinely needs a human — not what the sources or sensible convention already
  settle (no obvious or busywork questions). Don't batch; you may draft on the fly, but the
  **final files must fold in every clarification and stand complete** — never partial.
- `headless`, or if the human can't answer now — write no docs and produce a **Blocking Context
  Report**: an **ordered clarification agenda** (most critical first) you resume by answering and
  re-running.

Never substitute an AI assumption for a missing or unclear important concern. A completion report
is **not** a place to dump unresolved important decisions — if those remain, the run is **Blocked,
not Completed**; only genuinely non-blocking items become deferred TBDs.

**Treat as architecturally significant (missing *or* unclear ⇒ blocking):** service / bounded-context / data
ownership · cross-service communication · API / event authority · security · privacy · audit ·
compliance · technology / platform constraints · current-vs-target for a visible brownfield gap · a needed architecture decision.

### When approved sources are absent (no SAD / ADRs / specs)

With no authoritative source to point to, don't block outright. **Infer candidate rules
from representative code and conventions for lower-risk concerns, and mark every one as
*proposed / unapproved*** — existing code is evidence, not authority. For **architecturally significant**
concerns (the list above), flag or ask rather than infer — don't let a missing decision
become a silent assumption. Lean on `TBD`, ask-first, and the *Contributor
decisions needed* list; ask only the few genuinely blocking questions. Expect a Decision
of **Completed with TBDs**, with each inferred entry tagged *(proposed — needs approval)*
until a human confirms it or an ADR/SAD is created. Never present an inferred rule as
approved architecture.

## Phase 3 — Propose or update the context manifest

If none exists, propose `ai-enablement/context-manifest.yaml` in this shape — a **recommended
thin map, not an enforced schema.** Use discovered paths; mark unknowns `TBD`; omit what
doesn't apply; don't invent paths.

```yaml
guidance:        { context: <path>, guidelines: <path> }                 # OUTPUTS the toolkit maintains
sources:         { sad: [..], adrs: [..], specs: [..], diagrams: [..] }   # INPUTS (read-only)
code:            { representative: [..], known_legacy: [..], known_target: [..] }
solution_notes:  [..]
areas:           { <name>: [paths] }                                      # optional named scopes for scope=<area>
```

(The fully annotated version is in the playbook. When **reading** an existing manifest, be
tolerant — use whatever keys are present and fall back to conventional locations for the rest.)

## Phase 4 — Draft the AI Architecture Context

Create or update `docs/architecture/ai-context.md`.

**Run a coverage sweep.** Run over the **full checklist** (the concerns below) — checking what's **missing** as much as
what the sources mention. For each relevant content concern that could be misinterpreted by an
AI, decide how to cover it — sized against the existing artifacts (SAD, ADRs, LLD, security/privacy
requirements, specs):

- **Point** — an artifact covers it at an actionable level → reference it (must-read +
  a one-line operational pointer); do not restate.
- **Restate actionably** — covered but too abstract or buried to act on → add a thin
  operational rule and link back.
- **Flag for clarification** — covered, but the source is ambiguous or admits more than one
  valid reading on something that matters → don't pick a reading; flag it for clarification so
  a human states it explicitly.
- **Flag or fill** — nothing covers it. If it's **architecturally significant**, flag it for clarification —
  don't invent a rule. For lower-risk concerns you may add a *proposed* rule (marked proposed/TBD).
  Either way record the gap (the SAD/ADR/requirement may need creating; never decide it silently).

Write the file for AI consumption and easy review: conventional, stable headings (don't reinvent
the structure); short declarative bullets, not prose; links to sources instead of copies.

- **Rule shape** — one line per rule: the imperative rule, a link to the source that owns the full detail, and an inline *ask-first if …* where relevant. Prefer pointing to a canonical in-repo example ("mirror this") when one exists. State each rule once and cross-link — don't restate.
- **Weight** — make it visible: non-negotiables as **Never/Always**, preferences as **Prefer**.
- **Provenance** — open the file with one line: *generated & maintained by the toolkit; mirrors (never overrides) the SAD/ADRs/specs; drafts pending approval; evolve via `ai-guidance-update`.*
- **Sections** — use the order below as a sensible default, not a rigid template: write only sections with real content, omit what doesn't apply, never pad.

- Purpose & scope — covered areas vs `TBD`
- Read order & authority order
- Must-read sources — links to SAD / ADRs / specs / diagrams
- System overview (minimal)
- Technology & platform — languages, frameworks, runtimes, datastores; allowed/forbidden tech
- Architecture style & modularity
- Boundaries & ownership
- Data ownership & access
- Integration & communication — sync/async; allowed/forbidden; API & event ownership
- Security, privacy, audit & compliance
- Resilience & error handling
- Logging & observability
- Current-vs-target & Brownfield Guardrails
- Prohibited shortcuts & ask-first triggers
- Open gaps / TBDs

**Thin ≠ narrow:** cover every relevant concern, but where an artifact already covers
one well, shrink to a pointer. **Exclude:** full SAD content, long ADR rationale, large
copied diagrams, detailed coding conventions, implementation plans, story-specific
details, unapproved decisions, generic software-engineering advice.

## Phase 5 — Draft the AI Coding Guidelines

Create or update `docs/engineering/ai-coding-guidelines.md`. Open with a one-line provenance
header — *generated & maintained by the toolkit; applies the Architecture Context in code
(doesn't redefine it); drafts pending approval; evolve via `ai-guidance-update`.*
**Cover the concerns below that apply** (guidance for a consistent order, **not a rigid
template** — write only sections with real content; omit the rest; never pad):

- Scope control — minimal change; reuse before adding
- Technology & libraries — approved languages/frameworks/libraries; how to add a dependency
- Repository structure & placement
- Layering & module conventions
- Naming conventions
- DTOs, mapping & validation
- Error handling
- Contract-change workflow — API / event / data / UI
- Testing expectations
- Logging & observability
- Security & privacy coding rules
- Brownfield implementation rules
- Prohibited behaviors & ask-first triggers
- Reference implementations & links
- Open gaps / TBDs

Do not redefine architecture — reference the Architecture Context instead.

## Phase 6 — Validate against representative code

Inspect representative code and tests. Classify each relevant pattern as: aligned /
current-approved practice / target-ready / target-not-ready / brownfield exception /
known legacy / suspected drift / ask-first. Create Brownfield Guardrails **only** when
current implementation and target direction differ in a way that could mislead AI.

**Doc-only (no source access):** skip this phase and current-vs-target Guardrails — both need
code; rely on the docs (higher authority than code anyway) and flag ambiguity rather than infer.

## Phase 7 — Produce output

End with the **Blocked** or **Completed** output (see *Output format*). A Completed run writes
`docs/architecture/ai-context.md`, `docs/engineering/ai-coding-guidelines.md`, the manifest and
root-instruction proposals (if missing), Guardrails (only where needed), and the report below.

## Output format

Per **Phase 2**, the run ends one of two ways.

**Blocked (nothing written)** — a blocking item is unresolved; write no files and emit a
resumable agenda:

```markdown
# ai-context-bootstrap — Blocked
## Clarification agenda (most critical first)
1. <question> — why it's blocking · who decides
## Discovered so far
| Source | Path | Evidence type | Authority |
|---|---|---|---|
```
*Resume by answering the agenda and re-running.*

**Completed** — the deliverable is the **drafted files** (Phase 7, drafts pending approval) plus
this report; no blocking items remain, only non-blocking deferrals:

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
