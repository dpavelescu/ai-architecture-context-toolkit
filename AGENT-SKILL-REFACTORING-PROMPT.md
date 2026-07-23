# Prompt — refactor agents and skills for instruction quality

Paste this as the task in any repo that holds agent or skill definitions
(`*.agent.md`, `SKILL.md`, sub-agent files, system prompts). Name the files to audit.

---

## What you are fixing

Agent and skill files drift into **documentation**. They accumulate prose that explains, justifies,
or narrates instead of instructing — and the executing model then has to infer what to *do* from text
written for a human reader.

**The test for every line: does it change what the agent does?** If not, cut it or convert it to a
directive. Apply the rules below, then the *Do not remove* list, then verify.

---

## Rules

### 1. Every line instructs

Cut narration about the document's own structure, motivation, justification, and commentary about the
file's own configuration.

- ✗ "These are what the later steps draw on." · "Nothing is written yet." · "This split is the run."
- ✗ "everything downstream is only as good as this" · "so a reader sees what changed without opening the files"
- ✗ "(subagents in the IDE via `agents:` — ensure it's enabled)" ← commentary about its own frontmatter
- ✓ State the action. Ordering and purpose come from the structure, not from prose about the structure.

### 2. No defensive negatives

Do not deny what a step manifestly isn't doing.

- ✗ "Write no files here." on a step whose job is emitting a report
- ✓ **Keep** a negative when it forbids a *plausible wrong action no step enumerates*: "Read-only —
  never edit X (only Y writes it)", "never write into a SAD/ADR/spec; flag or draft for a human".

The difference: a fence stops a reader from crossing something they'd otherwise cross. Noise denies
something nobody would do.

### 3. No leakage between caller and callee

A caller must not restate a called skill's procedure, output shape, or rules. Legitimate caller
content is only: **the call, the inputs that represent decisions, and the routing of what returns.**

- ✗ "source map first, search fallback" ← the skill's procedure
- ✗ "returns them in authority order" ← the skill's output shape
- ✗ "In the Context, add a Guardrail…" ← *where* an artifact lands is the writer's job
- ✗ Re-listing the callee's status enum, gate, or template

Symmetrically: a skill must not narrate its callers or who owns what.

### 4. No reader cross-references

- ✗ "(see **Output**)" · "(below)" · "(see §4)"
- ✓ "emit the Result **in the Output format**" · "in the shape the Output gives"

### 5. No positional coupling

Never navigate by step number — it breaks silently on renumber and forces the reader to jump.

- ✗ Forward: "steps 5–6 write it" · "stays a candidate for step 4"
- ✗ Provenance: "`sources` = step 1's `sources`" · "the final rules from steps 2–3"
- ✓ Refer to a value **by what it is**: "the final rules", "the still-undecided candidates",
  "the resolved `ledger`".

### 6. Pass decisions, not plumbing

At a call site, express only what the caller *decides*. Let the callee's declared Inputs own the rest.

- **Drop** echoes where the value name already matches the input name (`sources` = the sources).
- **Keep** filters ("the *still-undecided* candidates"), conditionals ("merging into the existing file
  *on a refresh*"), name-mismatches ("from the **final rules**" → the callee's `coverage-decisions`),
  and required mode choices (`resolve-decisions`).

If discovery loads material once into shared context, later steps should not re-declare where it came
from.

### 7. Constraints must fence what no step enumerates

A Constraint earns its place only if it governs something the Process doesn't already say. If one step
owns it, move it there and delete the constraint.

### 8. Descriptions are a selection surface, not documentation

A description states **what it does, what it produces, and its own boundary — as fact**.

- ✗ Transplanted docs: "Use when: … Not for: …"
- ✗ Advocacy: "so useful learnings don't die in chat"
- ✗ Narrating siblings: "the siblings own the rest of the loop: X reviews…, Y changes it"
- ✓ "Generates A and B from C… Creates the guidance; doesn't review or change it."

Each sibling advertises itself; state only your own limit.

### 8a. Persona and identity: place by consumer, don't duplicate

The `description` and the body have **two different consumers**, so identity splits between them rather
than living in both.

| Surface | Consumer | Carries |
|---|---|---|
| frontmatter `description` | the **selector** (router deciding *when to invoke*) | identity as fact — what it does, what it produces, its boundary |
| body (`Constraints` / `Process`) | the **executor** (the running agent) | the stance *enacted* — as constraints and steps |

- ✗ Restating identity in both places → recurring token cost in every run, and two copies that drift.
- ✗ Persona only in the body → the selector can't see it → mis-selection.
- ✗ Behavior only in `description` → the executor may not receive it as an operating instruction.

Rule of thumb: `description` = the minimum a router needs to pick correctly; body = the minimum an
executor needs to behave correctly; their overlap ≈ zero.

**Two kinds of opener — one is noise, one earns its place.** Don't collapse them.

- **Identity restatement** ("You are a system-architecture reviewer. Your job is to review boundaries")
  → *always delete*. It repeats the description, instructs nothing, dilutes the executor's attention.
  The body already *is* a boundary reviewer by virtue of its constraints and steps.
- **A dispositional opener** ("distrust the change that mirrors existing code — similarity is a reason
  to check, not proof it's approved") → *keep, when it earns it*. This is not a restatement; it primes
  a stance that shapes how the whole review is carried out.

Role priming is real: a *specific* disposition activates a prior that **generalizes to cases the rules
never enumerated** — which a constraint list, by construction, cannot. But it earns its one line only
when **both**:
  1. **the task is open-ended** — an unbounded judgment/detection space (a reviewer hunting "locally
     reasonable but wrong"), not a closed pipeline whose steps already cover the space; **and**
  2. **the stance is specific** — "assume this change is breaking until a source proves otherwise," not
     the generic "you are a careful reviewer" (near-noise on a capable model).

Two failure modes to respect: a **closed/deterministic** agent (discover → assess → write) gains
nothing — there are no unlisted cases to generalize to, so any opener is restatement. And a disposition
**over-fires** (manufactures findings to fulfill the persona) unless a precision bound pairs with it —
so the disposition primes *recall*, and a `cite-or-don't-raise` / `right-size` constraint protects
*precision*. Ship them together.

### 9. Don't invent structure an input already carries

If a config/manifest already types or groups its entries, consume that structure — don't re-classify
into a parallel taxonomy of your own.

### 10. One word, one meaning

Pick a verb per concept and hold it. Example from a real sweep: "draft" meant *propose text for a human
to own* in one place and *write the file* in another, two lines apart. Also: no set notation (`∈`), one
canonical product name, one canonical path.

---

## Structure: phases with gates

For any multi-step agent, use heading-delimited phases, not a numbered list — once a phase has a body,
list-continuation indentation is fragile and phase boundaries blur.

```markdown
## Process

### Phase N — <Name>

**Goal** — <the target state, stated independent of method, so the agent can replan when the
method hits something unexpected.>

**Procedure**
- <one discrete action per bullet, in order>
- <conditionals are bullets too: "If X → do Y">
- <an action with sub-outcomes>:
  - <nested outcome>

**Complete when** <the *verifiable* invariant that must hold — name the thing that can silently
break, not a restatement of the action.>
```

Rules for this shape:

- **Goal ≠ Complete when.** Goal is the outcome; Complete when is the check *including the invariant
  that can silently break*. If they say the same thing, one is noise. (Goal: "the ledger carries every
  open decision this run surfaced." Complete when: "…**with the prior `Settled` list preserved**.")
- **No `Produces:` label.** It re-threads provenance between phases; the consumer already names what it
  needs.
- **No generic gate line** ("each phase runs until its Complete when holds") — that describes the
  notation. State ordering where it's dangerous, in domain terms: "never start writing while a live
  question is outstanding."
- **Don't force phases onto atomic steps.** A reviewer whose steps are single checks should stay a
  numbered list; wrapping seven one-liners in Goal/Procedure/Complete-when is ceremony.

**Why gates matter:** the executor is non-deterministic and may treat a step as a checklist tick —
"I searched once, therefore Discover is done." A completion condition converts *do X* into *do X until
Y holds*.

**Side benefit worth expecting:** writing a completion condition forces you to name what must be true,
which surfaces directives that were quietly missing. In one sweep, a gate referenced a state
("recorded `unclear`") that the body never instructed the agent to produce.

---

## When phases should merge (and when not)

Judge a merge on three axes, not on symmetry with sibling agents:

1. **Distinct outcome?** Two outcomes → two phases.
2. **Same mode of work?** Retrieval / judgment / human interaction / writing / reporting. Different
   modes → don't merge.
3. **Gate strength.** Would one compound gate be a weaker check? A merged gate can be satisfied while
   silently failing one original invariant — e.g. "candidates resolved AND ledger written" is
   order-blind: it passes if the ledger was written *before* asking.

Also check **conditionality**: if one phase is unconditional and its neighbour is parameterised,
merging invites the parameter to suppress both.

Do not normalise phase counts across agents. An agent that does more earns more phases.

---

## Interaction design (agents that ask a human)

- If the analysis already derived a **proposal and rationale**, the question must carry them. Don't
  pose a bare question when the agent has already done the reasoning.
- Offer suggestions **as suggestions, never as a closed set** — the human may take one, edit it, answer
  in their own terms, or defer. If the AI frames the options, a human picking from them is ratifying
  the framing rather than deciding.
- Surface **standing**: if a proposal is derived from low-authority evidence, say so, so a
  recommendation doesn't become an anchor at the moment of ratification.
- Define the **no-one-to-answer** branch explicitly. Headless/CI runs must neither block nor guess.

---

## Contract hygiene

- Every declared Input is consumed by some step.
- Every advertised Output value — **every enum member** — is reachable from some step.
- Every call passes exactly the callee's declared Inputs; every referenced skill/agent exists.
- Each failure path says what to do next (remediate or escalate), not just that it failed.
- If the repo ships the same capability in two packagings, they must match on **behavior, inputs,
  output shape and enums** — house style may differ.

---

## Do not remove (deleting these is a regression)

- Governance fences that forbid a plausible wrong action: read-only rules, "propose; a human
  approves", "X is evidence, not authority", "never write into <human-owned artifact>".
- Genuine branch outcomes: the no-evidence stop, the non-interactive branch, refresh/baseline handling.
- Give-up and escalation paths, and the single-blocking-question rule.
- Output templates, enums, decision lists, routing tables, concern checklists.
- Right-sizing rules ("a small low-risk change gets a one-line answer").

---

## Verify

1. Re-read each file end to end: does it read as an executable process?
2. Every declared input consumed; every enum value reachable.
3. No cross-references to step numbers; no `(see …)`.
4. Every referenced skill/agent resolves on disk.
5. Where two builds exist, diff their phase names, inputs, and output blocks.
6. Report what you removed **and** anything borderline you kept, with the reason.
