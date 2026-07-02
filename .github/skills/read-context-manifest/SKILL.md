---
name: read-context-manifest
description: >-
  Locate a repo's AI-context inputs via the source map — the thin, structured map of where guidance,
  sources, and the clarifications ledger live, plus the authority order. When it's present, resolve each
  entry (explicit path, else search fallback); when it's absent, discover by convention and propose the
  map. Returns a structured source list, not prose. Use during discovery in bootstrap, check, or update.
---

## Inputs

- **repo root** — the repository to discover AI-context inputs in.
- **scope** — bound on what to discover (default `repo`): `repo` (whole repo), a name from the map's `areas`, or a path/glob.

## Procedure

`ai-enablement/context-manifest.{yaml,yml}` is the **source map**: a thin, structured map of a repo's AI-context inputs, outputs, and authority order — a recommendation, not an enforced schema.

1. Look for `ai-enablement/context-manifest.{yaml,yml}` at the repo root.
2. **Resolve each entry.** Use its explicit path (globs allowed). If an entry is omitted, or its path resolves to nothing, fall back to the conventional location below by name and search for it. (An entry may name an external locator for future MCP resolution; treat an unresolved external entry as not-found and search.) Stop once the requested inputs are located.
3. **Bound by `scope`:** when `scope` names an `areas` entry, resolve its paths; when it is a path/glob, bound discovery to it; otherwise (`repo`) discover repo-wide.
4. **Absent or unresolved.** Discover by convention, bounded by `scope`:
   - root instruction file — `AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`
   - existing guidance — `docs/architecture/ai-context.md`, `docs/engineering/ai-coding-guidelines.md`
   - clarifications ledger — `docs/architecture/ai-clarifications.md`
   - architecture sources — SAD, ADRs, diagrams, decision logs
   - formal specs — OpenAPI, AsyncAPI, data, UI, security/privacy/audit/compliance
   - code evidence — representative services/modules, tests, CI checks
   - supporting memory — `docs/solutions/`, prior reports

   Then propose the source map as a draft (pending approval) from this shape: use discovered paths, mark unknowns `TBD`, omit what doesn't apply, do not invent paths.

   ```yaml
   guidance:        { context: <path>, guidelines: <path>, clarifications: <path> }  # OUTPUTS the toolkit maintains
   sources:         { sad: [..], adrs: [..], specs: [..], diagrams: [..] }            # INPUTS (read-only)
   code:            { representative: [..], known_legacy: [..], known_target: [..] }
   solution_notes:  [..]
   authority:       [sad, adrs, specs, diagrams, code]                               # high → low; code is lowest
   areas:           { <name>: [paths] }                                              # optional named scopes for scope=<area>
   ```

## Output

- **Map resolved (success):** a structured source list the calling agent consumes — one entry per resolved source as `{ name, type, path, authority }`, where `type` is the source category it resolved under (sad/adr/spec/diagram/code), ordered high→low authority (code lowest) — plus the guidance and clarifications-ledger paths.
- **Map absent or unresolved:** the proposed source-map draft above, pending approval.
