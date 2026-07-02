---
name: write-brownfield-guardrail
description: >-
  Write a Brownfield Guardrail when current code and target direction differ in a way that
  could mislead the AI — capturing what to use, what not to copy, and when to ask. Use when
  creating, updating, or applying one (in bootstrap, guidance updates, or brownfield review).
  A Guardrail records a divergence; it never originates an architecture decision.
---

## Inputs

- **trigger** — a misleading divergence between current code and target direction that could
  lead the AI astray. Absent a divergence, there is nothing to write.
- **template fields** — the Guardrail fields the writer fills: Topic, Status, Source, Current
  state, Target direction, Rule for new work, Rule for existing code, Do not copy, Ask when.

## Procedure

1. **Gate.** If current code and target direction are aligned, stop and emit nothing.
   Otherwise continue.
2. **Select exactly one Status** from the four reachable options:
   - current is approved → `Use current`
   - target is adopted (new work follows it even if code differs) → `Use target`
   - target exists but is out of scope (don't move there unless scoped) → `Target not ready`
   - no decision exists yet → `Ask first`
3. **If the Guardrail requires a decision that doesn't exist yet**, set Status `Ask first`,
   fill `Ask when` with the deciding owner, and recommend formalizing the decision. Do not
   decide it within the Guardrail — a Guardrail records a divergence; it never originates an
   architecture decision.

## Output

Return the filled Guardrail template — the single artifact this skill produces:

```markdown
## Brownfield Guardrail: <Topic>
Status: <Use current | Use target | Target not ready | Ask first>
Source: <SAD / ADR / spec / decision>
Current state: <what exists today>
Target direction: <what new work should use, if known>
Rule for new work: <what to do for new work>
Rule for existing code: <what to keep or must not change>
Do not copy: <misleading legacy pattern>
Ask when: <conditions needing clarification>
```
