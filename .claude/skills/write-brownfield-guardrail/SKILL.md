---
name: write-brownfield-guardrail
description: >-
  Writes a Brownfield Guardrail for a case where current code and target direction differ enough to
  mislead the AI: what to use, what not to copy, and when to ask instead. Records a divergence
  someone else decided; never originates an architecture decision, and emits nothing where current
  and target are aligned.
---

Take a **trigger** — a divergence between current code and target direction that could mislead the
AI — and write a Guardrail for it. If current code and target direction are aligned, stop and emit
nothing.

**Statuses:** `Use current` (current is approved) · `Use target` (new work follows target
even if code differs) · `Target not ready` (target exists; don't move there unless scoped)
· `Ask first` (don't decide alone — needs human clarification).

**Template:**

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

If the Guardrail needs a decision that doesn't exist yet, set Status `Ask first`, fill `Ask when` with
the deciding owner, and recommend formalizing the decision. Do not decide it within the Guardrail.
