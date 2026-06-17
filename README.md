# AI Architecture Context & Coding Guidelines

**Make AI coding agents respect your approved architecture and decisions — especially in messy, real-world (brownfield) projects.**

This repo gives you two thin files, three skills, and four optional reviewer agents that tell an AI agent *how to apply your approved architecture* before it plans or writes code. No new architecture document. No heavy process. It's designed to drop into an existing project.

---

## The problem

You point an AI agent at your project and ask it to build something. To act, it reads whatever it can find — the SAD, ADRs, specs, and code — and **fills the gaps with assumptions.** When the guidance is thin, it goes wrong in three quiet ways:

- **Misinterpretation** — it draws the wrong operational conclusion from an artifact (treats a draft ADR as decided, a diagram as binding, or a stale doc as current).
- **Silent conflict resolution** — two sources disagree (code vs the SAD, an ADR vs a spec) and it just picks one — usually whatever the code already does — without flagging it.
- **Silent gap-filling** — a critical constraint isn't written down anywhere, so it invents an answer instead of asking.

The sharpest case is **brownfield code**: the agent finds a pattern and copies it confidently — but it might be approved ✅, tolerated legacy ⚠️, a half-finished migration 🚧, or a deadline shortcut 🩹, and code looks like code.

> "Other services call each other directly over REST, so I added another REST call."
> *(…but your target is async events — it just expanded the coupling you're removing, and it passed tests.)*

A senior engineer would have known — which source wins, what's stale, what's missing, whom to ask. The AI doesn't, because nobody told it. And an architect can't review every story across every team to catch it.

**The fix:** write down — once, thinly — what the AI needs before it acts: **what has authority, what conflicts to surface, what's missing (ask, don't guess), and what not to copy.**

---

## The solution in one picture

```
   Your existing docs            Two thin AI-facing files            The AI agent
   (unchanged)                   (this repo helps you create)

   ┌──────────────┐              ┌─────────────────────────┐
   │ SAD / ADRs   │  ───links──▶ │ AI Architecture Context │ ──┐
   │ Formal specs │              │  (what has authority,   │   │   reads these
   │ Diagrams     │              │   what not to copy,     │   │   BEFORE it
   └──────────────┘              │   when to ask)          │   ├──▶ plans or
                                 ├─────────────────────────┤   │    writes code
                                 │ AI Coding Guidelines    │ ──┘
                                 │  (how to implement it)  │
                                 └─────────────────────────┘
```

The two files are **thin**: they don't repeat your architecture, they *point* to it and add only the rules that change what the AI does. If your SAD changes but the AI's behavior shouldn't, the files don't change.

---

## How it works: one small loop

| Step | Skill | When | What it does |
|---|---|---|---|
| **1. Bootstrap** | `ai-context-bootstrap` | Once per repo | Reads your repo and drafts the two thin files (+ a manifest and Guardrails if needed) |
| **2. Check** | `ai-context-check` | Every story / plan / PR | Checks the proposed work against the files — catches "locally reasonable but architecturally wrong" before it ships |
| **3. Update** | `ai-guidance-update` | Only when a learning should become a rule | Folds an approved learning into the files, with human approval |

Day to day it's **Check ↔ Update**: bootstrap once, then check each story, and occasionally capture a learning. Each story leaves the guidance a little better than it found it.

Optional reviewer **agents** (architecture, engineering, brownfield — plus a contract & compliance reviewer for regulated/contract-heavy work) plug into the Check/Update steps *only if* reviews get broad. Skip them for a pilot.

---

## What's in this repo

Two idiomatic builds — same behavior, packaged for each tool. Use the one for your agent.

```
README.md                                  ← you are here (the why + onboarding)
AI-Architecture-Context-and-               ← the full playbook (all the detail)
  Coding-Guidelines-Playbook.md

.claude/                                   ← Claude Code build
  skills/    ai-context-bootstrap | ai-context-check | ai-guidance-update   (orchestrators = skills)
  agents/    architecture-boundary | engineering-convention | brownfield-governance | contract-compliance   (reviewers = sub-agents)

.github/                                   ← GitHub Copilot build (drop in the agents + skills folders)
  agents/    ai-context-bootstrap | ai-context-check | ai-guidance-update   (orchestrators = custom agents)
             + the 4 reviewers             (reviewers = custom agents, delegated by check via agents:)
  skills/    coverage-sweep | brownfield-guardrail   (shared capabilities, auto-loaded)
```
> No global `copilot-instructions.md` is shipped — the short behavioral house rules are inlined in each agent, and the authority/read order lives in the generated Context — so dropping these folders into a project won't touch its own instructions. Agents omit `tools:` (they use your configured Copilot tools); reviewers are read-only by instruction.

> Same responsibilities and boundaries in both; only the *packaging* differs (Claude Code: skills + sub-agents; Copilot: custom agents + Agent Skills). Pick one build — you don't need both.

The two files the skills *produce* in your project end up at:

```
docs/architecture/ai-context.md            ← AI Architecture Context
docs/engineering/ai-coding-guidelines.md   ← AI Coding Guidelines
ai-enablement/context-manifest.yaml        ← optional map of your sources
AGENTS.md / CLAUDE.md / copilot-instructions.md  ← tells the agent to read the above
```

---

## Onboarding into a brownfield project

> **Goal:** an AI agent that, before touching your repo, knows your boundaries, knows what *not* to copy, and asks when unsure.

### Step 1 — Copy the toolkit in

Copy the build for your tool into your project's root:
- **Claude Code:** the `.claude/` folder (skills + sub-agents, discovered natively).
- **GitHub Copilot:** the `.github/agents/` and `.github/skills/` folders (custom agents + Agent Skills).

### Step 2 — Run bootstrap

```
/ai-context-bootstrap mode=interactive            # whole repo
# or focus one area first:
/ai-context-bootstrap scope=services/order-service mode=interactive
```

It will:
- discover your SAD, ADRs, specs, and representative code automatically (you won't paste documents)
- draft `docs/architecture/ai-context.md` and `docs/engineering/ai-coding-guidelines.md`
- ask you **one question at a time**, only when a real gap blocks it (e.g. *"Which is the authority for cross-service comms — SAD §4.3, ADR-012, or current code?"*)
- flag anything it can't safely decide as `TBD` or "ask first" instead of guessing

Start small: add `scope=<area>` (a path, paths/glob, or a manifest `areas:` name) to focus one service or area if the whole repo is too big.

### Step 3 — Review and approve the drafts

Read the two generated files. Fix the `TBD`s. This is where you encode the few things that matter — *"new cross-service comms use events, not REST," "no service reads another's DB," "don't copy the old validation helper."* Keep it thin.

### Step 4 — Add a root instruction file

Add `AGENTS.md` (or `CLAUDE.md`, or `.github/copilot-instructions.md`) that tells the agent to read the context files first. A template is in the [full playbook](AI-Architecture-Context-and-Coding-Guidelines-Playbook.md#5-the-minimum-set).

### Step 5 — Use it on real work

Before building a story, check the plan against your guidance:

```
/ai-context-check work=<story-or-plan> mode=analyze-only
```

You get an alignment report: what's fine, what's risky, and any "this copies legacy you're moving away from" warnings.

### Step 6 — Capture lessons worth keeping (later, not now)

When you notice the AI making the *same* mistake across stories, fold the lesson in:

```
/ai-guidance-update source=<the-finding> mode=analyze-only     # propose
/ai-guidance-update source=<approved-update> mode=apply-approved-update   # apply, once approved
```

That's the whole loop. **You don't need steps 6 or the agents to start** — bootstrap + check is enough for a pilot.

---

## The rules that keep it safe (and small)

- **Existing code is evidence, not authority.** A pattern existing in the repo doesn't make it approved for new work.
- **The AI never decides governance silently.** Architecture, security, privacy, audit, compliance, and contract changes need a human. The AI proposes; you approve.
- **One blocking question at a time.** No 20-question intake forms.
- **Right-size everything.** Small, low-risk work gets a compact pass. Full ceremony is reserved for large or risky changes. The point is *leverage, not paperwork.*
- **Humans still review.** This makes what reaches review safer; it doesn't replace review.

---

## When *not* to use this

- Greenfield project with a tiny team and one clear style → probably overkill; a short `CLAUDE.md` may be enough.
- You only want code formatting/linting rules → use a linter, not this.

This earns its keep when an AI must apply **approved architecture and decisions across artifacts that can mislead it** — stale docs, conflicting sources, or brownfield code — i.e. most brownfield systems.

---

## Learn more

The full reference — principles, authority order, "what goes where," Brownfield Guardrails, and the complete skill/agent specs — is in **[the playbook](AI-Architecture-Context-and-Coding-Guidelines-Playbook.md)**.

> **In one line:** your SAD and ADRs stay the source of truth; these thin files make that truth *usable by AI* — so the agent applies your architecture instead of copying whatever it finds.
