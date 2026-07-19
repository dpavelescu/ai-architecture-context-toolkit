---
name: write-guidance-file
description: >-
  Writes or refreshes the two thin AI-facing files — the AI Architecture Rules and the AI Coding
  Guidelines — from the final rules an assessment settled: provenance header, one-line rule shape,
  visible weighting, and the ordered section lists. Merges into an existing file rather than
  regenerating it. Never writes an open decision or an undecided proposal; those live in the
  clarifications ledger.
---

## Inputs

- **target-file** — which file to write or refresh: the AI Architecture Rules (`docs/architecture/ai-architecture-rules.md`) or the AI Coding Guidelines (`docs/engineering/ai-coding-guidelines.md`). On a refresh, use the path discovery resolved.
- **coverage-decisions** — the **final rules** from `assess-coverage` (concerns an approved source settles). Open decisions are not written here — they go to the clarifications ledger.
- **baseline** — the existing approved file, when one exists. Absent on a first write.
- **sources** — the resolved source list from the source map (`read-source-map`): SAD, ADRs, specs, diagrams, and canonical in-repo examples, each with its authority and path to link.

## Procedure

1. **With a `baseline`, merge into it; never regenerate it.** Keep its human edits and approved rules as they stand, and apply only what `coverage-decisions` evidences — a rule it corrects, a rule it adds. Dropping or rewriting an approved rule needs approval, so leave one an approved source no longer supports where it is and let the caller raise it. Without a `baseline`, write the file fresh.
2. **Use conventional, stable headings.** Do not reinvent the structure.
3. **Write short declarative bullets, not prose.**
4. **Link to sources, do not copy them.**
5. **Shape each rule as one line:** the imperative rule, a link to the source that owns the full detail, and an inline *ask-first if …* trigger. Point to a canonical in-repo example ("mirror this") when one exists. State each rule once and cross-link; do not restate.
6. **Make weight visible:** mark non-negotiables as **Never/Always** and preferences as **Prefer**.
7. **Open each file with its provenance header (one line):**
   - Architecture Rules: *generated & maintained by the toolkit; mirrors (never overrides) the approved sources it links; evolve via `ai-guidance-update`.*
   - Guidelines: *generated & maintained by the toolkit; applies the Architecture Rules in code (doesn't redefine them); evolve via `ai-guidance-update`.*
8. **With no `coverage-decisions` and no `baseline`, that header is the whole file** — report it back as header-only so the caller can say so; never pad it with generic advice or undecided proposals. With a `baseline` and no `coverage-decisions`, return the baseline unchanged and report it unchanged.
9. **Follow the Output section order as a baseline, not a ceiling.** Write only sections with real content; omit concerns that don't apply; never pad. Keep the listed sections in their given relative order; append any added section after the listed ones (or in the nearest topical position), never reordering the listed set.
10. **Add a section for any repo- or domain-specific concern that matters even if it isn't listed** (e.g. multi-tenancy, performance/SLAs, i18n), kept concrete and repo-specific, not generic advice.
11. **Cover every relevant concern, but shrink to a pointer where an artifact already covers one well.**
12. **Keep the file clean and final.** Write only decided rules, ready for downstream use. **Exclude** all plumbing — proposal statuses, placeholders, `TBD`/proposed markers, and open decisions (those live in the clarifications ledger, never here) — and full SAD content, long ADR rationale, large copied diagrams, implementation plans, story-specific detail, unapproved decisions, generic engineering advice. Keep a Brownfield Guardrail's `Status` and `Ask first`; send the decision an `Ask first` waits on to the ledger.

## Output

**ai-architecture-rules.md** — sections in this order:
- Purpose & scope — covered areas
- Read order & authority order
- Must-read sources — a pointer to the source map when the repo has one; otherwise the approved sources themselves, linked (never a prose source list)
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

**ai-coding-guidelines.md** (don't redefine architecture — link to the Architecture Rules) — sections in this order:
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
