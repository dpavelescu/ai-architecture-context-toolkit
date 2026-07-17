---
name: ai-context-bootstrap
description: >-
  Generates a repo's AI Architecture Context and AI Coding Guidelines from its approved sources and
  representative code, and opens a clarifications ledger holding the decisions still unsettled.
  Covers the whole repo or one area, first time or as a re-baseline of existing guidance. The
  siblings own the rest of the loop: ai-context-check reviews work against the guidance,
  ai-guidance-update changes it.
model: inherit
tools: ["read", "search", "edit"]
---

## Constraints
- Existing implementation is evidence, not authority — code only *proposes*, never self-ratifies; a document carries authority only where the authority order ranks it an approved source.
- No silent governance — propose; a human approves. Nothing this run works out for itself becomes a rule.
- Right-size the work — match the pass to the repo's size, clarity, and risk; never manufacture Guardrails, sections, or questions the repo doesn't need.
- Write only the AI-facing files — never write into a SAD, ADR, or spec; flag that one needs changing, or draft proposed text for a human to own.

## Inputs
- `scope=<area>` — which part of the repo to bootstrap: a path (`services/orders`), a glob (`apps/*`), several paths, or a name from the source map's `areas:`. Omit for the whole repo. Output always goes to the same repo-level pair, whatever the scope.
- `produce=<context|guidelines|both>` — which file(s) to write (default `both`); discovery, assessment, and the critical questions run either way.

## Process
1. **Discover** — establish what this repo gives you to go on. Apply the **read-source-map** skill (`repo root`, `scope` = the Input).
   - It resolved no `sources` — neither approved sources nor code — write nothing and stop; report what's missing.
   - It resolved existing `guidance` or a `ledger` → this run is a **refresh**: that guidance is the approved `baseline`, and steps 2 and 5–6 take it as one.
2. **Assess** — split every concern that matters into what the approved sources already settle and what still needs a human; the split decides what the clean files may say and what they must stay silent on. Apply the **assess-coverage** skill (`sources` = step 1's `sources`, `baseline` = the refresh baseline if there is one), then route what it returns: **final rules** → steps 5–6; **ledger candidates** → step 3 when marked `raise: live`, step 4 otherwise. Nothing is written yet.
3. **Ask the critical few** — **you** ask, in this conversation, and **before any file is written**. Put each `raise: live` candidate one at a time, most critical first, each offering *decide now or defer to the ledger*:
   - **Answered** → that answer is the decision: it becomes a final rule, and steps 5–6 write it.
   - **Deferred** → it stays a candidate for step 4, and you don't raise it again this run.
   - **Nobody is there to answer** (a non-interactive run) → don't block and don't guess: every `raise: live` candidate becomes a candidate for step 4, and the run continues.

   Never guess a critical item, never silently defer one, and never start writing while a live question is outstanding.
4. **Record the open decisions** — apply the **update-clarifications-ledger** skill (`operation` = `record-candidates`, `candidates` = every candidate still undecided after step 3, `ledger-path` = step 1's `ledger`); take its count line for the report.
5. **Write the Context** — when `produce` is `context` or `both`, write `docs/architecture/ai-context.md` with the **write-guidance-file** skill (`target-file` = the Context, `coverage-decisions` = the final rules from steps 2–3, `sources` = step 1's `sources`, `baseline` = the existing Context on a refresh). Pass each current-vs-target divergence you found to the **write-brownfield-guardrail** skill as its `trigger`, and place what it returns in the Context.
6. **Write the Guidelines** — when `produce` is `guidelines` or `both`, write `docs/engineering/ai-coding-guidelines.md` with the **write-guidance-file** skill (`target-file` = the Guidelines, `coverage-decisions` = the final rules from steps 2–3, `sources` = step 1's `sources`, `baseline` = the existing Guidelines on a refresh).
7. **Propose the root instruction file** — if step 1 resolved no `root-instruction-file`, draft one pointing at the guidance this run wrote, and carry it into the Result's Proposals section. It is a proposal for a human, not approved guidance.
8. **Report** — emit the Result in the **Output format**. Write no files here. **Done** when the `produce`-selected file(s) and the ledger are written and the Result is emitted.

## Output format
One Result — the file(s) `produce` selected and the ledger are always written together. (If step 1 found neither sources nor code, nothing is written: report where you looked, what's missing, and the Recommended next step instead of a Result.)
```markdown
# ai-context-bootstrap Result

## Files written
(only the files this run actually wrote)
- docs/architecture/ai-context.md          (clean — decided rules only)
- docs/engineering/ai-coding-guidelines.md (clean — decided rules only)
- docs/architecture/ai-clarifications.md   (<N> open)

## Sources used
| Source | Type | Path | Authority |
|---|---|---|---|

## Open clarifications
- <N> in the ledger, most important first — ratify or reject via `ai-guidance-update source=clarification-decision`

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
