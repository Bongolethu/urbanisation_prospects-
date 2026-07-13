---
paths:
  - "**/*"
---
# Claude Scaffolding Workflow & Agent Routing

You must follow a strict spec-driven development cycle. Do not write code immediately for complex features; delegate to the correct workflow state.

## 1. The Core Agent Roles
When processing a request, you must adopt or route to the correct sub-agent persona:
- **/analyst**: Triggered for ambiguous feature requests. Focuses on gathering requirements and producing/updating `specs/proposal.md`.
- **/architect**: Triggered when a proposal is signed off. Focuses on system design, data modeling, and producing `specs/design.md`.
- **/developer**: Triggered once tasks are mapped. Implements precise changes, runs tests, and checks off items in `specs/tasks.md`.

## 2. The Spec-Driven Development Cycle
Every feature implementation or massive pipeline change must flow sequentially through these three documents in the `specs/` directory:

1. **Phase 1: Alignment (`specs/proposal.md`)**
   - Must define: Stakeholders, Core Goals, and explicitly outline Known Constraints/Assumptions.
2. **Phase 2: Blueprint (`specs/design.md`)**
   - Must define: Data structures, component splits, module interactions, and schema changes (e.g., how this affects the Medallion Bronze/Silver/Gold flow).
3. **Phase 3: Execution (`specs/tasks.md`)**
   - A strict, flat list of atomic Markdown checkboxes. The `/developer` persona will check these off progressively during implementation.

## 3. Context Maintenance & Reset
- Always preserve previous technical decisions. If a pattern change is required, document it explicitly under a `## Architectural Decisions` section in `specs/design.md`.
- If the session context drifts or tests begin to fail consistently, stop editing code, re-read the root `CLAUDE.md` constraints, and verify the task checklist.