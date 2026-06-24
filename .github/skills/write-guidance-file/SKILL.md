---
name: write-guidance-file
description: >-
  How to draft or refresh the two thin AI-facing files — the AI Architecture Context and the
  AI Coding Guidelines: provenance header, rule shape, weighting, the ordered section lists, and
  what to exclude. Use while writing either file in bootstrap or guidance-update. Pairs with
  assess-coverage (what to cover); this skill is how to write it.
---

Write both files for AI consumption and easy review: conventional, stable headings (don't
reinvent the structure); short declarative bullets, not prose; links to sources instead of copies.

**Rule shape** — one line per rule: the imperative rule, a link to the source that owns the full
detail, and an inline *ask-first if …* where relevant. Prefer pointing to a canonical in-repo
example ("mirror this") when one exists. State each rule once and cross-link — don't restate.

**Weight** — make it visible: non-negotiables as **Never/Always**, preferences as **Prefer**.

**Provenance header** — open each file with one line:
- Context: *generated & maintained by the toolkit; mirrors (never overrides) the SAD/ADRs/specs; drafts pending approval; evolve via `ai-guidance-update`.*
- Guidelines: *generated & maintained by the toolkit; applies the Architecture Context in code (doesn't redefine it); drafts pending approval; evolve via `ai-guidance-update`.*

**Section order** — the lists below are a sensible, consistent order and a baseline, not a ceiling:
write only sections with real content, omit concerns that don't apply, never pad — and **add a
section for any repo- or domain-specific concern that matters even if it isn't listed** (e.g.
multi-tenancy, performance/SLAs, i18n), kept concrete and repo-specific, not generic advice.
Thin ≠ narrow: cover every relevant concern, but shrink to a pointer where an artifact already
covers one well.

**ai-context.md:**
- Purpose & scope — covered areas vs `TBD`
- Read order & authority order
- Must-read sources — SAD / ADRs / specs / diagrams
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
- Open gaps / TBDs

**ai-coding-guidelines.md** (don't redefine architecture — link to the Context):
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
- Open gaps / TBDs

**Exclude:** full SAD content, long ADR rationale, large copied diagrams, implementation plans,
story-specific detail, unapproved decisions, generic engineering advice.
