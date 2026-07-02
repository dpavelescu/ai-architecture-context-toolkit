---
name: read-context-manifest
description: >-
  Locate a repo's AI-context inputs via the source map — the thin, structured map of where guidance,
  sources, and the clarifications ledger live, plus the authority order. When it's present, resolve each
  entry (explicit path, else search fallback); when it's absent, discover by convention and propose the
  map. Returns a structured source list, not prose. Use during discovery in bootstrap, check, or update.
---

`ai-enablement/context-manifest.{yaml,yml}` is the **source map**: a thin, structured map of a repo's
AI-context inputs, outputs, and authority order — a recommendation, not an enforced schema.

**Resolve each entry.** Use its explicit path (globs allowed). If an entry is omitted, or its path
resolves to nothing, fall back to the conventional location below by name and search for it. (An entry
may name an external locator for future MCP resolution; treat an unresolved external entry as not-found
and search.) Bound discovery by `scope` — `repo`, an `areas` name, or a path/glob — and stop once the
requested inputs are located.

**Absent or unresolved — discover by convention, then propose.** Bounded by `scope`:
- root instruction file — `AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`
- existing guidance — `docs/architecture/ai-context.md`, `docs/engineering/ai-coding-guidelines.md`
- clarifications ledger — `docs/architecture/ai-clarifications.md`
- architecture sources — SAD, ADRs, diagrams, decision logs
- formal specs — OpenAPI, AsyncAPI, data, UI, security/privacy/audit/compliance
- code evidence — representative services/modules, tests, CI checks
- supporting memory — `docs/solutions/`, prior reports

Then propose the source map (draft pending approval): use discovered paths, mark unknowns `TBD`,
omit what doesn't apply, don't invent paths.

**Shape:**
```yaml
guidance:        { context: <path>, guidelines: <path>, clarifications: <path> }  # OUTPUTS the toolkit maintains
sources:         { sad: [..], adrs: [..], specs: [..], diagrams: [..] }            # INPUTS (read-only)
code:            { representative: [..], known_legacy: [..], known_target: [..] }
solution_notes:  [..]
authority:       [sad, adrs, specs, diagrams, code]                               # high → low; code is lowest
areas:           { <name>: [paths] }                                              # optional named scopes for scope=<area>
```

**Returns** a structured source list — one entry per resolved source as `{ name, type, path, authority }`
(`type` = the category it resolved under: sad/adr/spec/diagram/code), ordered high→low authority (code
lowest) — plus the guidance and clarifications-ledger paths. When the map is absent, returns the proposed
draft above.
