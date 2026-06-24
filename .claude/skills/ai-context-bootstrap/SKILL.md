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

Runs in the main conversation so it can ask blocking questions interactively. It calls the
shared skills **read-context-manifest**, **assess-coverage**, **write-guidance-file**, and
**write-brownfield-guardrail** for the reusable capabilities — invoke each where the Process
names it rather than re-deriving its logic here.

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
`target_output_dir=<path>`. If no mode is given, use `interactive`. `produce` selects which
file(s) to draft — `context` runs step 3, `guidelines` runs step 4, `both` runs both; discovery
and assessment (steps 1–2) always run, because the Coding Guidelines apply the Architecture
Context and must read it either way.

## Scope

`scope` is which part of the repo to bootstrap: a path (`services/orders`), a glob (`apps/*`),
several paths, or a name from the manifest's `areas:`. Omit it for the whole repo. A path or a
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
   and few or no Guardrails. Reserve the full treatment for large, ambiguous, or
   high-risk repos. Don't manufacture Guardrails, sections, or questions the situation
   doesn't need.

Additional constraints: do not create a second SAD; do not copy long architecture
rationale; keep the AI-facing guidance thin. Use repo-relative paths everywhere; never
absolute paths. Write only the AI-facing layer (Context, Guidelines, Guardrails,
manifest/root-file, candidate solution notes); never write SAD/ADRs/specs/tracker items —
flag or draft those for a human.

## Process

1. **Discover** — locate inputs with the **read-context-manifest** skill (manifest first,
   conventional fallback, bounded by `scope`). Sample representative code — entry points and
   public APIs, the in-scope modules/services, the largest or most-recently-changed areas, and
   their tests; read excerpts, not whole trees. If discovery is thin, don't draft silently:
   state what's missing and either ask the user to point at sources (interactive) or record an
   insufficiency note and proceed in no-source mode with proposals + TBDs (headless).
   - **If guidance already exists (refresh).** When discovery finds an existing `ai-context.md`
     / `ai-coding-guidelines.md` (or approved Guardrails), this run is a refresh — a health-check
     and re-baseline, never a regeneration: treat the existing files as the approved baseline
     (preserve human edits, filled-in TBDs, approved entries); validate against the repo and
     propose drift, new gaps, stale entries, and new sources as approval-gated changes; never
     delete or rewrite a rule without approval (flag it instead). Per-learning evolution belongs
     to `ai-guidance-update`.
2. **Assess sufficiency** — apply the **assess-coverage** skill over the full concern checklist,
   then resolve before drafting:
   - **Detect** what's missing as much as present: an architecturally significant concern no
     source covers; an ambiguous statement (more than one reading); a self-contradicting source;
     a cross-source conflict.
   - **Classify** each as blocking or non-blocking — blocking = it would force an assumption on
     something architecturally significant (ownership, data, cross-service communication,
     API/event authority, security, privacy, audit, compliance, technology/platform,
     current-vs-target, or a needed architecture decision).
   - **Clarify** blocking items first. `interactive`: ask the single most critical question,
     wait, then the next, in order of criticality, until none remain — only what genuinely needs
     a human. `headless` (or the human can't answer now): write no files and emit the Blocked
     agenda.
   - **Gate**: draft only when no blocking item remains — the files then fold in every
     clarification and stand complete; never substitute an assumption for a missing
     architecturally significant concern. With no SAD/ADRs/specs, infer lower-risk rules from
     code as *proposed* (architecturally significant → ask, don't infer); with no code access,
     draft from the docs and skip code validation and current-vs-target Guardrails.
3. **Draft the Context** — write `docs/architecture/ai-context.md` per the **write-guidance-file**
   skill. Validate against representative code and add a Guardrail (the **write-brownfield-guardrail**
   skill) only where current implementation and target direction differ enough to mislead the AI.
4. **Draft the Guidelines** — write `docs/engineering/ai-coding-guidelines.md` per the
   **write-guidance-file** skill. Don't redefine architecture; link to the Context.
5. **Propose** — if missing, propose the manifest (**read-context-manifest** skill) and the
   repo's root instruction file.
6. **Produce output** — emit the Blocked or Completed result (see **Output format**). A Completed
   run writes the Context, the Guidelines, the manifest/root-instruction proposals (if missing),
   Guardrails (only where needed), and the report.

## Output format

The run ends one of two ways, set by the **Gate** in step 2.

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

**Completed** — the deliverable is the drafted files (drafts pending approval) plus this report;
no blocking items remain, only non-blocking deferrals:

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
