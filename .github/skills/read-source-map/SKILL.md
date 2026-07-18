---
name: read-source-map
description: >-
  Locates and selects a repo's AI-context inputs — existing guidance, the clarifications ledger, and
  the approved sources that actually carry the information the guidance needs — and hands back a
  structured list rather than prose. Reads the source map when the repo has one, otherwise searches
  conventional locations; selects sources by what they cover, not by filename. Discovery in bootstrap,
  check, and update all comes through here.
---

## Inputs

- **repo root** — the repository to discover AI-context inputs in.
- **scope** — bound on what to discover (default `repo`): `repo` (whole repo), a name from the map's `areas`, or a path/glob.
- **relevance criteria** — the concerns the guidance captures, as listed in `assess-coverage` (boundaries, data ownership, integration, contracts, security/privacy/compliance, technology, architecture style, resilience, current-vs-target).

## Procedure

1. Look for `ai-enablement/source-map.{yaml,yml}` at the repo root. **Present → resolve through it. Absent → search by convention (step 4).** Never author or propose a map.
2. **Resolve each entry.** Use its explicit path (globs allowed). If an entry is omitted, or its path resolves to nothing, fall back to the conventional location the convention search names and search for it. (An entry may name an external locator for future MCP resolution; treat an unresolved external entry as not-found and search.) Stop once the requested inputs are located.
3. **Bound by `scope` — implementation only.** `scope` selects which code the run examines: an `areas` name resolves to its paths; a path/glob bounds to itself; `repo` is the whole tree. It never bounds the documents. Guidance, the ledger, and the root instruction file are repo-level, and the approved sources resolve repo-wide however narrow the scope. Where a source is organized per area, favour the parts covering the scope; never drop a source for sitting outside the scope's paths.
4. **Search by convention** — for a repo with no map, and for anything a map leaves unresolved, under step 3's `scope` rule:
   - root instruction file — `AGENTS.md`, `CLAUDE.md`, `.github/copilot-instructions.md`
   - existing guidance — `docs/architecture/ai-architecture-rules.md`, `docs/engineering/ai-coding-guidelines.md`
   - clarifications ledger — `docs/architecture/ai-clarifications.md`
   - architecture sources — SAD, ADRs, diagrams, decision logs
   - formal specs — OpenAPI, AsyncAPI, data, UI, security/privacy/audit/compliance
   - code evidence — representative services/modules, tests, CI checks
   - supporting memory — `docs/solutions/`, prior reports

   These names are hints for where to look, not the relevance test — step 5 is. Without a map, authority follows the default order: sad → adrs → specs → diagrams → code (code lowest).
5. **Select the relevant subset — the sources that carry information the Architecture Rules and Coding Guidelines need.** Look only for information about the captured concerns; a source carrying none is not relevant. Take the type structure as given — a map's keys (`sad`/`adrs`/`specs`/`diagrams`/`code`, with supporting `solution_notes` kept separate), or the convention categories from step 4 when there's no map; don't re-type or invent a structure. Keep the entries that speak to a concern — judged by each entry's `covers`/`description`, the doc's stated purpose, or (no map) its content; name and location are weak hints only — and drop those that speak to none (a test plan, runbook, onboarding guide). **When genuinely unsure, keep it** — `assess-coverage` still decides whether it settles anything.
6. **Sample code evidence; never read whole trees.** Code resolves as *representative* evidence: entry points and public APIs, the in-scope modules/services, the largest or most-recently-changed areas, and their tests. Read excerpts — enough to evidence a pattern, not an audit of the tree.

When a repo does have a map, read it tolerantly — use whatever keys exist, ignore the rest, resolve each entry path-else-search, and treat each entry's `covers`/`description` as the relevance hint step 5 checks:

```yaml
guidance:  { rules: <path>, guidelines: <path>, clarifications: <path> }   # OUTPUTS the toolkit maintains
sources:                                                                    # INPUTS (read-only)
  sad:      [{ path: <path>, covers: [<concern>, ...], description: <what it settles> }]
  adrs:     [{ path: <path>, covers: [..] }]
  specs:    [..]
  diagrams: [..]
code:      { representative: [..], known_legacy: [..], known_target: [..] }
solution_notes: [..]
authority: [sad, adrs, specs, diagrams, code]                              # high → low; code is lowest
areas:     { <name>: [paths] }                                             # optional named scopes for scope=<area>
```

`covers`/`description` are optional per entry; absent, step 5 judges relevance from the doc itself.

## Output

Named buckets, not prose. Return every bucket; a caller takes only the ones it needs:

- **sources** — the sources selected as relevant to the guidance's concerns (by what they cover, not by name): one entry per source as `{ name, type, path, authority, covers }`, `type` being one of sad | adr | spec | diagram, ordered high→low authority, followed by **code** entries (always lowest authority; evidence that proposes, never ratifies). `covers` names the concerns the source speaks to.
- **guidance** — paths to an existing AI Architecture Rules, AI Coding Guidelines, and any Brownfield Guardrails; absent if the repo has none. Presence means a caller is re-baselining, not starting fresh.
- **ledger** — path to the clarifications ledger; the conventional path if it doesn't exist yet.
- **memory** — solution notes and prior reports; empty if none. **Supporting memory only: never authoritative, never a settling source.** It is not part of `sources`.
- **root-instruction-file** — its path, or absent.

**When nothing relevant resolves** — empty `sources` and no `guidance`, naming where it looked. The caller decides what to do.
