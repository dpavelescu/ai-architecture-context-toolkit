---
name: brownfield-guardrail
description: >-
  The Brownfield Guardrail format and rules — for when current code and target direction
  differ in a way that could mislead the AI. Use when creating, updating, or applying a
  Guardrail (during bootstrap, guidance updates, or brownfield review).
---

Create a Guardrail **only** when current code and target direction differ in a way that
could mislead the AI. Don't create one for aligned situations.

**Statuses:** `Use current` (current is approved) · `Use target` (new work follows target
even if code differs) · `Target not ready` (target exists; don't move there unless scoped)
· `Ask first` (don't decide alone — needs human clarification).

**Template:**

```markdown
## Brownfield Guardrail: <Topic>
Status: <Use current | Use target | Target not ready | Ask first>
Source:            <SAD / ADR / spec / decision>
Current state:     <what exists today>
Target direction:  <what new work should use, if known>
Rule for new work:     <what to do for new work>
Rule for existing code:<what to keep or must not change>
Do not copy:       <misleading legacy pattern>
Ask when:          <conditions needing clarification>
```

A Guardrail is operational guidance, not an architecture decision: if it needs a decision
that doesn't exist yet, mark `Ask first` (or `pending ADR` with an owner) — never decide it
silently.
