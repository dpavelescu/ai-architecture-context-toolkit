---
name: read-context-manifest
description: >-
  Locate a repo's AI-context inputs via the manifest — the thin map of where guidance and sources
  live. When it's present, read it tolerantly to find inputs; when it's absent, fall back to
  conventional locations and propose the file. Use during discovery in bootstrap, check, or update.
---

`ai-enablement/context-manifest.{yaml,yml}` is a thin map of a repo's AI-context inputs and
outputs — a recommendation, not an enforced schema.

**Present — read it tolerantly.** Treat it as the authoritative map of inputs: read what it lists,
use whatever keys exist, and fall back to conventional locations for anything it omits. Don't hunt
beyond what's needed.

**Absent — fall back, then propose.** Discover by convention, bounded by `scope`:
- root instruction file — `AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`
- existing guidance — `docs/architecture/ai-context.md`, `docs/engineering/ai-coding-guidelines.md`
- architecture sources — SAD, ADRs, diagrams, decision logs
- formal specs — OpenAPI, AsyncAPI, data, UI, security/privacy/audit/compliance
- code evidence — representative services/modules, tests, CI checks
- supporting memory — `docs/solutions/`, prior reports

Then propose the manifest (draft pending approval): use discovered paths, mark unknowns `TBD`,
omit what doesn't apply, don't invent paths.

**Shape:**
```yaml
guidance:        { context: <path>, guidelines: <path> }                 # OUTPUTS the toolkit maintains
sources:         { sad: [..], adrs: [..], specs: [..], diagrams: [..] }   # INPUTS (read-only)
code:            { representative: [..], known_legacy: [..], known_target: [..] }
solution_notes:  [..]
areas:           { <name>: [paths] }                                      # optional named scopes for scope=<area>
```
