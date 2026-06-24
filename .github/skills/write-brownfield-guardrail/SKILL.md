---
name: write-brownfield-guardrail
description: >-
  Write a Brownfield Guardrail when current code and target direction differ in a way that
  could mislead the AI — capturing what to use, what not to copy, and when to ask. Use when
  creating, updating, or applying one (in bootstrap, guidance updates, or brownfield review).
  A Guardrail records a divergence; it never originates an architecture decision.
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
that doesn't exist yet, mark `Ask first` (with an owner) and recommend formalizing it — never
decide it silently.
