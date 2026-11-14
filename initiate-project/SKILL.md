---
name: initiate-project
description: >-
  Collaborative 0-to-1 Project Initiation workflow (researcher + planner + hr-scouter).
  Combines ecosystem compatibility research, interactive blueprint creation, and
  team/effort sizing before coding begins.
---

# 0-to-1 Initiation Protocol: `researcher` + `planner` + `hr-scouter`

This skill coordinates the 3 early-phase agents so new projects (or major version upgrades) start with verified technical facts, unambiguous requirements, and an efficient parallel team formation.

---

## 3-Phase Funnel (With Parallel Start Option)

```text
   ┌────────────────────────────────────────────────────────────┐
   │ 1. RESEARCHER (Ecosystem & Compatibility Audit) [parallel] │
   │    + PLANNER (Interactive Blueprint Q&A)                   │
   │    + DATA-ARCHITECT (optional, if DB schema is needed)     │
   └─────────────────────────────┬──────────────────────────────┘
                                 │ docs/project-brief.md (+ draft DB entities)
                                 ▼
   ┌────────────────────────────────────────────────────────────┐
   │ 2. HR-SCOUTER (Fixed medium: Team Formation, Parallel Plan,│
   │    1-by-1 Commands & low/medium/high Presets)               │
   └─────────────────────────────┬──────────────────────────────┘
                                 │ Ready-to-run commands
                                 ▼
                    [READY FOR PARALLEL EXECUTION]
```

---

### Phase 1A: `researcher` — Technical Fact-Finding *(Can run in parallel with initial `planner` Q&A)*
1. **Targets**:
   - Check execution environment, target runtime/language/SDK/framework versions, and existing project configs.
   - Inspect external dependencies, library compatibility, and potential breaking changes.
   - Survey proven architectural patterns across Web, API, CLI, library, desktop, or mobile.
2. **Role Boundary**: Does not decide MVP product features (`planner`) and does not write production code.
3. **Output**: Concise technical brief covering:
   - *Active & Target Versions*
   - *Technical Constraints & Compatibility Guard*
   - *Recommended Patterns & Trade-offs*

---

### Phase 1B: `planner` *(+ `data-architect` if DB-backed)* — Blueprint & Ambiguity Removal
1. **Discussion Targets**:
   - Incorporate `researcher` facts and present clear options to the User.
   - Ask targeted questions to lock MVP vs later-phase scope.
   - If database persistence is needed, consult **`data-architect`** to validate core entities and draft `docs/db-schema.md`.
2. **Role Boundary**: `planner` does not design code architecture (`architect`), does not size agent efforts (`hr-scouter`), and delegates column/index details to `data-architect`.
3. **Output**: `docs/project-brief.md` (and draft `docs/db-schema.md` if applicable) with Vision, Rough Tech Stack, Approved Decisions, and MVP vs Future Scope.

---

### Phase 2: `hr-scouter` *(Fixed `medium` effort)* — Team Sizing, 1-by-1 Commands & Parallel Plan
1. **Assessment Targets**:
   - Scan `docs/project-brief.md` and repository readiness.
   - Assign strict non-overlapping `owns` boundaries, group independent agents into parallel stages (`P1`, `P2`, ...), and merge minor roles where appropriate.
   - Output **1-by-1 `/summon-agent` commands** for each agent plus **3 Effort Presets**:
     - **Low**: Minimalist, fast (single MVP focus).
     - **Medium (Recommended)**: Balanced, with reviewer and test verification.
     - **High**: Maximum platform capability, parallel QA/testing, and full release docs.
2. **Role Boundary**: Read-Only; never edits files or runs implementation.
3. **Output**: Team formation table, 1-by-1 commands, and parallel batch command ready for execution.

---

## Quick Invocation

- Run via slash command: `/initiate-project <project_idea>`
- Or invoke directly:
  ```text
  /summon-agent [researcher (medium) || planner (medium)] -> hr-scouter (medium)
  ```
