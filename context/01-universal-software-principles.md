Timeless architecture and design heuristics. Use as a foundation for reasoning, reviews, and architectural ADRs.

---

## Core Architectural Thinking
- **Everything is a trade-off.** No “best practice” exists without context. Ask: *it depends on what?*
- **Make decisions reversible early.** Favour options that can be undone cheaply.
- **Coupling & Cohesion:** Minimize dependencies between modules, maximize functional focus inside modules.
- **Quality Attributes drive design:** Performance, Availability, Modifiability, Security, Usability, Testability.
- **Document intent, not just structure.** Use ADRs for every important or irreversible choice.
- **Architecture is code.** Validate design through implementation and metrics.
- **Feedback loops > prediction.** Build-measure-learn, not guess upfront.

---

## Design & Evolution Principles
- **YAGNI:** Implement only what’s needed now, but design seams for future change.
- **KISS:** Prefer simplicity over cleverness. Complexity multiplies cost.
- **SRP:** One reason to change = one component.
- **Encapsulation over inheritance.** Hide details; expose contracts.
- **Composition over inheritance.** Assemble behaviors dynamically.
- **Fail fast, recover gracefully.** Resilience is an architectural responsibility.
- **Observability = design requirement.** You can’t operate what you can’t see.

---

## Reasoning Framework
| Lens | Question |
|------|-----------|
| **Domain** | What problem are we solving? |
| **Quality Attributes** | Which non-functionals dominate? |
| **Constraints** | What can’t we change? |
| **Context** | Who are the users and systems around us? |
| **Trade-offs** | What do we gain vs lose? |
| **Feedback** | How will we measure success? |

---

## Collaboration
- Architecture is **a conversation**, not a diagram.
- Communicate in layers (C4 model) + prose (ARC42).
- Create **shared language**: glossary, ubiquitous domain terms.
- Continuous alignment beats documentation drift.

---

## Timeless Quotes
> “The heart of software is its ability to solve domain-related problems for its user.” ― Eric Evans