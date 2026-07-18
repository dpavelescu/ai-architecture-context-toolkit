# AI Architecture Rules & AI Coding Guidelines — A Generation & Maintenance Playbook

**Audience:** Architects, tech leads, developers, QA, platform engineers, engineering managers

---

## The one idea to remember

> **Make approved architecture usable by AI agents — and let each story leave the guidance a little better than it found it.**

We do this with two thin, AI-facing files (plus optional helpers). They don't replace your architecture docs. They sit on top and tell the AI *how to apply* them safely.

One principle underpins it: **each unit of work should make the next one easier.** Context and guidance accumulate — you set the context up once, use it on every story, and fold back only the few learnings worth keeping. Over time the AI gets safer and you write less, not more.

---

## What you're doing → what to run

Find your situation, run the matching workflow, act on what it hands back — the workflows are tool-neutral (see the per-surface invocation below for the exact Copilot/Claude form).

| You're doing / your situation | Run | You get back | Then |
|---|---|---|---|
| Starting AI work in a repo (no guidance yet) | `ai-context-bootstrap` | Clean Architecture Rules + Guidelines + a **clarifications ledger** of open decisions | Work the ledger → ratify with `ai-guidance-update` |
| Onboarding a service/area, or architecture moved on | `ai-context-bootstrap scope=<area>` (refresh) | Updated clean files; drift + new gaps added to the ledger | Ratify the new ledger items |
| Deciding the clarifications a run left you | fill `decision:` in the ledger → `ai-guidance-update source=<ledger> mode=apply-approved-update` | Accepted → folded into clean files; rejected → Settled | Ledger shrinks; nothing re-nags |
| About to implement / reviewing a PR | `ai-context-check work=<…>` | Alignment report; open-ledger concerns flagged **not-yet-binding** | Fix, or raise the decision |
| You learned something worth keeping | `ai-guidance-update source=<learning> mode=analyze-only` | Analysis + smallest suggested update | Approve → apply |
| Sources are stale, code is the truth now | bootstrap refresh, or `ai-guidance-update` (code-vs-stale-source) | Corrected rule + upstream SAD/ADR flagged stale | Human updates the source |
| Many repos | Appendix | Shared system Architecture Rules + per-repo link | — |

The steady state is **Check ↔ ratify-the-ledger**; you bootstrap once.

---

## Tool scope & terms

This playbook is the **tool-neutral reference** — it describes the toolkit by what each part *does*: three **workflows** (bootstrap, check, update) and four optional **reviewers**. The concepts are identical across coding agents; only the packaging differs:

- **Claude Code** — workflows are *skills* (`.claude/skills/`), reviewers are *sub-agents* (`.claude/agents/`); invoked as `/slash` commands.
- **GitHub Copilot** — all seven are *custom agents* (`.github/agents/`) plus five shared *Agent Skills* (`.github/skills/`); invoked from the agent picker.

For concreteness the examples below use the Claude form (`skill`, `/ai-context-bootstrap …`). Read **"skill"** as "the workflow" and **"sub-agent"** as "the delegated reviewer" — they map directly to the Copilot build (see the README for the per-tool layout).

The activity map above is agnostic; here is how each surface translates it directly.

### Claude Code

Workflows are `/slash` skills (`.claude/skills/`); reviewers are sub-agents (`.claude/agents/`). The three invocations:

- `/ai-context-bootstrap [scope=<area>] [produce=<context|guidelines|both>]`
- `/ai-context-check work=<story|artifact|plan|pr|diff|solution-note> [scope=<area>]`
- `/ai-guidance-update source=<…> [mode=<analyze-only|apply-approved-update>]`

### GitHub Copilot

The seven are custom agents (`.github/agents/`) plus five shared Agent Skills (`.github/skills/`), invoked from the agent picker; the reviewers run as parallel subagents (`/fleet` in Copilot CLI). Same parameters as above — `scope` / `produce` on bootstrap, `work` / `scope` on check, `source` / `mode` on update.

---

## The loop

Everything in this guide is one loop with three steps:

```
                ┌─────────────────────────────────────┐
                │                                       │
                ▼                                       │
   ┌────────────────────┐                              │
   │ 1. BOOTSTRAP        │  Set up the thin context     │
   │   (once per repo)   │  the AI will read.           │
   └─────────┬──────────┘                              │
             │                                          │
             ▼                                          │
   ┌────────────────────┐                              │
   │ 2. CHECK            │  Before/while you build,     │
   │   (every story)     │  check the work against it.  │
   └─────────┬──────────┘                              │
             │                                          │
             ▼                                          │
   ┌────────────────────┐                              │
   │ 3. UPDATE           │  When a learning lands,      │
   │   (only when needed)│  fold it back in.  ──────────┘
   └────────────────────┘
```

The day-to-day cycle is **Check ↔ Update** — you bootstrap once, then keep checking stories and occasionally folding a learning back in. Re-bootstrap only for a new service or a major reset.

| Step | Skill | When |
|---|---|---|
| Bootstrap | `ai-context-bootstrap` | Once per repo/service, then rarely |
| Check | `ai-context-check` | Every story, plan, or PR |
| Update | `ai-guidance-update` | Only when a learning will change future AI behavior |

Optional reviewer agents (architecture, engineering, brownfield, and — for regulated/contract-heavy work — contract & compliance) plug into the Check and Update steps when reviews get broad. Start without them.

---

## 1. Why these two files exist

To act, an AI agent reads what's in the repo — the SAD, ADRs, specs, and code — and fills the gaps with assumptions. When guidance is thin it **(a) misinterprets** an artifact, **(b) silently resolves a conflict** between two sources, or **(c) invents a missing constraint** instead of asking. Brownfield code is the sharpest case — approved patterns, tolerated legacy, half-finished migrations, and shortcuts all look alike — but the same happens with stale docs, draft ADRs, and partial specs. So we tell the AI what it can't safely infer.

**AI Architecture Rules** tells the AI:

- which architecture docs to read, and what has authority
- which constraints are non-negotiable
- which current patterns must **not** be copied
- where current code differs from the target direction
- when to ask instead of guessing

**AI Coding Guidelines** turn those constraints into concrete coding behavior.

### Where each thing lives

| Artifact | Audience | Job |
|---|---|---|
| SAD | Humans | Explain the architecture |
| ADRs | Humans + AI | Record decisions and rationale |
| Formal specs (OpenAPI, AsyncAPI, data, security…) | Humans + AI | Define contract truth |
| **AI Architecture Rules** | **AI + reviewers** | **Tell AI how to apply architecture safely** |
| **AI Coding Guidelines** | **AI + developers** | **Tell AI how to implement consistently** |
| Code | Humans + AI | Evidence of what was built — not proof it was approved |

### What the Architecture Rules must cover — and at what level

This is the Architecture Rules' actual job. **"Thin" does not mean "narrow":** it must **cover every concern where the AI could misinterpret** and that's relevant to the work — ownership, data, integration, contracts, security, privacy, audit, compliance, architecture style, and any brownfield divergence. What keeps it thin is *how* it covers each one, decided against your existing artifacts (SAD, ADRs, LLD, security/privacy requirements, specs).

**Surface by relevance, not by inventory.** Don't catalog every existing pattern. A concern earns a place only if it clears one test: (a) variation is **already evidenced** in code, (b) it would be **high-impact** if it varied, or (c) it's a **general / framework-level standardization**. Drop concerns that are uniform *and* low-impact *and* local — they don't need a rule.

Each surfaced concern then routes one of two ways:

| If an approved source… | …then the concern becomes |
|---|---|
| covers it **actionably** (the AI can act on the pointer as-is) | a **final rule** in the clean file — **points** to the source (must-read + one-line operational pointer), don't restate |
| covers it but **too abstractly / buried** to act on, yet the direction is settled | a **final rule** — a thin operational rule that makes the settled source actionable, links back |
| covers it **ambiguously** (open to >1 reading), or **conflicts** with another source | a **ledger candidate** — a proposal (recommended rule) + rationale for a human to decide; nothing is picked silently |
| **doesn't cover it** (or it lives only in code) | a **ledger candidate** — code may *propose* a rule (lowest authority, never self-ratifying), captured as a proposal + rationale; the SAD/ADR/requirement may need creating (a governance item, never silent) |

So the Architecture Rules is an **index + gap-filler**: settled sources shrink to pointers (final rules in the clean file); everything unsettled — ambiguous, uncovered, conflicting, or only code-evidenced — becomes a **ledger candidate** a human decides. Nothing undecided enters the clean file.

**Concerns to sweep** (any that are relevant **and** could be misinterpreted):

- ownership & boundaries · data ownership & access
- integration (sync/async; allowed/forbidden) · API & event contracts
- **security · data privacy / PII · audit · compliance**
- technology & platform (languages/frameworks/runtimes/datastores; allowed/forbidden)
- architecture style & modularity
- resilience & error handling · logging & observability — *only where architecturally constrained*
- current-vs-target (brownfield) divergences

This list is a **baseline, not a ceiling** — also surface any concern specific to this repo or domain that could mislead an AI even if it isn't listed (e.g. performance/latency SLAs, multi-tenancy & data isolation, internationalization/accessibility, caching, concurrency). For each: decide **point / restate-actionably / fill-and-flag**. **Every concern is equal** — security, privacy, and contracts deserve the same care as ownership or architecture style. Skip one only when it's genuinely irrelevant — never because it's "not architecture," and never drop a real concern for being "off-list."

The two examples below just show *how* a concern becomes an actionable rule. They're illustrations, not the focus.

#### Example: ownership across teams

With a single team, ownership is mostly a formality. With **multiple teams** it becomes a hard boundary: the code the AI reads may belong to another team, so it must not silently change their service, read their database, or alter a shared contract — even when that's the easy path (and an architect can't catch every such case across every team at PR time). The Architecture Rules states **who owns each thing and how others get to it**:

| Asset | Owner | How others use it |
|---|---|---|
| order-service, orders DB | Orders team | Others read via `GET /orders` or the `OrderPlaced` event |
| customers DB, customer PII | Customers team | Others read via `GET /customers/{id}` or the `CustomerUpdated` event; PII is never duplicated |
| payment.events (AsyncAPI) | Payments team | Contract changes need Payments-team approval |

- ❌ **Without it**, the AI reads the `customers` table directly because the connection is reachable — silently coupling Orders to Customers.
- ✅ **With it**, the AI uses `GET /customers/{id}` or the `CustomerUpdated` event — and if neither exists, it **asks the Customers team** instead of reaching in.

*Capture rule:* name the **owning team** and **how others get the data / whom to ask** — that's what turns "ask first" into something the AI can act on.

#### Example: boundaries by architecture style

The architecture **style** decides what kind of boundary exists and how strict it is — and it matters even with one team. The sharpest case is the **modular monolith**: everything compiles together, so the AI can `import` across module lines and it will build *and pass tests*. Nothing physically stops it, which is exactly why the boundary must be written down.

| Style | The boundary that matters | The AI must **not** |
|---|---|---|
| **Modular monolith** | Module isolation inside one deployable | import another module's internals; call across modules except through the public API; read its tables |
| **Microservices / distributed** | Network/service line + independent data stores | add a new synchronous dependency by default; read another service's DB; change a shared contract unilaterally |
| **Layered** | Dependency direction (e.g. `api → application → domain → infrastructure`) | introduce a reverse or skip dependency; import a framework into the domain layer |

Most systems combine these; capture only the lines that constrain the work at hand — usually layering plus the module or service boundary.

---

## 2. Six principles

1. **Architecture Rules constrains Coding Guidelines.** Guidelines apply architecture; they never redefine it. *Example: Architecture Rules says "no service reads another service's DB." Guidelines show how repositories, clients, DTOs, events, and tests respect that.*

2. **What's in the repo is evidence, not authority.** A pattern in the code — or a statement in a stale or draft doc — doesn't make it valid for new work; only an approved source does.

3. **Keep the Architecture Rules thin — but not narrow.** It must *cover* every concern the AI could misinterpret (see §1 "What the Architecture Rules must cover"), but where an artifact already covers one well, shrink to a pointer instead of restating it. *Rule of thumb: if the SAD changes but AI behavior doesn't, the Architecture Rules doesn't change.*

4. **The Architecture Rules is a planning-time fitness function.** It forces the AI to check boundaries (ownership, data access, allowed coupling, API/event impact, current-vs-target, security/privacy/audit/compliance, ask-first cases) *before* proposing a solution. Some rules can later become real checks (ArchUnit, dependency/contract tests, lint, CI).

5. **Humans still review.** This improves what reaches review; it doesn't remove review. A solution can pass tests, look clean, and still expand coupling or violate intent. Make key constraints visible *before* the plan, not only at PR time.

6. **No silent governance.** AI may *propose* classifications, Guardrails, or updates. AI must never silently approve architecture, security, privacy, audit, compliance, contract, or coding-standard decisions. Those need human approval.

---

## 3. Read order vs. authority order

The AI **reads** AI-facing files first — but they are **not** the highest authority.

**Read order** (start cheap, go deep as needed):

1. Root instruction file (`AGENTS.md` / `CLAUDE.md` / `.github/copilot-instructions.md`)
2. Source map, if present
3. AI Architecture Rules
4. AI Coding Guidelines
5. The SAD, ADRs, specs, diagrams, and code those files point to
6. Relevant implementation code and tests
7. Solution notes (supporting memory only)

**Authority order** (this is the canonical list — referenced everywhere else as "the authority order"):

1. Approved story / requirement / acceptance criteria / Story Artifact decisions
2. Formal specs (OpenAPI, AsyncAPI, Figma, data, security, privacy, audit, compliance)
3. Approved ADRs
4. SAD and approved architecture docs
5. AI Architecture Rules + approved Brownfield Guardrails
6. AI Coding Guidelines
7. Approved reference implementations
8. Local code and tests (implementation evidence)
9. Solution notes (supporting memory only)

**When sources conflict — or a source contradicts itself — the AI does not resolve it silently; it raises a context conflict.** The team then decides what's stale (Architecture Rules? SAD? code?) and whether a Guardrail, guidance update, ADR, SAD, or spec change is needed (the AI writes only the AI-facing ones, with approval; ADR/SAD/spec stay human-owned).

---

## 4. What goes where

| File | Put in it | Keep out of it |
|---|---|---|
| **SAD / ADRs** | Narrative, diagrams, decisions, rationale, trade-offs, deployment & context views | — |
| **Formal specs** | API/event/UI/data contracts; security, privacy, audit, compliance rules | — |
| **AI Architecture Rules** | Only what changes AI behavior: authority order, must-read sources, non-negotiable constraints, ownership & data rules, allowed/forbidden integration, current-vs-target, "do not copy" rules, ask-first triggers, links back | Full SAD content, long rationale, big diagrams, coding conventions, plans, story details, generic engineering advice |
| **AI Coding Guidelines** | Coding behavior, scope control, repo/layering conventions, testing, naming, DTO/mapping/validation/error-handling, contract-change workflow, logging/observability, AI-specific prohibitions, ask-first triggers | Architecture rationale — link to the Architecture Rules instead |

---

## 5. The minimum set

Start with the smallest useful set. Don't add area-specific guidance until repeated use proves you need it.

```
docs/architecture/ai-architecture-rules.md
docs/engineering/ai-coding-guidelines.md
docs/architecture/ai-clarifications.md   (the clarifications ledger the toolkit maintains)
AGENTS.md  (or CLAUDE.md / .github/copilot-instructions.md)

# optional but recommended
ai-enablement/source-map.yaml   (the optional source map)
```

### Source map (optional)

A small, stable **structured map** so skills can *resolve* sources instead of asking you to paste them. It's a map, not another architecture doc — keep it tiny. Each source resolves **by explicit path, else search fallback**: when a path is given it's used as-is; when a path is omitted or resolves to nothing, the toolkit searches by conventional location. The map also carries the **authority order** and the path to the **clarifications ledger**. (External locators — MCP or remote sources — are possible later; nothing to configure today.) Use real discovered paths; mark unknowns `TBD`.

```yaml
# source-map.yaml — a thin, structured map of where this repo's
# AI-context lives, so the agents don't hardcode paths. Each entry resolves by explicit
# path, else search fallback (omitted / unresolved → search conventional locations).
# Fill in real paths; mark unknowns TBD; omit what doesn't apply. A map, not another doc.

# OUTPUTS — the AI-facing files the toolkit produces and maintains:
guidance:
  rules:          docs/architecture/ai-architecture-rules.md
  guidelines:     docs/engineering/ai-coding-guidelines.md
  clarifications: docs/architecture/ai-clarifications.md   # the open-decisions ledger

# AUTHORITY — the order the toolkit trusts sources in (highest → lowest):
authority: [sad, adrs, specs, diagrams, code]

# INPUTS — read-only; the toolkit reads these, never writes them:
sources:                               # authoritative, human-owned artifacts
  sad:      [docs/architecture/sad.md]
  adrs:     [docs/architecture/adrs/]
  specs:    [docs/contracts/openapi/, docs/contracts/asyncapi/]   # API/event/data/UI/security/privacy/audit/compliance
  diagrams: [docs/architecture/diagrams/]
code:                                  # what the agents sample to ground on (not the whole repo)
  representative: [services/, libs/]   # entry points / key modules
  known_legacy:   []                   # patterns the AI must NOT copy
  known_target:   []                   # exemplary "do it like this" code
solution_notes: [docs/solutions/]      # supporting memory (lowest authority)

# Named scopes for bootstrap/check, so a bounded context that spans dirs is selectable:
areas:
  # payments: [services/payments, libs/payments-sdk]
  # eventing: [services/*/events, libs/event-bus]
```

The parts: **`guidance`** (the OUTPUTS the toolkit writes, now including the `clarifications` ledger), the **`authority`** order it trusts sources in, and the INPUTS it reads (`sources` / `code` / `solution_notes`), plus optional `areas` for named scopes. Constraints and the *detailed* authority reasoning still live in the Architecture Rules — the map carries only the ordered list.

The map is **optional and yours to author** — a *recommended thin map, not an enforced schema*, and nothing generates it for you. Every agent **reads** it tolerantly (use whatever keys exist, resolve each entry by path-else-search) and **searches conventional locations when it's absent or partial** — which is the normal path, not a degraded one. Write one when convention-searching keeps landing in the wrong place, or when you want named `areas:` scopes; skip it otherwise.

### The clarifications ledger

`docs/architecture/ai-clarifications.md` is a **lean, ephemeral markdown worklist that holds only open decisions** — the one place undecided things live, keeping the Architecture Rules and Guidelines clean and final. Bootstrap writes it alongside the two clean files; a human works it; `ai-guidance-update` folds accepted items back out of it. Both go through the shared **`update-clarifications-ledger`** capability, which owns the file's structure — so recording candidates and retiring decided items behave the same either way.

Each open item is one entry:

```
[<concern>] Proposal: <recommended rule>.
why: <evidence of variation / impact / framework standardization>.
raise: <live | ledger>   (critical → live; all others → ledger)
decision:
```

Below the open items, a `## Settled — won't re-propose` section keeps one line per **rejected** item, so re-runs don't nag you about something you already turned down.

The ledger **trends to empty**: an accepted item leaves by being folded into the clean files (smallest change) and dropped from Open; a rejected item moves to Settled; nothing decided stays in the ledger.

### Root instruction file (template)

Keep it small. It mostly points at the other files and states the core rules (authority, brownfield, conflict, questions, governance, memory).

```markdown
# AI Working Instructions

## Read order
Before analysis, planning, coding, or review, read:
1. ai-enablement/source-map.yaml   # the source map, if this repo has one
2. docs/architecture/ai-architecture-rules.md
3. docs/engineering/ai-coding-guidelines.md
Then read the SAD, ADRs, specs, diagrams, code, and tests they reference.
Open decisions live in the clarifications ledger (docs/architecture/ai-clarifications.md), not in the two files above.

## Using the Architecture Rules
- The Architecture Rules' rules are **pointers, not the whole truth** — for architecturally significant detail, read the linked source. If a rule and its source disagree, the **source wins**: raise a conflict.
- When the Architecture Rules names a **canonical example**, mirror it rather than inventing a new shape.

## Source authority
Use the authority order (see the guide). Approved requirements win; solution notes are memory only.

## Brownfield rule
Existing code is evidence, not approved architecture. Don't copy patterns the
Architecture Rules or a Brownfield Guardrail marks as current-but-not-target. If current and
target differ and no Guardrail covers it — and it touches architecture, ownership,
data, contracts, security, privacy, audit, or compliance — ask first.

## Conflict rule
If sources conflict — or a source contradicts itself — don't decide silently. Raise a context conflict.

## Question rule
Ask one blocking question at a time. Put non-blocking questions in the report.

## Governance rule
Human approval is required for changes to: architecture, ownership, data ownership,
API/event contracts, security, privacy, audit, compliance, coding standards, or
Brownfield Guardrail status.

## Solution memory rule
docs/solutions/ is supporting memory only. It never overrides requirements, specs,
SAD, ADRs, the Architecture Rules, the Guidelines, or Guardrails.
```

### One repo or many

This works the same whether you have one repo or many — it isn't built around either.

- **Single repo (the default):** the files above live at the repo root and point to your local SAD/ADRs/specs.
- **Multiple repos:** keep one **central, system-level Architecture Rules** (in a platform/architecture repo) that governs cross-repo concerns and points to the cross-repo SAD/ADRs; each repo runs the same toolkit, and its source map simply **links up** to that central Architecture Rules as a must-read source, adding only repo-local specifics.

Same mechanism, one level up — **no special multi-repo mode**, and you add the central layer only if cross-repo governance actually needs it. (See the *Appendix — Scaling to many repos* for a worked example.)

### Generated file structure (the two files)

Both files are **optimized for AI consumption and easy to review** — conventional, stable headings (don't reinvent the structure), short declarative bullet rules (not prose), links to sources instead of copies. Write each rule as a **pointer to its source** — link the SAD/ADR/spec that owns the full detail; the thin rule is never the complete truth. **Prefer pointing to a canonical in-repo example** over prose whenever one exists. **Don't repeat content** — state each rule once and cross-link rather than restating. **Rule shape:** one line per rule — the imperative rule, a link to its source, and an inline *ask-first if …* where relevant (plus a canonical example to mirror when one exists). Make the **weight** visible: non-negotiables as **Never/Always**, preferences as **Prefer**. The concern lists below are **a baseline for a sensible, consistent order — not a rigid template and not a ceiling:** include only sections with real content, **omit concerns that don't apply, never pad to fill the structure,** adapt to the repo, and **add a section for any repo- or domain-specific concern that matters even if it isn't listed** (e.g. multi-tenancy, performance/SLAs, i18n) — kept concrete, not generic advice. Each file holds **only decided, ready-to-use rules** — no statuses, placeholders, `TBD`/proposed markers, or "open gaps" sections; every open decision lives in the clarifications ledger instead, so these two files stay clean and final. Each opens with a one-line provenance header (*generated & maintained by the toolkit; the Architecture Rules mirrors — never overrides — the SAD/ADRs/specs, the Guidelines apply it in code; these files hold only decided rules — open decisions live in the clarifications ledger; evolve via the update skill*), then the applicable concerns:

**`docs/architecture/ai-architecture-rules.md`** — Purpose & scope — covered areas · Read order & authority order · Must-read sources — a pointer to the source map, or the approved sources themselves when there's no map · System overview · Technology & platform (languages, frameworks, runtimes, datastores; allowed/forbidden) · Architecture style & modularity · Boundaries & ownership · Data ownership & access · Integration & communication (sync/async; allowed/forbidden; API & event ownership) · Security, privacy, audit & compliance · Resilience & error handling · Logging & observability · Current-vs-target & Brownfield Guardrails · Prohibited shortcuts & ask-first triggers

**`docs/engineering/ai-coding-guidelines.md`** — Scope control · Technology & libraries (approved stack; adding a dependency) · Repository structure & placement · Layering & module conventions · Naming · DTOs, mapping & validation · Error handling · Contract-change workflow (API/event/data/UI) · Testing · Logging & observability · Security & privacy coding rules · Brownfield implementation rules · Prohibited behaviors & ask-first triggers · Reference implementations & links

---

## 6. Constraints for every skill

These apply to all three skills — defined once here, referenced by each.

1. **Discover first.** Inspect the repo for the root file, source map, Architecture Rules, Guidelines, relevant SAD/ADRs/specs, code/tests, and solution notes. Never ask the user to paste something that's discoverable.
2. **Ask only critical items live — one at a time.** Only **critical** concerns (security, privacy, compliance, data ownership, a needed architecture decision) may interrupt, and each is asked on its own, offering **decide now or defer to the ledger**. Everything else becomes a **ledger candidate** with no live question. Prefer multiple-choice when you do ask.
   > *Example:* "Which source is the authority for cross-service communication? **A)** SAD §4.3 · **B)** ADR-012 · **C)** current code · **D)** decide later — defer to the ledger."
3. **Use safe defaults.** Code is just another source for *proposals* — lowest authority, never self-ratifying: it can propose even a significant rule, but only as a **ledger candidate**, never a final rule. If a missing decision touches architecture, ownership, data, contracts, or security/privacy/audit/compliance, never invent the answer — a **critical** one is asked live (decide-now-or-defer); everything else goes to the ledger with a recommended rule. **Nothing undecided ever enters the clean Architecture Rules/Guidelines.**
4. **Interactive by nature; writing is gated.** Every skill asks a critical blocking question only when something genuinely needs a human; otherwise it records the item in the ledger or its report (in a non-interactive context it never blocks — undecided items just land in the ledger). Writing is the gated dial: `ai-context-bootstrap` **always writes the clean Architecture Rules/Guidelines plus the clarifications ledger together** (the only exception: if discovery finds neither sources nor code, it writes nothing and says what's missing — there is no "Blocked, nothing written" gate); `ai-context-check` and the reviewers are read-only; `ai-guidance-update` is read-only in `analyze-only` (default) and writes only in `apply-approved-update`. Default to the safest option.
5. **Classify evidence.** Tag every source: approved requirement / Story Artifact / formal spec / approved ADR / SAD / Architecture Rules / Guidelines / Guardrail / approved reference impl / current code / known legacy / suspected drift / candidate learning / supporting memory. **Current code is never "approved architecture" unless an approved source confirms it.**
6. **Produce durable output.** Always emit a file or report — an updated/draft artifact, a validation/alignment/analysis report, a blocking question, or a stopped state with reason. **Chat history is never the source of truth.**
7. **Right-size the work.** Match the amount of ceremony to the size, clarity, and risk of the work. A small repo or an aligned, low-risk change gets a compact pass — skip phases that add nothing and prefer a short report (or "no change needed"). Reserve the full multi-phase treatment for large, ambiguous, or high-risk work. Don't manufacture Guardrails, reports, or questions the situation doesn't need. *(This is the principle that keeps the whole approach lightweight: match ceremony to the work.)*
8. **Write only the AI-facing layer.** Skills write the Architecture Rules, Coding Guidelines, Guardrails, the **clarifications ledger**, and candidate solution notes (supporting memory) — governance-affecting changes always gated by human approval. They *draft* a root instruction file for a human when the repo has none, and they never author the source map: it's optional and yours, read when present and convention-searched when absent. They **never write SAD, ADRs, formal specs, or tracker items** (Jira / Story Artifacts); for those they only *flag* that a change is needed or *draft* proposed text for a human to review and commit.

---

## 7. Brownfield Guardrails

Use a Guardrail **only** when current code and target direction differ in a way that could mislead the AI. Don't make them for aligned situations.

**Statuses:** `Use current` (current is approved) · `Use target` (new work follows target even if code differs) · `Target not ready` (target exists, don't move there unless scoped) · `Ask first` (don't decide alone — needs human clarification).

**Template:**

```markdown
## Brownfield Guardrail: <Topic>
Status: <Use current | Use target | Target not ready | Ask first>
Source:           <SAD / ADR / spec / decision>
Current state:    <what exists today>
Target direction: <what new work should use, if known>
Rule for new work:    <what to do for new work>
Rule for existing code:<what to keep or must not change>
Do not copy:      <misleading legacy pattern>
Ask when:         <conditions needing clarification>
```

**Example:**

```markdown
## Brownfield Guardrail: Cross-service communication
Status: Use target
Source: SAD §4.3, ADR-012
Current state:    Some services still call each other directly over REST.
Target direction: New cross-service business comms use async events.
Rule for new work:    Use async events unless sync is explicitly approved.
Rule for existing code:Existing REST calls may stay; don't migrate unless in scope.
Do not copy:      Don't add a new sync service-to-service call just because similar ones exist.
Ask when:         A story seems to need a new sync dependency; eventual consistency
                  isn't acceptable; data ownership is unclear.
```

---

## 8. Using the guidance during delivery (the Check step)

1. Read root file → source map → Architecture Rules → Guidelines → the clarifications ledger → relevant SAD/ADRs/specs/diagrams
2. Analyze the story; identify affected architecture areas and relevant Guardrails
3. Produce a plan **constrained by the guidance**
4. Implement only after the plan is accepted
5. Review the implementation against the plan and guidance
6. Capture a reusable learning **only if it's actually reusable**

A concern that is still **open in the ledger** is treated as **not yet binding** — flag it, but don't enforce it as a rule and don't read silence as approval (silence ≠ approval).

---

## 9. Evolving the guidance (the Update step)

This is where guidance evolves — deliberately, not on every story.

When a candidate learning appears — a review finding, an approved ADR or spec change, or a captured solution note — `update` asks **two questions**. (Note: it does **not** depend on counting how often a pattern occurred — you usually can't know that.)

1. **Altitude — does it belong at governance level?** Place the learning on the ladder below. Most learnings sit *below* the line and need no change to the AI-facing guidance.
2. **Conflict / drift — does it clash with approved direction?** If it contradicts the Architecture Rules, Guidelines, ADRs, or specs, that's drift to be **mediated**, not a pattern to bless.

**The altitude ladder** — route each learning to its home:

| Learning | Home | Governance level? |
|---|---|---|
| Story-specific decision | Story Artifact / Jira | below |
| Implementation detail | PR / plan / solution note | below |
| Candidate / unproven learning | Solution note | below |
| Reusable coding convention | AI Coding Guidelines | **yes** |
| Architecture rule (changes AI behavior) | AI Architecture Rules | **yes** |
| Repeated brownfield ambiguity | Brownfield Guardrail | **yes** |
| Decision rationale | ADR | yes — human-owned |
| Contract change | Formal spec | yes — human-owned |

Only the **yes** rows touch governance, and only the first three of those are AI-facing files this toolkit writes; ADR/SAD/spec are human-owned (see *Two-speed governance* below).

**When a learning conflicts with current guidance:** don't update silently. Classify the conflict, name the affected source of truth, recommend one action (no update / update Architecture Rules / update Guidelines / Guardrail / raise ADR / update spec), and **apply only approved updates.**

### The promotion gate

A candidate is **not approved guidance.** The **clarifications ledger is the carrier of open decisions**, and `ai-guidance-update` is the gate that ratifies them — deciding whether, and where, a candidate becomes a rule:

- **Ratifying a filled ledger is the main path.** A human fills each entry's `decision:` (accept / edit / reject); running `ai-guidance-update` with `source=clarification-decision` treats **the filled ledger itself as the approval** — accepted items fold into the clean Architecture Rules/Guidelines and leave Open, rejected items drop to Settled.
- **Analyze first (for out-of-band learnings).** For a learning that isn't already in the ledger, the default is `analyze-only`: it classifies the candidate by altitude, checks for conflict/drift, and produces a proposal with a **"human approval required"** flag. It never auto-promotes — a captured note does not silently become a rule.
- **Most candidates don't enter the Architecture Rules.** Route by the altitude ladder above: only a *behavior-changing architecture rule* reaches the Architecture Rules, a reusable convention reaches the Guidelines, and most learnings reach neither (or "no update needed").
- **Promote thin, by reference.** When a learning is promoted, write a **one-line operational rule that links back to its source** — never copy the source's detail into governance. The detailed memory stays in the note; the Architecture Rules/Guidelines holds only the binding rule. This is what keeps governance from becoming a second, bloated copy of your notes.
- **Human approves, then it writes.** Only `apply-approved-update`, with explicit approval (a filled ledger counts), makes the smallest change and preserves the link to the approved source.

`ai-context-check` is the proactive counterpart — it *detects* violations before they ship; `ai-guidance-update` is what turns a confirmed learning into an approved rule.

### Two-speed governance: the Architecture Rules vs SAD/ADRs

A new direction must never be *decided* in the AI Architecture Rules — the authority order ranks ADRs and the SAD **above** it. Truth flows one way: **decision (ADR / SAD / spec / approved story) → AI Architecture Rules (an operational mirror that links back).** The Architecture Rules is a fast reflection of approved architecture, not the system of record for a decision.

That splits an update into two cases:

- **The decision already exists upstream** (an ADR/spec changed, or an approved story set the direction). `update` simply *propagates* it into the Architecture Rules — safe to apply immediately (through the human gate), because it decides nothing; it mirrors the source and links to it.
- **The new pattern has no approved home yet.** The Architecture Rules must not silently become the decision. Choose by stakes:
  - *High-stakes / irreversible / cross-team / contract / security* → **raise an ADR first**; the Architecture Rules stays **Ask first** until the decision exists.
  - *Low-stakes, operational* → a **human-approved Brownfield Guardrail marked `Ask first`, with an owner**, guides the AI now; recommend formalizing the decision (ADR/SAD) when it matters, then reconcile and link.

This is **two-speed governance**: a fast AI-facing lane (Architecture Rules / Guidelines / Guardrails) that `update` writes *with approval*, and a slow authoritative lane (ADR / SAD / specs) that **humans own** — `update` only *flags or drafts* those, never writes them. Provisional entries live in the **clarifications ledger** (not in the clean files), so they resurface for anyone working the ledger until a human decides — and, where formalization is needed, until the upstream decision is formalized and linked.

> **Keep it light.** The provisional-card path (with owners and review dates) is *optional* — it's for teams that already track debt. Until then, just use **ADR-first** (or plain "ask first") and accept a slower cadence. Adopt the lifecycle only once the tracking earns its keep.

---

## 10. The three skills (templates)

The ready-to-use files live in `.claude/skills/<name>/SKILL.md`; the summaries below are the same content in shorter form for readers. If you edit one, treat the file in `.claude/` as canonical. Each skill follows the constraints in §6, so those aren't repeated below.

### Invoking the skills — parameters & modes

The knobs below are the whole surface. **Complexity is not a knob** — every skill *right-sizes automatically* (§6.7): a small or low-risk job gets a compact pass.

| Knob | Values | Meaning | Used by |
|---|---|---|---|
| `scope` | optional; omit = whole repo. A path, paths/glob, or a source map `areas:` name | the area a run focuses on; output stays the single repo-level set; runs compound | bootstrap, check |
| `produce` | `context` / `guidelines` / `both` *(default `both`)* | which artifact bootstrap writes | bootstrap |
| `work` | `story` / `artifact` / `plan` / `pr` / `diff` / `solution-note` | what check is reviewing | check |
| `mode` | `analyze-only` *(default)* / `apply-approved-update` | `analyze-only` proposes without writing; `apply-approved-update` writes only explicitly approved changes | update |
| `source` | `learning` / `solution-note` / `pr-finding` / `review-issue` / `adr` / `spec-change` / `clarification-decision` / `approved-update` | the learning update evaluates (`clarification-decision` = a filled ledger) | update |

**Choosing values:**
- **scope** — start at `service`/`module` for a pilot or a large repo; `repository` for a small, cohesive one.
- **write** — only `ai-guidance-update` writes governance, and only in `apply-approved-update` with explicit approval; its default `analyze-only` just proposes. Bootstrap writes the clean files and the ledger outright, and only proposes a root instruction file; check and the reviewers are read-only.
- **which skill, when** — see *The loop* (top) and §12: bootstrap once, check per story, update only to promote a confirmed learning.

**Interactivity is automatic, not a knob.** Every skill asks a blocking question only for a **critical** item that genuinely needs a human — one at a time, offering decide-now-or-defer; everything else becomes a ledger candidate with no live question. In a non-interactive context (no one to answer), bootstrap still **writes the clean files plus the ledger** — undecided items simply land in the ledger rather than blocking.

### 10.1 `ai-context-bootstrap` — set up the context

**Purpose:** Create or refresh the minimum AI-facing guidance (clean Architecture Rules, clean Guidelines, Guardrails where needed, a root-file proposal if missing, and a clarifications ledger of the open decisions).

**Re-runs are safe.** Where guidance already exists it enters **refresh mode** — it validates the existing files against the repo and routes drift/new gaps into the **clarifications ledger** as candidates (preserving the `Settled` section so it never re-nags), **never overwriting human-approved content**. (Incremental per-learning evolution is `ai-guidance-update`'s job.)

**Use when:** starting AI delivery in a repo; onboarding a new service/module/context/team; creating the first Architecture Rules or Guidelines; checking whether existing guidance is usable. **Not** for story-specific planning (use `ai-context-check`) or guidance evolution (use `ai-guidance-update`).

**Invoke:** `/ai-context-bootstrap [scope=<area>] [produce=<context|guidelines|both>]` (omit `scope` for the whole repo — `area` = a path, paths/glob, or a source map `areas:` name; `produce` defaults to `both` and selects which file(s) get written — discovery, assessment, and the critical questions always run). The skill is interactive: it asks a blocking question only for a critical item (decide-now-or-defer), before it writes anything; everything else lands in the ledger, and so does a critical item when there's no one there to answer.

**Phases:**

1. **Discover** the repo (root file, source map, existing guidance, SAD/ADRs/diagrams, specs, representative code/tests/CI, solution notes). Classify each source. Strategy: resolve via the **source map** if the repo has one (each entry by explicit path, else search fallback), otherwise convention-scan bounded by `scope` — same result either way, and no map is generated; **sample** representative code (don't read the whole tree). Only if discovery finds **neither sources nor code** does it write nothing and say what's missing.
2. **Assess (relevance gate → route):** run the concern checklist and apply the **relevance gate** (see §1) — surface a concern only if variation is already evidenced, it's high-impact, or it's a framework-level standardization; drop concerns that are uniform *and* low-impact *and* local. Don't catalog every existing pattern. Route each surfaced concern: an approved source that settles it actionably → a **final rule** for the clean file; anything unsettled (ambiguous, uncovered, conflicting, or only code-evidenced) → a **ledger candidate** (proposal + rationale). Ask **only critical** items live — one at a time, offering decide-now-or-defer-to-the-ledger; everything else becomes a ledger candidate with no live question. Nothing undecided enters the clean files. Code may *propose* even significant rules (lowest authority → candidate only), never self-ratify.
3. **Ask the critical few** — before anything is written, and only for an item nothing settles and nobody should guess (security, privacy, compliance, data ownership, a needed architecture decision): one at a time, most critical first, each offering decide-now-or-defer-to-the-ledger. An answer becomes a final rule; a deferral stays a candidate; with no one there to answer, it stays a candidate and the run continues rather than blocking. Everything else is never asked. The split is final after this.
4. **Write the clarifications ledger** → `docs/architecture/ai-clarifications.md` via the **`update-clarifications-ledger`** capability: every still-undecided candidate as an entry (`[<concern>] Proposal / why / raise / decision:`), plus the `## Settled — won't re-propose` section (preserved across refreshes so re-runs don't nag).
5. **Write the clean AI Architecture Rules** → `docs/architecture/ai-architecture-rules.md` — **final rules only, no TBDs/placeholders/open-gaps sections**. Write a final rule for each concern a source settles (point / restate-actionably); everything unsettled went to the ledger, not the file. Lay it out in the standard concern sections (§5 "Generated file structure") — guidance, not a rigid template: only sections with real content, omit the rest, don't pad. Each rule a pointer to its source; prefer a canonical in-repo example; don't repeat content. Thin ≠ narrow.
6. **Write the clean AI Coding Guidelines** → `docs/engineering/ai-coding-guidelines.md` — final rules only, same clean-file discipline. Lay it out in the standard coding-concern sections (§5 "Generated file structure"); same writing rules as the Architecture Rules. Don't redefine architecture — link to it.
7. **Validate against representative code.** Classify each pattern (aligned / current-approved / target-ready / target-not-ready / brownfield exception / known legacy / suspected drift / ask-first). Make Guardrails only for misleading current-vs-target gaps. *(Doc-only, no source access: skip this and current-vs-target Guardrails — both need code; rely on the docs and send ambiguity to the ledger rather than infer.)* Then **propose the root instruction file** if the repo has none, pointing at the guidance this run wrote — a draft for a human, never written as approved guidance. The source map is never generated: present → resolved through, absent → convention search.
8. **Report:** the single Result below — clean files + ledger written together (unless discovery found neither sources nor code, in which case nothing is written and it says what's missing).

**Output format** — a single Result (clean files and the ledger are always written together):

```markdown
# ai-context-bootstrap Result
## Files written        (clean context · clean guidelines · ai-clarifications.md with N open)
## Sources used         | Source | Type | Path | Authority |
## Open clarifications  (N — ratify via ai-guidance-update source=clarification-decision)
## Brownfield Guardrails created | Topic | Status | Reason |
## Refresh summary (refresh runs only)   (Kept · Added · Drift → ledger · Stale · Settled preserved)
## Recommended next step
```

_(If discovery finds neither sources nor code, nothing is written and the run reports what's missing instead of a Result.)_

---

### 10.2 `ai-context-check` — use the context on real work

**Purpose:** Check whether a story, analysis, plan, PR, diff, or solution note aligns with the approved Architecture Rules and Guidelines. A planning-time and review-time governance check that catches *locally reasonable but directionally wrong* solutions.

**Use when:** reviewing a Jira story, Story Artifact, AI analysis, plan, PR, diff, solution note, or proposed learning — ideally **before** implementation.

**Invoke:** `/ai-context-check work=<story|artifact|plan|pr|diff|solution-note> [scope=<area>]` (read-only; interactive — it asks one blocking question only when needed, otherwise records it in the report).

**Phases:**

1. **Discover** context (same sources as bootstrap, resolved via the **source map** — path-else-search; read the clarifications ledger too) and classify each. A concern still **open in the ledger** is **not yet binding** — flag it as such; silence ≠ approval.
2. **Understand the work:** intent, affected service/module/context, data ownership, API/event/UI contracts, security/privacy/audit/compliance behavior, changed files, the pattern being used, current-vs-target implications, relevant Guardrails, and any open-ledger concern the work touches. If intent is unclear and risk is material, ask one blocking question.
3. **Delegate the dimension reviews** — for each dimension the work touches, delegate to its reviewer (architecture-boundary / engineering-convention / contract-compliance / brownfield-governance), in parallel; each reviewer owns its dimension's checks. Right-size: skip dimensions the work doesn't touch.
4. **Coverage-gap check (cross-cutting).** Flag any concern the work depends on that the Architecture Rules is silent on and no source artifact covers actionably — note what's missing and where it belongs (Architecture Rules / SAD / ADR / requirement / spec), and recommend `ai-guidance-update`. Don't silently fill it.
5. **Output:** synthesize the reviewers' findings into a Context Alignment Report (incl. coverage gaps).

**Output format:**

```markdown
# Context Alignment Report
## Decision   (Ready | Ready with risks | Needs clarification | Blocked by architecture decision
              | Requires guidance update analysis | Requires formal spec update | Requires ADR/SAD update)
## Reviewed input        (type / reference / scope / mode)
## Summary
## Architecture alignment      | Area | Status | Finding | Evidence |   (status: aligned/risk/conflict/unclear/n-a)
## Coding guideline alignment  | Area | Status | Finding | Evidence |
## Brownfield risks            | Pattern | Classification | Risk | Recommendation |
## Contract & compliance impact| Area | Impact | Finding | Action |
## Source conflicts            | Conflict | Sources | Risk | Recommendation |
## Coverage gaps               | Concern | Missing guidance | Where it belongs (Architecture Rules/SAD/ADR/req/spec) |
## Blocking question     (one, or "None.")
## Non-blocking open points
## Recommended next action  (proceed | proceed with risks | clarify | update plan/PR
                            | run ai-guidance-update analyze-only | raise architecture decision
                            | update spec | create/update Guardrail)
```

**Stop and ask one blocking question — otherwise report it — when:** the solution needs an architecture decision; ownership, data ownership, or contract authority is unclear; security/privacy/audit/compliance impact is unclear; current and target conflict; or the work hits an ask-first trigger.

---

### 10.3 `ai-guidance-update` — capture the learning

**Purpose:** Analyze and apply controlled updates to the Architecture Rules, Guidelines, and Guardrails — so useful learnings don't die in chat/PRs, and unapproved ones don't silently become rules.

**Use when:** a candidate learning could change future AI behavior — a target direction becomes clear; an ADR or spec changes; a reference implementation is approved; a brownfield pattern is misleading the AI; a solution note is worth promoting; a pattern should no longer be copied; or a conflict is detected. Run it to decide *whether* and *where* a learning becomes a rule — not because something recurred a set number of times.

**No baseline yet?** If no Architecture Rules/Guidelines exist, it recommends `ai-context-bootstrap` first and parks the learning as a candidate note — it never fabricates a baseline. (Like all skills, it never writes SAD/ADRs/specs — only flags or drafts them.)

**Invoke:**
- Analyze: `/ai-guidance-update source=<learning|solution-note|pr-finding|review-issue|adr|spec-change|clarification-decision> mode=analyze-only` (default)
- Apply: `/ai-guidance-update source=<approved-update|clarification-decision> mode=apply-approved-update` — a **filled clarifications ledger** is a valid apply `source` (its `decision:` fields ARE the approval)

**Constraints:** never apply governance-impacting updates without explicit approval; never auto-promote solution notes to guidance; keep changes minimal; touch one thing; preserve links to approved sources; flag conflicts rather than resolving them.

**Phases:**

1. **Discover** current guidance, the **clarifications ledger**, and the source learning. When `source=clarification-decision`, the filled ledger IS the input: read each entry's `decision:`.
2. **Classify the learning:** story-specific / impl detail / reusable convention / architecture rule / brownfield ambiguity / contract change / security|privacy|audit|compliance rule / reference impl / candidate memory / suspected drift / conflict.
3. **Decide the target** using the routing table in §9. (Rationale → SAD/ADR. Contract truth → spec. Conventions → Guidelines. Behavior-changing architecture → Architecture Rules. Current-vs-target → Guardrail. Story-specific → Jira. Unproven → solution note.)
4. **Conflict check** against requirements, Story Artifact, specs, ADRs, SAD, Architecture Rules, Guidelines, Guardrails, reference impl, current code, solution notes. If it conflicts, don't apply — produce a conflict finding.
5. **Analyze-only output:** a Guidance Update Analysis (change no files).
6. **Apply-approved-update:** verify explicit approval + target + text → apply the smallest change → don't touch unrelated sections → preserve/add source links → flag any conflict found → produce an Applied Update Report. If approval is missing, stop with an Approval Missing Report. **Working the ledger** (`source=clarification-decision`): for each filled entry — *accept/edit* → fold the (possibly edited) rule into the clean Architecture Rules/Guidelines with the smallest change and **drop it from Open**; *reject* → move it to **`## Settled — won't re-propose`**. A **code-vs-stale-source** decision resolved in code's favour writes the corrected rule into the clean file **and flags the upstream SAD/ADR/spec as stale** (it never rewrites the source). The ledger trends to empty.

**Analyze-only output:**

```markdown
# Guidance Update Analysis
## Decision  (No update needed | Candidate update | Human approval required
             | Conflict detected | Apply-ready, approval already explicit)
## Source                (type / reference / summary)
## Learning classification
## Recommended target
## Reason
## Existing guidance affected
## Conflict check        | Source | Conflict? | Finding |
## Human approval required  (Yes/No; owner: Architect/Tech Lead/PO/Security/Privacy/Compliance/QA)
## Suggested minimal update  (smallest possible text change)
## Not included          (related changes intentionally excluded)
## Recommended next action
```

**Applied-update output:**

```markdown
# Applied Guidance Update Report
## Decision   (Applied | Not applied | Partially applied | Blocked)
## Files changed         | File | Section | Change |
## Approval source
## Summary of applied change
## Conflicts detected during application
## Follow-up recommended
```

**Stop when:** approval is missing; the target is unclear; the update conflicts with specs/SAD/ADRs; it would require an architecture decision or security/privacy/audit/compliance approval; it would change contract truth; or it's broader than what was approved.

---

## 11. The optional reviewer agents

The reviewer files live in `.claude/agents/` (Claude build) or `.github/agents/` (Copilot); the table below summarizes them. They own each dimension's review logic; `ai-context-check` delegates to them. Run them as **parallel sub-agents** when reviews get broad; for a narrow change the check workflow applies their criteria **inline from these same files** (one source either way). Each takes inputs from the orchestrating workflow, identifies missing context, and asks **at most one** blocking question.

| Agent | Reviews | Don't use for |
|---|---|---|
| `architecture-boundary-reviewer` | Service/context/module boundaries, ownership, data ownership, sync-vs-async, API/event ownership, dependency direction, shared-SDK use, architecture-sensitive refactors | Style, test naming, formatting, low-risk local details, story/AC writing |
| `engineering-convention-reviewer` | Repo structure, placement, layering, naming, DTO/mapping/validation/error-handling, logging/observability, tests, shared utils, scope control, AI-generated code | Architecture/data-ownership/scope/security/compliance/contract approvals (escalate those) |
| `brownfield-governance-reviewer` | Current-vs-target gaps **and** source conflicts together; decides if a Guardrail / guidance update / ADR / spec update / human decision is needed | Normal aligned work, simple style, product priorities, isolated low-risk details |
| `contract-compliance-reviewer` *(add only for regulated/contract-heavy work)* | API/event/data/UI contract changes & backward compatibility; security, privacy, audit, compliance behavior; flags governance-significant changes lacking an approved source | Architecture/ownership decisions, code style, work touching no contract and no sensitive data |

**Shared output shape:** Decision · what was reviewed · findings table (`Area | Status | Finding | Evidence`) · the relevant impact (coupling / ownership / scope / brownfield interpretation) · one blocking question or "None." · one recommendation. Every agent must **cite the rule/source each finding violates and the offending location** (file:line or contract field) — no vague "seems off" — and **anchor on what already exists** (the approved pattern/module/contract) before flagging an invented parallel structure.

The brownfield agent also uses the **statuses** and **conflict types** below, and includes a draft Guardrail only when one is needed:

- *Statuses:* Use current · Use target · Target not ready · Ask first
- *Conflict types:* no conflict · terminology mismatch · stale AI guidance · stale SAD/ADR · formal-spec mismatch · self-contradiction within one source · implementation drift · brownfield ambiguity · coding-guideline overreach · solution-note overreach · missing architecture decision · missing contract update · governance approval required

It ends with a short **"Do not do"** list (e.g. *don't implement the proposed coupling; don't update guidance; don't treat current code as approved*) until the issue is resolved.

---

## 12. Adoption path

1. **Create the minimum files** — `ai-architecture-rules.md`, `ai-coding-guidelines.md`, a root instruction file. (Source map optional; the clarifications ledger is generated by bootstrap.)
2. **Add the three skills** under `.claude/skills/`.
3. **Add agents only if useful.** Skip for a pilot; add when reviews get broad.
4. **Bootstrap:** `/ai-context-bootstrap` (whole repo; add `scope=<area>` to pilot one area) — writes the clean files plus the ledger of open decisions.
5. **Work the ledger → ratify:** fill each `decision:` in `docs/architecture/ai-clarifications.md`, then `/ai-guidance-update source=clarification-decision mode=apply-approved-update` — accepted items fold into the clean files, rejected ones go to Settled.
6. **Use on stories:** `/ai-context-check work=<story-or-plan>`
7. **Analyze a candidate learning:** `/ai-guidance-update source=<finding-or-note> mode=analyze-only`
8. **Apply only approved updates:** `/ai-guidance-update source=<approved-update> mode=apply-approved-update`

---

## 13. In one paragraph

The SAD and ADRs stay the source of architecture knowledge. The **AI Architecture Rules** is a thin operational layer that tells AI how to apply that knowledge safely; the **AI Coding Guidelines** tell AI how to implement within it; **Brownfield Guardrails** stop current-but-wrong patterns from becoming future architecture. Three small skills — **bootstrap, check, update** — form a loop that discovers context from the repo, checks every story against it, and folds back only the learnings that change future behavior. Skills ask one blocking question at a time, and humans approve every governance decision. It stays lightweight on purpose: leverage over ceremony, and each story leaving the guidance a little better than it found it.

---

## Appendix — Scaling to many repos (a worked example)

*The toolkit is single-repo by default. Use this only when your architecture and coding rules are genuinely cross-repo (e.g. many microservice / microfrontend repos). It's the same mechanism, one level up — no special mode.*

**The idea:** write the cross-repo rules **once** in a central repo; every repo **links** to it and adds only its own specifics. Rules live in one place → no drift; change them once → every repo gets it on a version bump.

```
platform-ai-context  (central, owned by architects)
   ├─ docs/architecture/ai-architecture-rules.md  ← cross-repo rules
   ├─ docs/engineering/ai-coding-guidelines.md
   └─ guardrails/
        ▲  vendored & version-pinned
   svc-orders    svc-payments    mfe-checkout    … (×N)
```

### Set up the central repo (once)
Run `ai-context-bootstrap` in `platform-ai-context` to create the cross-repo Architecture Rules + Coding Guidelines + Guardrails, pointing at the org SAD/ADRs/shared specs. Evolve them later with `ai-guidance-update` — the cross-team governance seat (see below).

### Updating the system-wide context

The skills are **per-repo**: running them in a repo affects *only that repo's* context. There is **no special "global" mode and no automatic cross-repo promotion** — "central" is just a normal repo whose context many repos consume. So the central context is updated the same way any repo's is: by running the skills **in the central repo**, against its own artifacts.

1. **Trigger** — a change in the central repo's authoritative artifacts (a cross-repo ADR / spec / SAD change), **or** a learning a **human** judges to have org-wide merit and **raises in the central repo** (usually as a proposed ADR/spec). Carrying a local learning up is a human act; a local `ai-guidance-update` never reaches global on its own.
2. `ai-guidance-update mode=analyze-only` (in the central repo) classifies it, conflict-checks it against the full org rules, and proposes the smallest change — flagged **human approval required**.
3. An architect / platform / security owner approves.
4. `ai-guidance-update mode=apply-approved-update` applies the minimal edit to the central Architecture Rules / Guidelines / Guardrails (links preserved). It still **never writes the SAD/ADRs/specs** — those it flags or drafts for a human.
5. Commit (and, if repos pin, tag a release). Propagation to the repos follows your distribution choice below.

### Wire one repo to it (each team, a few minutes)

1. **Link the central layer** (auto-consume the latest, by default):
   ```bash
   git submodule add -b main git@github.com:org/platform-ai-context .ai/central
   # CI (or before a run) pulls the latest:  git submodule update --remote .ai/central
   ```
2. **Point the agent at it** — `AGENTS.md`:
   ```markdown
   ## Read order
   1. .ai/central/docs/architecture/ai-architecture-rules.md  # cross-repo (authoritative)
   2. .ai/central/docs/engineering/ai-coding-guidelines.md
   3. .ai/central/guardrails/
   4. docs/architecture/ai-architecture-rules.md              # this repo's specifics
   ## Authority: central wins for cross-repo concerns; local governs local-only; on conflict, raise it.
   ```
3. **Point the skills at it** — the source map (`ai-enablement/source-map.yaml`):
   ```yaml
   guidance:                                  # canonical keys; central paths first, then this repo's
     rules:      [.ai/central/docs/architecture/ai-architecture-rules.md, docs/architecture/ai-architecture-rules.md]
     guidelines: [.ai/central/docs/engineering/ai-coding-guidelines.md]
     clarifications: docs/architecture/ai-clarifications.md   # ledger stays local to this repo
   ```
4. **Write local specifics:** `/ai-context-bootstrap` — it reads central first and writes only this repo's local Architecture Rules (cross-repo concerns just link back to central).
5. **Use it:** `/ai-context-check work=<PR>` — enforces central + local together.

### Staying current — auto-consume by default; pin only when it gates

By default the link above **tracks latest**, so central rule changes are picked up automatically (next CI run / `git submodule update --remote`). That's the right choice when the context is **advisory** — the agent reads it, nothing hard-blocks on it.

**Pin a version instead** only when `ai-context-check` gates CI or reviews must be reproducible — so an edit in the central repo can't change whether your *unchanged* code passes:
```bash
git -C .ai/central checkout v1.5.0   # pin; adopt later versions via a reviewable bump PR
```
Hybrid: auto-consume the bulk, and have the central repo declare a **minimum version** that CI enforces — so urgent rules (e.g. a security fix) still land immediately.

**Toolkit vs. your plumbing:** the toolkit gives the model, the skills, and the linking; you supply ordinary dependency distribution (submodule or package) — and, *if you pin*, a fleet-bump bot — the same rails you already use for shared libraries.
