---
name: update-clarifications-ledger
description: >-
  Owns the clarifications ledger, the one file open decisions live in. Records new candidates
  without re-proposing anything already settled, retires decided ones (accepted out of Open,
  rejected to Settled), and preserves human edits. Every read and write of the ledger goes through
  here — bootstrap records into it, ai-guidance-update retires from it.
---

Write the clarifications ledger at `ledger-path` — resolved by `read-source-map`, conventionally
`docs/architecture/ai-clarifications.md`.

**Read the existing ledger first and treat it as the baseline.** Preserve human edits, entry wording,
filled `decision:` lines, and the whole `## Settled — won't re-propose` list. Never regenerate it from
scratch. If it doesn't exist, create it from the Shape given and start from an empty baseline.

**Recording candidates** (`operation` = `record-candidates`) — for each candidate from
`assess-coverage`, already shaped `[<concern>] Proposal / why / raise / decision:`:
- `## Settled` already names the concern → **drop the candidate; never re-propose a settled item.**
- an `## Open` entry already carries the same `[<concern>]` tag → **drop the candidate;** the existing
  entry stands unchanged, keeping its wording and any filled `decision:`.
- otherwise → append it under `## Open` with an empty `decision:`.

Then order `## Open` most important first (`raise: live` entries above `raise: ledger` ones).

**Resolving decisions** (`operation` = `resolve-decisions`) — for each `## Open` entry:
- **accept / edit** → remove it from `## Open`. The caller owns folding the rule into the clean file
  via `write-guidance-file`; if it reports that the fold didn't happen, leave the entry in `## Open`
  and say so — an accepted rule must never fall out of both files.
- **reject** → remove it from `## Open` and add one line to `## Settled — won't re-propose`.
- **empty `decision:`** → leave it untouched; it is still open.

A candidate never travels into the Architecture Rules or Guidelines, and nothing decided stays in `## Open`.

**Shape:**

```markdown
# Clarifications ledger

Open decisions for this repo's AI guidance. Fill a `decision:` (accept / edit / reject), then ratify
with `/ai-guidance-update source=clarification-decision mode=apply-approved-update`.

## Open

[<concern>] Proposal: <recommended rule>.
why: <evidence of variation / impact / framework standardization>.
raise: <live | ledger>
decision:

## Settled — won't re-propose

- [<concern>] <rejected proposal> — rejected: <reason, if given>
```

Report one count line — `open:` is the number of `## Open` entries after the write; counters that
don't apply to this run's `operation` emit `0`:

```
open: <N> · added: <N> · dropped-as-settled: <N> · accepted-out: <N> · rejected-to-settled: <N>
```
