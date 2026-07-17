---
name: read-source-map
description: >-
  Locates a repo's AI-context inputs — existing guidance, the approved sources, the clarifications
  ledger, and the authority order — and hands back a structured list rather than prose. Reads the
  source map when the repo has one, and otherwise searches the conventional locations. Discovery in
  bootstrap, check, and update all comes through here.
---

Look for `ai-enablement/source-map.{yaml,yml}` at the repo root. **Present → resolve through it.
Absent → search by convention.** Never author or propose a map.

**Resolve each entry** (map present). Use its explicit path (globs allowed). If an entry is omitted, or
its path resolves to nothing, fall back to the conventional location the convention search names and
search for it. (An entry may name an external locator for future MCP resolution; treat an unresolved
external entry as not-found and search.) Stop once the requested inputs are located.

**Bound by `scope` — implementation only.** `scope` selects which code the run examines: an `areas` name
resolves to its paths; a path/glob bounds to itself; `repo` is the whole tree. It never bounds the
documents. Guidance, the ledger, and the root instruction file are repo-level, and the approved sources
resolve repo-wide however narrow the scope. Where a source is organized per area, favour the parts
covering the scope; never drop a source for sitting outside the scope's paths.

**Sample code evidence; never read whole trees.** Code resolves as *representative* evidence: entry
points and public APIs, the in-scope modules/services, the largest or most-recently-changed areas, and
their tests. Read excerpts — enough to evidence a pattern, not an audit of the tree.

**Search by convention** — for a repo with no map, and for anything a map leaves unresolved, under the
same `scope` rule:
- root instruction file — `AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`
- existing guidance — `docs/architecture/ai-context.md`, `docs/engineering/ai-coding-guidelines.md`
- clarifications ledger — `docs/architecture/ai-clarifications.md`
- architecture sources — SAD, ADRs, diagrams, decision logs
- formal specs — OpenAPI, AsyncAPI, data, UI, security/privacy/audit/compliance
- code evidence — representative services/modules, tests, CI checks
- supporting memory — `docs/solutions/`, prior reports

Search for each name; a source kept outside its conventional location still resolves. Without a map,
authority follows the default order: sad → adrs → specs → diagrams → code (code lowest).

**Shape** — when a repo does have a map, read it tolerantly: use whatever keys exist, ignore the rest,
resolve each entry path-else-search.

```yaml
guidance:        { context: <path>, guidelines: <path>, clarifications: <path> }  # OUTPUTS the toolkit maintains
sources:         { sad: [..], adrs: [..], specs: [..], diagrams: [..] }            # INPUTS (read-only)
code:            { representative: [..], known_legacy: [..], known_target: [..] }
solution_notes:  [..]
authority:       [sad, adrs, specs, diagrams, code]                               # high → low; code is lowest
areas:           { <name>: [paths] }                                              # optional named scopes for scope=<area>
```

**Returns named buckets**, not prose. Return every bucket; a caller takes only the ones it needs:

- **sources** — the inputs that can settle a concern: one entry per resolved source as
  `{ name, type, path, authority }`, `type` being one of sad | adr | spec | diagram, ordered high→low
  authority, followed by **code** entries (always lowest authority; evidence that proposes, never
  ratifies).
- **guidance** — paths to an existing AI Architecture Context, AI Coding Guidelines, and any Brownfield
  Guardrails; absent if the repo has none. Presence means a caller is re-baselining, not starting fresh.
- **ledger** — path to the clarifications ledger; the conventional path if it doesn't exist yet.
- **memory** — solution notes and prior reports; empty if none. **Supporting memory only: never
  authoritative, never a settling source.** It is not part of `sources`.
- **root-instruction-file** — its path, or absent.

When nothing resolves, return empty `sources` and no `guidance`, naming where you looked; the caller
decides what to do.
