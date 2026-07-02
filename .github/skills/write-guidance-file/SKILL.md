---
name: write-guidance-file
description: >-
  How to draft or refresh the two thin AI-facing files — the AI Architecture Context and the
  AI Coding Guidelines: provenance header, rule shape, weighting, the ordered section lists, and
  what to exclude. Use while writing either file in bootstrap or guidance-update. Pairs with
  assess-coverage (what to cover); this skill is how to write it.
---

## Inputs

- **target-file** — which file to draft or refresh: the AI Architecture Context (`ai-context.md`) or the AI Coding Guidelines (`ai-coding-guidelines.md`).
- **coverage-decisions** — the **final rules** from `assess-coverage` (concerns an approved source settles). Open decisions are not drafted here — they go to the clarifications ledger.
- **sources** — the resolved source list from the source map (`read-context-manifest`): SAD, ADRs, specs, diagrams, and canonical in-repo examples, each with its authority and path to link.

## Procedure

1. **Use conventional, stable headings.** Do not reinvent the structure.
2. **Write short declarative bullets, not prose.**
3. **Link to sources, do not copy them.**
4. **Shape each rule as one line:** the imperative rule, a link to the source that owns the full detail, and an inline *ask-first if …* trigger. Point to a canonical in-repo example ("mirror this") when one exists. State each rule once and cross-link; do not restate.
5. **Make weight visible:** mark non-negotiables as **Never/Always** and preferences as **Prefer**.
6. **Open each file with its provenance header (one line):**
   - Context: *generated & maintained by the toolkit; mirrors (never overrides) the sources in the source map; evolve via `ai-guidance-update`.*
   - Guidelines: *generated & maintained by the toolkit; applies the Architecture Context in code (doesn't redefine it); evolve via `ai-guidance-update`.*
7. **Follow the Output section order as a baseline, not a ceiling.** Write only sections with real content; omit concerns that don't apply; never pad. Keep the listed sections in their given relative order; append any added section after the listed ones (or in the nearest topical position), never reordering the listed set.
8. **Add a section for any repo- or domain-specific concern that matters even if it isn't listed** (e.g. multi-tenancy, performance/SLAs, i18n), kept concrete and repo-specific, not generic advice.
9. **Cover every relevant concern, but shrink to a pointer where an artifact already covers one well.** Thin does not mean narrow.
10. **Keep the file clean and final.** Write only decided rules, ready for downstream use. **Exclude** all plumbing — statuses, placeholders, `TBD`/proposed markers, and open decisions (those live in the clarifications ledger, never here) — and full SAD content, long ADR rationale, large copied diagrams, implementation plans, story-specific detail, unapproved decisions, generic engineering advice.

## Output

**ai-context.md** — sections in this order:
- Purpose & scope — covered areas
- Read order & authority order
- Must-read sources — pointer to the source map (no prose source list)
- System overview (minimal)
- Technology & platform — languages/frameworks/runtimes/datastores; allowed/forbidden
- Architecture style & modularity
- Boundaries & ownership
- Data ownership & access
- Integration & communication — sync/async; API & event ownership
- Security, privacy, audit & compliance
- Resilience & error handling
- Logging & observability
- Current-vs-target & Brownfield Guardrails
- Prohibited shortcuts & ask-first triggers

**ai-coding-guidelines.md** (don't redefine architecture — link to the Context) — sections in this order:
- Scope control — minimal change; reuse before adding
- Technology & libraries — approved stack; how to add a dependency
- Repository structure & placement
- Layering & module conventions
- Naming
- DTOs, mapping & validation
- Error handling
- Contract-change workflow — API / event / data / UI
- Testing
- Logging & observability
- Security & privacy coding rules
- Brownfield implementation rules
- Prohibited behaviors & ask-first triggers
- Reference implementations & links
