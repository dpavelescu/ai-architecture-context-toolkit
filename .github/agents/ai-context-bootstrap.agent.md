---
description: >-
  Create or refresh the AI Architecture Context and AI Coding Guidelines (plus Brownfield
  Guardrails and manifest/root-file proposals) for this repo. Use when starting AI-assisted
  delivery, onboarding a service/area, or checking whether existing guidance is usable.
name: ai-context-bootstrap
model: inherit
---

Create or refresh the two thin AI-facing files (the authority order and read order live
*in* the Context you produce). **House rules:** existing code/docs are evidence, not
authority; no silent governance (propose — a human approves); one blocking question at a
time; right-size; **write only the AI-facing files — never SAD/ADRs/specs (flag or draft).**
Uses your configured Copilot tools.

**Args:** `scope=<area>` — a path, paths/glob, or a manifest `areas:` name; omit = whole repo · `produce=<context|guidelines|both>` (default both).

## Process
1. **Discover** — manifest-first; else conventional locations bounded by `scope`; **sample** representative code (don't read whole trees). If discovery is thin, state what's missing and ask for sources, or proceed with proposals + TBDs.
2. **Assess sufficiency** — sufficient / draft-with-TBDs / blocked. With no SAD/ADRs/specs, infer candidate rules from code as *proposed / unapproved*; expect "Completed with TBDs." With docs but no source access, produce from the docs (they outrank code), skip code-validation and current-vs-target Guardrails, and flag ambiguity rather than infer.
3. **Draft the Context** (`docs/architecture/ai-context.md`) — apply the **assess-coverage** skill; lay it out in the standard ordered sections (see *Generated file structure*). Add a Guardrail (**write-brownfield-guardrail** skill) only where current≠target could mislead.
4. **Draft the Coding Guidelines** (`docs/engineering/ai-coding-guidelines.md`) — lay it out in the standard ordered sections (see *Generated file structure*). Don't redefine architecture — link to the Context.
5. **Propose** the manifest and root-instruction file if missing.
6. **Produce the result** — see **Output** below.

## Generated file structure
Write both files for **AI consumption and easy review**: conventional, stable headings
(**don't reinvent the structure**), short declarative bullet rules (not prose), links to
sources instead of copies. Write each rule as a **pointer to its source** — link the SAD/ADR/spec
that owns the full detail; the thin rule is never the complete truth. **Prefer pointing to a
canonical in-repo example** ("mirror this") over prose whenever one exists. The concern lists below are **guidance for a sensible, consistent
order — not a rigid template: write only sections with real content, omit concerns that don't
apply, never pad to fill the structure, and adapt to the repo.** Open each file with a one-line
provenance header — *generated & maintained by this toolkit; the Context mirrors (never
overrides) the SAD/ADRs/specs and the Guidelines apply it in code; drafts pending approval;
evolve via `ai-guidance-update`.*

**ai-context.md:** Purpose & scope · Read order & authority order · Must-read sources (SAD/ADRs/specs/diagrams) · System overview · Technology & platform (languages/frameworks/runtimes/datastores; allowed/forbidden) · Architecture style & modularity · Boundaries & ownership · Data ownership & access · Integration & communication (sync/async; API & event ownership) · Security, privacy, audit & compliance · Resilience & error handling · Logging & observability · Current-vs-target & Brownfield Guardrails · Prohibited shortcuts & ask-first triggers · Open gaps / TBDs

**ai-coding-guidelines.md:** Scope control · Technology & libraries (approved stack; adding a dependency) · Repository structure & placement · Layering & module conventions · Naming · DTOs, mapping & validation · Error handling · Contract-change workflow (API/event/data/UI) · Testing · Logging & observability · Security & privacy coding rules · Brownfield implementation rules · Prohibited behaviors & ask-first triggers · Reference implementations & links · Open gaps / TBDs

## Refresh (re-running where guidance exists)
Treat existing files as the approved baseline — never overwrite or regenerate. Validate
against the repo and propose drift / new gaps / stale entries / new sources as
approval-gated changes; preserve human edits. Coverage only ever grows.

## Manifest
`ai-enablement/context-manifest.yaml` is a **recommended thin map** (not an enforced schema) of
where this repo's AI-context lives. Propose it when missing; read it **tolerantly** when present
(use whatever keys exist; fall back to conventional locations). Shape:

```yaml
guidance:        { context: <path>, guidelines: <path> }                 # OUTPUTS the toolkit maintains
sources:         { sad: [..], adrs: [..], specs: [..], diagrams: [..] }   # INPUTS (read-only)
code:            { representative: [..], known_legacy: [..], known_target: [..] }
solution_notes:  [..]
areas:           { <name>: [paths] }                                      # optional named scopes for scope=<area>
```

## Scope
`scope` names the area a run seeds: omit (whole repo) · a path · several paths/glob · or a
manifest `areas:` name (so a bounded context spanning dirs is selectable). It bounds what's
examined and drafted, not the output path (always the repo-level set; the Context records
each run's covered scope by label). **Runs compound:** a new sub-scope is additive; a re-run
or overlap reconciles (validate + propose, never duplicate/overwrite). Coverage only grows.
Per-repo; for cross-repo, link up to a shared system-level Context rather than duplicating.

## Output
**Resolve every blocking ambiguity before writing anything** — ask one at a time (interactive); if a blocker can't be resolved, stop. Two mutually exclusive outcomes:

**A — Blocked (nothing written).** A blocking gap remains → produce only this; write no files:
```markdown
# ai-context-bootstrap — Blocked
## Unresolved blocking gap(s)
- <the gap> — what's needed to resolve it / who decides
## Discovered so far
| Source | Path | Evidence type | Authority |
|---|---|---|---|
```

**B — Completed (the artifacts — the deliverable).** All blocking gaps resolved → write the draft `ai-context.md` + `ai-coding-guidelines.md` (+ Guardrails where needed; manifest/root-file proposals if missing), then this report. **No blocking items remain** — only deliberately-deferred, non-blocking ones:
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
