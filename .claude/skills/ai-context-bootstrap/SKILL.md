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

## Scope — what a run seeds

`scope` names the area this run seeds. It can be:
- **omitted** → the whole repo;
- **a path** → `services/orders`;
- **several paths or a glob** → `services/orders, libs/payments` · `apps/*`;
- **a name defined in the manifest's `areas:`** → `payments` (resolves to its mapped paths) — this is how a bounded context that spans directories becomes selectable.

A path (or a defined `area`) is the real selector — the agent bounds discovery to it. A bare
free-form phrase is only a weak hint; prefer a path or a defined area.

Scope bounds what the run **examines and drafts**, not the output path. Output is always the
single repo-level set (`docs/architecture/ai-context.md`, `docs/engineering/ai-coding-guidelines.md`);
the Context's *Purpose & scope* section **records each run's covered scope by its label**.

**Runs compound into the one Context** (this is what makes seeding incremental):
- a **new sub-scope** is **additive** — its coverage is appended; the rest is untouched;
- a **re-run over, or overlapping, an already-covered scope** **reconciles** — validate the
  existing entries, propose drift/additions, **never duplicate or overwrite** approved content
  (refresh mode).
Coverage only ever **grows**; *Purpose & scope* shows the union of covered areas vs what's still `TBD`.

**Multiple repos:** the toolkit is per-repo — run it in each repo. Cross-repo architecture
(the SAD/ADRs usually span repos) is governed *above* the repo: keep a shared
**system-level Context** alongside the cross-repo SAD/ADRs, and have each repo's
Context/manifest **link up** to it and add only repo-local specifics — point, don't
duplicate.

## House rules (apply throughout)

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

## Refresh mode — re-running where guidance already exists

If Phase 1 finds existing `ai-context.md` / `ai-coding-guidelines.md` (or approved
Guardrails), switch to **refresh mode** — a health-check and re-baseline, **never a
regeneration**:

- **Treat the existing files as the approved baseline.** Never overwrite or regenerate
  them wholesale; preserve all human edits, filled-in TBDs, and approved entries.
- **Validate them against the current repo** and report changes as approval-gated proposals:
  - **drift** — code or an approved source has moved away from a stated rule
  - **new gaps** — a relevant dimension or area is now uncovered
  - **stale entries** — a rule whose source changed or was removed
  - **new sources** — newly-found SAD/ADRs/specs/code to link
- **Apply only minimal, approved additions** (same human gate as a first run). Never delete
  or rewrite an existing rule without explicit approval; if a rule looks wrong, **flag it,
  don't silently change it.**
- **Boundary:** refresh is for re-baselining, large drift, or onboarding a new area.
  Incremental, per-learning evolution belongs to `ai-guidance-update`.

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

## Phase 2 — Assess sufficiency

Decide: **sufficient to draft** / **sufficient to draft with TBDs** / **insufficient
(blocking gaps)**. Blocking gaps include missing or unclear:

- service / bounded-context / data ownership
- cross-service communication rule
- API or event authority
- security, privacy, audit, or compliance constraints
- current-vs-target direction for a visible brownfield conflict
- source-of-truth conflict between SAD, ADRs, specs, AI context, and code

In `interactive` mode, ask exactly one blocking question. In `headless` mode, do not
ask — stop and produce a **Blocking Context Report**.

### When approved sources are absent (no SAD / ADRs / specs)

With no authoritative source to point to, don't block outright. **Infer candidate rules
from representative code and conventions, and mark every one as *proposed / unapproved*** —
existing code is evidence, not authority. Lean on `TBD`, ask-first, and the *Contributor
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

**Run a coverage sweep.** For each dimension below that is relevant to this repo and
could be misinterpreted by an AI, choose one of three — sized against the existing
artifacts (SAD, ADRs, LLD, security/privacy requirements, specs):

- **Point** — an artifact covers it at an actionable level → reference it (must-read +
  a one-line operational pointer); do not restate.
- **Restate actionably** — covered but too abstract or buried to act on → add a thin
  operational rule and link back.
- **Fill and flag** — not covered anywhere (or only implied in code) → capture the
  operational rule here, and record a gap in *Contributor decisions needed* (the
  SAD/ADR/requirement may need creating or updating; never decide it silently).

Dimensions to sweep (all equal — don't over-weight any one):

- ownership & boundaries; data ownership & access
- integration (sync/async; allowed/forbidden); API & event contracts
- security; data privacy / PII; audit; compliance
- architecture style & modularity (modular monolith / microservices-distributed /
  layered) — e.g. in a modular monolith, state module isolation explicitly because
  nothing physically enforces it
- error handling / resilience, logging / observability — only where architecturally
  constrained
- current-vs-target (brownfield) divergences

**The Context must include:** purpose & scope; read order; authority order; must-read
sources; minimal system overview; ownership & boundary rules; data ownership;
integration rules; API & event rules; architecture style & modularity rules; security /
privacy / audit / compliance constraints; current-vs-target guidance; Brownfield Guardrails
(only where needed); prohibited shortcuts; ask-first triggers; links to SAD /
ADRs / specs / diagrams.

**Thin ≠ narrow:** cover every relevant dimension, but where an artifact already covers
one well, shrink to a pointer. **Exclude:** full SAD content, long ADR rationale, large
copied diagrams, detailed coding conventions, implementation plans, story-specific
details, unapproved decisions, generic software-engineering advice.

## Phase 5 — Draft the AI Coding Guidelines

Create or update `docs/engineering/ai-coding-guidelines.md`. **Include:** scope
control; repository structure; layering & module conventions; how to apply the
Architecture Context in code; DTO / mapping / validation / error-handling; API & event
contract-change workflow; testing expectations; logging & observability; security /
privacy / audit / compliance coding rules; prohibited implementation behaviors;
ask-first triggers; brownfield implementation rules (only where needed); reference
implementations.

Do not redefine architecture — reference the Architecture Context instead.

## Phase 6 — Validate against representative code

Inspect representative code and tests. Classify each relevant pattern as: aligned /
current-approved practice / target-ready / target-not-ready / brownfield exception /
known legacy / suspected drift / ask-first. Create Brownfield Guardrails **only** when
current implementation and target direction differ in a way that could mislead AI.

## Phase 7 — Produce output

1. `docs/architecture/ai-context.md`
2. `docs/engineering/ai-coding-guidelines.md`
3. `ai-enablement/context-manifest.yaml`, if missing or incomplete
4. root AI instruction proposal, if missing or incomplete
5. Validation Report
6. Brownfield Guardrails, only where needed
7. Contributor Decisions Needed

## Output format

**Resolve every blocking ambiguity first** (Phase 2 / Stop conditions); write files only when
none remain. If a blocker stays unresolved, **stop with the Blocking Context Report and write
no files** — that is the *Blocked* outcome.

On success, the deliverable is the **drafted files** (Phase 7) — drafts pending approval —
plus this completion report. By definition no blocking items remain; only deliberately-deferred,
non-blocking ones appear:

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

## Stop conditions

Stop and ask one blocking question (in `headless`, produce a Blocking Context Report
instead) when:

- an authority conflict cannot be resolved
- ownership or data ownership is unclear
- security, privacy, audit, or compliance implications are unclear
- current code and target direction conflict
- the AI would need to make an architecture decision
