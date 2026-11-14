---
name: nsquad
description: >-
  Modular 14-sub-agent architecture (researcher, planner, hr-scouter, orchestrator,
  architect, data-architect, reviewer, backend, ui-builder, debugger, refactorer,
  test-writer, a11y-style-checker, docs-release) with flexible effort selection
  (low/medium/max), contract readiness gates, parallel execution, and 0-to-1 initiation.
---

# Modular Sub-Agent Architecture (`AGENTS.md`)

> **Version:** 2.0.0 (Full English, 1-by-1 Specific Invocation & Expanded Parallelism) | **Style:** Clear, Structured, Zero Ambiguity

This document defines the architecture and collaboration protocol for multi-agent software development. It is **elastic**: you can run the full squad (14 agents), a lean subset (2–5 agents via `/panggil-agent`), or a single agent solo depending on task scale.

---

## 1. Model Mapping, Tool Parameters & Flexible Effort

When spawning sub-agents via `invoke_subagent` or `swarm`, map each agent tier and effort (`low`, `medium`, `max`) to Antigravity tool parameters:

| Design Tier | Agents | `Model` Tool Parameter | Effort Policy (`low` / `medium` / `max`) |
| :--- | :--- | :--- | :--- |
| **Argon** (`argon`) | **Core:** `architect`, `reviewer`<br>**Optional (large projects):** `planner`, `data-architect` | `inherit` *(if main session uses Argon)* or `pro` | **Flexible (`low` / `medium` / `max`)** per user preference or `hr-scouter` recommendation.<br>- For small-to-medium tasks, run `/planner` directly in the main session (no extra sub-agent) and merge `data-architect` into `architect`/`backend`.<br>- `low`: small review/design (1–2 files, straightforward flow).<br>- `medium`: standard architecture, DB schema, or medium PR review.<br>- `max`: complex cross-module architecture, large 0-to-1 design, or critical PR audit. |
| **Hydrogen** (`hydrogen`) | **Research & Coordination:** `researcher`, `hr-scouter`, `orchestrator`<br>**Core Workers:** `backend`, `ui-builder`, `debugger`, `refactorer`, `test-writer`<br>**Checklist & Docs:** `a11y-style-checker`, `docs-release` | `flash` *(research, coordination, & core workers)* / `flash_lite`–`flash` *(`a11y`, `docs`)* | **`hr-scouter` is fixed at `medium`.** All other agents are **flexible (`low` / `medium` / `max`)** per user preference or `hr-scouter` recommendation.<br>- Antigravity's `invoke_subagent` enum accepts `inherit`, `flash_lite`, `flash`, `pro`, so Hydrogen maps to `flash` (research, coordination, coding/debugging/testing) and `flash_lite` (`low` checklists/docs).<br>- `low`: checklist/template tasks (`a11y`, `docs` -> `flash_lite`), simple routing, or small edits.<br>- `medium`: SDK research (`researcher`), team sizing (`hr-scouter`, **fixed**), orchestration (`orchestrator`), or scoped implementation (`flash`).<br>- `max`: maximum reasoning available on the platform for complex modules from scratch, state/API integration, or hard root-cause debugging (`flash`). |

> **`define_subagent` Implementation Note:**
> - **Read-Only** agents (`researcher`, `hr-scouter`, `reviewer`, `a11y-style-checker`): set `EnableWriteTools: false`.
> - **Planning, Coordination & Writer** agents (`planner`, `orchestrator`, `architect`, `data-architect`, `backend`, `ui-builder`, `debugger`, `refactorer`, `test-writer`, `docs-release`): set `EnableWriteTools: true`.

---

## 2. General Rules (Apply to All Agents)

1. **Strict Ownership (`owns`)**: Work only on your assigned files. Never edit files outside your `owns` list unless explicitly delegated by `orchestrator`.
2. **Shared Files Rule**:
   - Shared files such as dependency manifests (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc.), `.env.example`, build configs, and shared type/contract definitions (`src/types/`, `shared/`, `contracts/`) **must never be edited concurrently**.
   - Changes to shared files must happen upfront by `architect` / `backend` (contract stage) or sequentially via `orchestrator`.
3. **Zero Assumptions**: If requirements or context are unclear, **ask `orchestrator` or the User** (`planner` uses interactive Q&A). Never guess.
4. **Context Hygiene (*Anti-Context Rot*)**: Read only files relevant to your task and `docs/`. Do not read the entire repository unnecessarily.
5. **Minimalism (YAGNI)**: Choose the simplest solution that solves the problem.
6. **Security**: Never hardcode secrets, API keys, or credentials; always use environment variables (`.env`).
7. **Self-Verification (*Pre-Handoff Check*)**: Every code-writing agent (`backend`, `ui-builder`, `debugger`, `refactorer`) **must** run a fast linter / typecheck / build check before handoff so `reviewer` cycles are not wasted on syntax/type errors.
8. **No Destructive Actions**: Never run `git push`, `merge`, deploy, or delete critical files/data without explicit **User** approval.
9. **Mandatory Handoff**: End every task with the standard **Handoff Report**.

---

## 3. Dynamic Collaboration Protocol (1 to 14 Agents)

To keep execution smooth even when only a subset of agents is active (e.g., via `/panggil-agent`), follow these 4 adaptive rules:

### A. File-Based Contracts & Readiness Gates (*Contract Readiness Gate*)
Agents coordinate through 5 documents in `docs/`:
- `docs/project-brief.md` — *(Optional, for new projects)* Blueprint from `planner` + user Q&A: product goal, tech stack, MVP scope, and phase roadmap.
- `docs/plan.md` — Technical design: module structure, data/state flow, dependencies, and work order (authored by `architect`).
- `docs/db-schema.md` — *(For database projects)* Tables/collections, columns, data types, relations, constraints, and indexes (authored by `data-architect`).
  - **Data Gate (`STATUS`)**: Line 1 of `docs/db-schema.md` must state `STATUS: DRAFT` or `STATUS: LOCKED`. `backend` may write production migrations/queries only after it becomes **`STATUS: LOCKED`**.
- `docs/api-contract.md` — Endpoints, request/response schemas, and error format (authored by `backend`).
  - **Parallel Gate (`STATUS`)**: Line 1 of `docs/api-contract.md` must state `STATUS: DRAFT` or `STATUS: LOCKED`. `ui-builder` and `test-writer` may run in parallel as soon as it becomes **`STATUS: LOCKED`**.
- `docs/decisions.md` — Active architecture/technical decision log (**max 30 concise entries**; archive or summarize older entries to avoid context bloat).

### B. *Graceful Fallback* (When Upstream Agents Are Skipped)
- **Without `planner`**: If the project already exists or requirements are clear, start directly with `hr-scouter`, `orchestrator`, or `architect`.
- **Without `hr-scouter`**: `orchestrator` picks the pipeline directly from the user's prompt.
- **Without `architect`**: If `docs/plan.md` does not exist, `orchestrator` (or the first active agent) follows `docs/project-brief.md` or writes a brief `docs/plan.md` before coding.
- **Without `backend` (UI-first / Mock)**: If `ui-builder` runs alone without `backend`, use mock state or data schemas from `docs/plan.md`.

### C. *Dynamic Handoff Routing* (When Downstream Agents Are Skipped)
Each agent has an ideal `handoff_to` list. If a target agent is **not active** in the current session:
- **Return the handoff report to `orchestrator`** (or directly to the **User** if running without an orchestrator).
- `orchestrator` routes the work to the next active agent.

### D. *Role Absorption* (Small Team Consolidation)
For lean runs (2–3 agents), `hr-scouter` or `orchestrator` merges minor roles via the sub-agent `Prompt`:
- `ui-builder` absorbs the `a11y-style-checker` checklist.
- `backend` / `debugger` absorbs `test-writer` unit/regression tests.
- `reviewer` or `orchestrator` absorbs `docs-release`.

---

## 4. Standard Handoff & Document Templates

### A. Standard Handoff Report (Required at End of Task)
```markdown
## Handoff: <sender_agent> -> <receiver_agent_or_orchestrator>
- Effort     : <pro/flash/flash_lite> (<low/medium/max>)
- Task       : <what was requested>
- Result     : <what was completed + list of created/modified file paths>
- Verified   : <typecheck/lint/test results before handoff>
- Decisions  : <key technical decisions + brief rationale>
- Pending    : <deferred items, edge cases, or risks>
- Next       : <next step for receiver + definition of done>
```

### B. `docs/project-brief.md` & `hr-scouter` Templates
- The canonical `docs/project-brief.md` blueprint template is defined in `planner/SKILL.md`.
- The canonical `hr-scouter` team formation table, 1-by-1 commands, and 3 effort presets (`Low` / `Medium` / `Max`) are defined in `hr-scouter/SKILL.md`.

### C. Concise `docs/plan.md` & `docs/api-contract.md` Templates
```markdown
# Implementation Plan: <Feature/Task Name> (`docs/plan.md`)
- **Goal**: <1-2 sentences>
- **Shared Files / New Dependencies**: <packages or shared types if any>
- **Data / State Schema**: <main tables/models/state>
- **Modules & Ownership**: <files to create/edit per agent>
- **Risks & Execution Order**: 1. ... 2. ...
```
```markdown
STATUS: DRAFT | LOCKED (`docs/api-contract.md`)
# API Contract: <Module Name>
## `<METHOD>` `<path>` | Auth: <Public/Bearer>
- **Request**: `{ "field": "type (validation)" }`
- **Response (`200/201`)**: `{ "data": { ... } }` | **Error**: `{ "error": { "code": "...", "message": "..." } }`
```

---

## 5. The 14 Sub-Agents (Stack-Agnostic & Zero Overlap)

System prompts, `tools:` allowlists, and canonical models for each agent live in `agents/<agent-name>/agent.md`.

| No | Agent | Tier / Tool `Model` | Effort | Write Tools | Ownership (`owns`) | Ideal Handoff | Focus & Boundaries (*Out of Scope*) |
| :- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 00 | `researcher` | Hydrogen (`flash`) | `low` / `medium` / `max` | `false` | `[]` | `planner`, `hr-scouter`, `architect` | Verify runtime/SDK versions, compatibility, breaking changes. *No MVP decisions or file edits.* |
| 0A | `planner` | Argon (`pro` / `inherit`)* | `low` / `medium` / `max` | `true` | `docs/project-brief.md` | `hr-scouter`, `architect`, `orchestrator` | Interactive MVP Q&A, stack, & roadmap (prefer `/planner` in main chat). *No app code.* |
| 0B | `hr-scouter` | Hydrogen (`flash`) | **`medium` (fixed)** | `false` | `[]` | `orchestrator` / User | Audit repo, recommend team, `owns`, 1-by-1 commands & presets (prefer `/hr-scouter` in main chat). |
| 1 | `orchestrator` | Hydrogen (`flash`) | `low` / `medium` / `max` | `true` | `docs/decisions.md` | All agents | Pick pipeline, maximize safe parallelism, prevent file conflicts, cap revise loop at 2x. *No feature code.* |
| 2 | `architect` | Argon (`pro` / `inherit`) | `low` / `medium` / `max` | `true` | `docs/plan.md`, `src/types/`, `shared/` | `backend`, `ui-builder`, `refactorer` | Technical blueprint, shared type contracts, YAGNI. *No business/UI logic or detailed DB schema.* |
| 2B | `data-architect` | Argon (`pro` / `inherit`)* | `low` / `medium` / `max` | `true` | `docs/db-schema.md` | `backend`, `planner`, `architect` | Design tables, relations, constraints, indexes & lock `STATUS: LOCKED`. *No migrations/API.* |
| 3 | `reviewer` | Argon (`pro` / `inherit`) | `low` / `medium` / `max` | `false` | `[]` | Workers / `docs-release` | Audit diff vs contracts, logic, security, tests; verdict `APPROVE`/`REVISE`. *No code edits.* |
| 4 | `backend` | Hydrogen (`flash`) | `low` / `medium` / `max` | `true` | Server/API/data modules, DB migrations, `docs/api-contract.md` | `ui-builder`, `test-writer`, `reviewer` | Lock `docs/api-contract.md` (`LOCKED`), implement logic/API/migrations, Pre-Handoff Check. *No UI.* |
| 5 | `ui-builder` | Hydrogen (`flash`) | `low` / `medium` / `max` | `true` | UI components, pages, view states, styling | `a11y-style-checker`, `test-writer`, `reviewer` | Build UI matching `LOCKED` contract (or mock), handle 4 states (`loading`, `empty`, `error`, `success`). |
| 6 | `debugger` | Hydrogen (`flash`) | `low` / `medium` / `max` | `true` | Bug-related files only | `test-writer`, `reviewer` | Reproduce bug, prove root cause, minimal safe fix + Pre-Handoff Check. *No broad refactors.* |
| 7 | `refactorer` | Hydrogen (`flash`) | `low` / `medium` / `max` | `true` | Target refactoring files | `test-writer`, `reviewer` | Reduce duplication & simplify structure with 100% identical external behavior. *No new features.* |
| 8 | `test-writer` | Hydrogen (`flash`) | `low` / `medium` / `max` | `true` | `tests/`, `**/*.test.*`, `**/*.spec.*` | `reviewer` / code owner | Write & run deterministic tests (happy path, edge cases, errors). *Never edit production code.* |
| 9 | `a11y-style-checker` | Hydrogen (`flash_lite`/`flash`) | `low` / `medium` / `max` | `false` | `[]` | `ui-builder`, `reviewer` | Audit contrast, ARIA/labels, keyboard nav, `alt`, headings, & visual consistency. *No code edits.* |
| 10 | `docs-release` | Hydrogen (`flash_lite`/`flash`) | `low` / `medium` / `max` | `true` | `README.md`, `CHANGELOG.md`, `VERSION.md`, `docs/` | `orchestrator` | Sync docs, changelog, & propose Conventional Commits. *Never `git push` or `merge`.* |

---

## 6. Pipelines & Maximized Parallel Execution

### A. Initiation & Pre-Execution
1. **New Project (0-to-1)**: Run **`/planner`** for interactive MVP & workflow planning (`docs/project-brief.md`).
2. **Team & Effort Sizing**: Run **`/hr-scouter`** to scan the repo and get team formation, `owns`, 1-by-1 commands, and effort presets (`low`/`medium`/`max`).
3. **Flexible Execution**: Run any 1-by-1 or parallel subset via **`/panggil-agent`** or **`/teamwork`** / `swarm`.

### B. Standard Pipelines (Full Parallel vs Lean Mode)
`orchestrator` selects a pipeline below, skips inactive agents, and **launches every non-conflicting step in parallel**:

| Task Type | Full Parallel Pipeline (`||` = Parallel) | Lean Mode (2–4 Agents) |
| :--- | :--- | :--- |
| **New Project (0→1)** | `[researcher \|\| planner]` → `hr-scouter` → `[architect \|\| data-architect]` *(lock `db-schema.md`)* → `backend` *(lock `api-contract.md`)* → `[backend \|\| ui-builder \|\| test-writer (contract tests)]` → `[a11y-style-checker \|\| test-writer (run) \|\| reviewer \|\| docs-release]` | `planner` → `backend` → `[ui-builder \|\| test-writer]` → `reviewer` |
| **Feature** | `hr-scouter` → `orchestrator` → `[architect \|\| data-architect]` → `backend` *(lock contract)* → `[backend \|\| ui-builder \|\| test-writer (contract tests)]` → `[a11y-style-checker \|\| test-writer (run) \|\| reviewer \|\| docs-release]` | `orchestrator` → `backend` *(contract + API)* → `[ui-builder \|\| test-writer]` → `reviewer` |
| **Bugfix** | `orchestrator` → `debugger` → `[test-writer \|\| reviewer \|\| docs-release]` | `debugger` *(fix + regression test)* → `reviewer` |
| **Refactor** | `orchestrator` → `architect` → `refactorer` → `[test-writer \|\| reviewer]` | `refactorer` → `reviewer` |
| **UI Only** | `orchestrator` → `ui-builder` → `[a11y-style-checker \|\| test-writer \|\| reviewer]` | `ui-builder` *(with self-a11y check)* → `reviewer` |

### C. Parallel Execution Rules (`invoke_subagent` Batching & `/swarm`)
Maximize concurrency whenever `owns` paths do not overlap:
1. **Batch `invoke_subagent` Calls in the Same Turn**: Whenever 2+ agents have disjoint `owns` (or are Read-Only), invoke them **simultaneously in a single turn** instead of sequentially.
2. **Workspace Isolation for Concurrent Writers**:
   - **Concurrent Writers** (e.g., `[backend || ui-builder || test-writer]`, or multiple sharded `backend` workers on different modules): use `workspace: "share"` (or `"branch"` for isolated clones) so builds/tests do not collide on the same working tree.
   - **Read-Only Agents** (`researcher`, `hr-scouter`, `a11y-style-checker`, `reviewer`): always use `workspace: "inherit"`.
3. **Contract-Unlocked Parallelism**:
   - Once `docs/db-schema.md` is `STATUS: LOCKED`, `backend` can start immediately without waiting for unrelated UI planning.
   - Once `docs/api-contract.md` is `STATUS: LOCKED`, `ui-builder` AND `test-writer` (API/integration test scaffolding) can run **in parallel** alongside remaining `backend` internals.
4. **Parallel QA & Release**:
   - After implementation passes *Pre-Handoff Check*, run `[a11y-style-checker || test-writer || reviewer || docs-release]` in parallel whenever their targets do not collide.

---

## 7. Flow Control & *Definition of Done* (DoD)

### Flow Control & Conflict Prevention
- **Revise Loop Cap (`revise_loop`)**: Max **2 rounds** of revisions between workers and `reviewer`/`a11y-style-checker`. If still `REVISE` after 2 rounds, escalate to `orchestrator` / User.
- **File Conflict Rule**: Two agents must never edit the same file concurrently. If multiple agents must touch a shared file (e.g., `package.json`), `orchestrator` sequences those edits.
- **User Escalation Triggers**:
  1. Ambiguous or contradictory requirements.
  2. Breaking changes to an API contract or database schema.
  3. Destructive actions: data deletion, `git push`, `merge`, or deploy.

### *Definition of Done*
- [ ] Implementation matches `docs/project-brief.md` / `docs/plan.md` and `orchestrator` acceptance criteria.
- [ ] Passes *Pre-Handoff Check* (build/typecheck/lint) and relevant tests pass.
- [ ] `reviewer` issues **APPROVE** (or self-verification passes when running without `reviewer`).
- [ ] Zero hardcoded credentials or secrets.
- [ ] `README.md` or `CHANGELOG.md` updated if application behavior changed.
