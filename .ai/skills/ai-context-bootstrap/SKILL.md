---
name: ai-context-bootstrap
description: >-
  Create or refresh the minimum AI-facing guidance (AI Architecture Context, AI
  Coding Guidelines, Brownfield Rule Cards, manifest/root-file proposals) for safe
  AI-assisted delivery in a repository. Use when starting AI delivery in a repo,
  onboarding a new service/module/bounded-context/team, creating the first Context
  or Guidelines, or checking whether existing guidance is usable. Not for
  story-specific planning (use ai-context-check) or guidance evolution (use
  ai-guidance-update).
---

# Skill: ai-context-bootstrap

## Purpose

Create or refresh the minimum AI-facing guidance required for safe AI-assisted
delivery. This skill produces or updates:

- AI Architecture Context (`docs/architecture/ai-context.md`)
- AI Coding Guidelines (`docs/engineering/ai-coding-guidelines.md`)
- Brownfield Rule Cards — only where needed
- a context manifest proposal, if missing
- a root AI instruction file proposal, if missing
- a validation report
- a list of contributor decisions needed

Intended for brownfield systems where existing code may not represent approved
architecture intent.

## When to use

- Starting AI-assisted delivery in a repository
- Onboarding a new service, module, bounded context, or team
- Creating the first AI Architecture Context or AI Coding Guidelines
- Validating whether existing AI guidance is usable
- Identifying brownfield differences that may mislead AI

Do **not** use for story-specific implementation planning (use `ai-context-check`)
or for approved guidance evolution (use `ai-guidance-update`).

## Invocation

```
/ai-context-bootstrap scope=<repository|service|module|bounded-context> mode=<interactive|headless>
```

Examples:

```
/ai-context-bootstrap scope=repository mode=interactive
/ai-context-bootstrap scope=services/order-service mode=interactive
/ai-context-bootstrap scope=bounded-context:payments mode=headless
```

Optional: `source_override=<path-or-reference>`, `representative_code_override=<path>`,
`target_output_dir=<path>`. If no mode is given, use `interactive`.

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
   and few or no Rule Cards. Reserve the full multi-phase treatment for large, ambiguous,
   or high-risk repos. Don't manufacture Rule Cards, sections, or questions the situation
   doesn't need.

Additional constraints: do not create a second SAD; do not copy long architecture
rationale; keep the AI-facing guidance thin. Use repo-relative paths everywhere; never
absolute paths.

## Phase 1 — Discover context

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

## Phase 3 — Propose or update the context manifest

If none exists, propose `ai-enablement/context-manifest.yaml` mapping: AI Context,
Guidelines, SAD, ADRs, formal specs, diagrams, representative code, known legacy
areas, known target examples, supporting memory. Use discovered paths; mark unknowns
`TBD`. Do not invent paths.

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

Dimensions to sweep:

- **architecture style & modularity** (modular monolith / microservices-distributed /
  layered) and the boundary rules each implies — e.g. in a modular monolith, state
  module isolation explicitly because nothing physically enforces it
- ownership & boundaries; data ownership & access
- integration (sync/async; allowed/forbidden); API & event contracts
- security; data privacy / PII; audit; compliance
- error handling / resilience, logging / observability — only where architecturally
  constrained
- current-vs-target (brownfield) divergences

**The Context must include:** purpose & scope; read order; authority order; must-read
sources; minimal system overview; **architecture style & modularity rules**; ownership
& boundary rules; data ownership; integration rules; API & event rules; security /
privacy / audit / compliance constraints; current-vs-target guidance; Brownfield Rule
Cards (only where needed); prohibited shortcuts; ask-first triggers; links to SAD /
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
known legacy / suspected drift / ask-first. Create Brownfield Rule Cards **only** when
current implementation and target direction differ in a way that could mislead AI.

## Phase 7 — Produce output

1. `docs/architecture/ai-context.md`
2. `docs/engineering/ai-coding-guidelines.md`
3. `ai-enablement/context-manifest.yaml`, if missing or incomplete
4. root AI instruction proposal, if missing or incomplete
5. Validation Report
6. Brownfield Rule Cards, only where needed
7. Contributor Decisions Needed

## Output format

```markdown
# ai-context-bootstrap Result

## Decision
Choose one: Completed | Completed with TBDs | Blocked | Analyze-only report produced

## Files created or updated
- <file>

## Context sources discovered
| Source | Path | Evidence type | Authority |
|---|---|---|---|

## Blocking question
Ask exactly one question, or write: None.

## Contributor decisions needed
| Decision | Reason | Blocking? | Suggested owner |
|---|---|---|---|

## Brownfield Rule Cards created
| Topic | Status | Reason |
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
