---
name: panggil-agent
description: >-
  Invoke one or more specific sub-agents (1-by-1 cherry-pick or parallel batches)
  from the 14-agent squad (researcher, planner, hr-scouter, orchestrator, architect,
  data-architect, reviewer, backend, ui-builder, debugger, refactorer, test-writer,
  a11y-style-checker, docs-release) with per-agent effort (low/medium/high), optional
  owns/workspace overrides, or hr-scouter selections. Use for /panggil-agent,
  "call specific agent", "run agent", or "panggil agent".
argument-hint: "[<agent> [(effort)] [|| <agent2>] <task>] (leave empty to pick from 14 agents)"
---

# Specific Sub-Agent Caller (`panggil-agent`)

Use this skill when the user wants to **call any agent 1-by-1 on demand**, launch **parallel batches (`||`)**, apply a **global effort preset (`low` / `medium` / `high`)**, or execute specific rows from `/hr-scouter` or `/planner`.

---

## 1. Input Modes (6 Flexible Styles)

0. **Interactive Picker (No Agent Specified — `/panggil-agent` or `/nsquad:panggil-agent <task>`)**:
   - Triggered automatically when the user invokes `/panggil-agent` without naming an agent (empty args or task-only).
   - **Step 1 — Pick Agent(s) & Effort via `ask_question`**:
     Call `ask_question` with two questions:
     - **Agent Selection** (`is_multi_select: true`): List all 14 agents with brief roles so the user can pick 1 (manual 1-by-1) or multiple (parallel batch):
       `researcher (SDK/docs lookup, read-only)`, `planner (MVP & roadmap Q&A)`, `hr-scouter (repo audit & formation)`, `orchestrator (pipeline & conflict guard)`, `architect (system blueprint & types)`, `data-architect (DB schema & ERD)`, `reviewer (code/security audit, read-only)`, `backend (API, logic, DB migrations)`, `ui-builder (UI components & styling)`, `debugger (root cause & bugfix)`, `refactorer (cleanup, same behavior)`, `test-writer (automated tests)`, `a11y-style-checker (a11y & UI audit, read-only)`, `docs-release (docs, changelog, release)`.
     - **Effort Level** (`is_multi_select: false`): `medium (Recommended)`, `low`, `high`. *(Note: `hr-scouter` stays `medium`.)*
   - **Step 2 — Task Prompt**:
     If the user already provided `<task>` after `/panggil-agent`, proceed immediately. Otherwise, ask the user in plain text what task they want the selected agent(s) to do.

1. **Single Agent (1-by-1 Specific Invocation)**:
   - Run exactly one agent with its own effort, optional `owns`, and task.
   - Examples:
     - `/panggil-agent debugger (high) Find the root cause of the null pointer in auth`
     - `/panggil-agent ui-builder (low) Fix button contrast and spacing on the login form`
     - `/panggil-agent data-architect (medium) Design docs/db-schema.md for multi-tenant billing`
     - `/panggil-agent backend (high) owns:src/api/auth/ Implement refresh token rotation`

2. **Parallel Batch & Chained Pipeline (`||` and `->`)**:
   - Use `||` (or `parallel:`) to run independent agents **simultaneously in the same turn** (or via `swarm` with `workspace: "share"` for concurrent writers).
   - Use `->` for sequential stages (e.g., lock contract first, then fan out in parallel).
   - Examples:
     - Parallel fan-out: `/panggil-agent [backend (high) || ui-builder (medium) || test-writer (medium)] Build user profile CRUD`
     - Staged parallel pipeline: `/panggil-agent architect (high) -> [backend (high) || ui-builder (high)] -> [a11y-style-checker (low) || reviewer (medium)] Add OAuth2 login`

3. **Custom Subset (Comma-Separated + Per-Agent Effort)**:
   - Example: `/panggil-agent backend (high), ui-builder (medium), reviewer (low) Add password reset`
   - *Rule:* If the selected agents have non-overlapping `owns` and no contract dependency between them, **run them in parallel automatically** rather than sequentially!

4. **Global Effort Preset (`opsi low` / `opsi medium` / `Opsi High` or `opsi high`)**:
   - **🟢 `opsi low` (Fast & Quota-Saving)**: Sets all invoked agents to **`low`** (except `hr-scouter`, which stays **`medium`**). Default lean formation if none named: `[backend (low) || ui-builder (low)]` or `debugger (low)`.
   - **🟡 `opsi medium` (Balanced / Standard)**: Sets all invoked agents to **`medium`**. Default formation if none named: `[backend (medium) || ui-builder (medium)] -> reviewer (medium)`.
   - **🔴 `Opsi High` / `opsi high` (Maximum / Production-Grade)**: Sets all invoked agents to **`high`** *(maximum platform capability, except `hr-scouter` fixed at `medium`)*. Default formation if none named: `architect (high) -> [backend (high) || ui-builder (high) || test-writer (high)] -> reviewer (high)`.

5. **Pick From `/hr-scouter` or `/planner` Output**:
   - Examples:
     - `/panggil-agent run #1 and #3 from hr-scouter` (cherry-pick specific rows 1-by-1)
     - `/panggil-agent use hr-scouter recommendation`
     - `/panggil-agent use Opsi High from hr-scouter`
   - Read the latest `hr-scouter` or `planner` output in this conversation and execute the selected rows/preset.

---

## 2. Quick Reference: 14 Agents, Universal Tiers & Effort

Map `Universal Tier` to your platform's model (`reasoning` -> AGY `pro` / Claude `opus` / Codex flagship; `standard` -> AGY `flash` / Claude `sonnet` / Codex standard; `fast` -> AGY `flash_lite` / Claude `haiku` / Codex mini).

| Agent | Universal Tier | Valid Effort | Write Access | Default Ownership (`owns`) |
| :--- | :--- | :--- | :--- | :--- |
| `researcher` | `standard` | `low`, `medium`, `high` | `false` *(Read-Only)* | `[]` — Runtime/SDK versions, compatibility, & architecture patterns |
| `planner` | `reasoning` *(optional)* | `low`, `medium`, `high` | `true` (`docs/project-brief.md`) | `docs/project-brief.md` — MVP Q&A, stack, & roadmap *(prefer `/planner` in main chat)* |
| `hr-scouter` | `standard` | **`medium` *(fixed)*** | `false` *(Read-Only)* | `[]` — Repo audit, team formation, `owns`, & 1-by-1 commands *(prefer `/hr-scouter` in main chat)* |
| `orchestrator` | `standard` | `low`, `medium`, `high` | `true` (`docs/decisions.md`) | `docs/decisions.md` — Pipeline selection, parallel scheduling, & conflict prevention |
| `architect` | `reasoning` | `low`, `medium`, `high` | `true` | `docs/plan.md`, shared types — Technical blueprint & module contracts |
| `data-architect` | `reasoning` | `low`, `medium`, `high` | `true` | `docs/db-schema.md` — DB tables, relations, constraints, & indexes |
| `reviewer` | `reasoning` | `low`, `medium`, `high` | `false` *(Read-Only)* | `[]` — Quality, security, logic audit, & `APPROVE`/`REVISE` verdict |
| `backend` | `standard` | `low`, `medium`, `high` | `true` | Core modules/services/API/data, DB migrations, & `docs/api-contract.md` |
| `ui-builder` | `standard` | `low`, `medium`, `high` | `true` | UI components, pages, view states, & styling |
| `debugger` | `standard` | `low`, `medium`, `high` | `true` | Bug-related files — Reproduction, root cause, & minimal fix |
| `refactorer` | `standard` | `low`, `medium`, `high` | `true` | Target files — Structural cleanup with 100% identical external behavior |
| `test-writer` | `standard` | `low`, `medium`, `high` | `true` | `tests/`, `**/*.test.*`, `**/*.spec.*` — Automated tests without editing prod code |
| `a11y-style-checker` | `fast` | `low`, `medium`, `high` | `false` *(Read-Only)* | `[]` — Accessibility & visual consistency audit |
| `docs-release` | `fast` | `low`, `medium`, `high` | `true` | `README.md`, `CHANGELOG.md`, `VERSION.md`, `docs/` — Documentation & release |

> **Effort & Alias Rules:**
> - Accept `high` (and `max`/`maks` as aliases) for high effort.
> - **`hr-scouter` always runs at `medium`.** All other agents support `low`, `medium`, or `high` per user preference or `hr-scouter` recommendation.
> - If the user omits effort, ask briefly whether they want **Low**, **Medium**, **High**, or **`hr-scouter` recommendation** (or default to `medium`).
> - Include explicit reasoning-depth instructions in each sub-agent's `Prompt` (`low`: fast checklist/minimal edit; `medium`: thorough spec-aligned work; `high`: deep analysis, all edge cases, strict verification).

---

## 3. Execution Protocol (Parallel-First & Adaptive)

1. **One-Line Formation Confirmation**:
   State the mode, agents, tiers, efforts, and parallel stages (e.g., *"Running [Custom Parallel]: `architect` (reasoning - high) -> [`backend` (standard - high) || `ui-builder` (standard - medium) || `test-writer` (standard - medium)] -> `reviewer` (reasoning - high)..."*).
2. **Parallel Scheduling & Workspace Isolation**:
   - **Launch independent agents simultaneously** in a single turn via multiple `invoke_subagent` calls (or `swarm`).
   - Use `Workspace: "share"` whenever 2+ writer agents run in parallel so their builds/tests do not collide. Use `Workspace: "inherit"` for Read-Only agents (`researcher`, `hr-scouter`, `reviewer`, `a11y-style-checker`).
   - Respect contract gates only when dependent: `data-architect` locks `docs/db-schema.md` before `backend` writes DB queries/migrations; `backend` locks `docs/api-contract.md` before `ui-builder` and `test-writer` integrate real endpoints (otherwise `ui-builder` can run in parallel using mocks).
3. **Graceful Fallback & Role Absorption**:
   - If `planner` is invoked, run interactive Q&A in the main chat to produce `docs/project-brief.md`.
   - If `architect` is skipped, the first active agent works directly from `docs/project-brief.md` or user instructions.
   - If `a11y-style-checker` is skipped while `ui-builder` is active, fold basic a11y checks into `ui-builder`'s prompt.
   - If `test-writer` is skipped while `backend`/`debugger` is active at `medium`/`high`, ask them to include unit/regression tests.
4. **Delegate via `define_subagent` & `invoke_subagent`**:
   - Set `EnableWriteTools: false` for `researcher`, `hr-scouter`, `reviewer`, and `a11y-style-checker`.
   - Require every code writer to run a *Pre-Handoff Check* (typecheck/lint/build) before finishing.
5. **Standard Handoff Report**:
   Every agent ends with:
   ```markdown
   ## Handoff: <sender_agent> -> <next_agent_or_User>
   - Effort     : <reasoning/standard/fast> (<low/medium/high>)
   - Task       : <what was requested>
   - Result     : <what was completed + file paths>
   - Verified   : <typecheck/lint/test results>
   - Decisions  : <key technical decisions>
   - Pending    : <risks or deferred items>
   - Next       : <next step>
   ```
6. **Post-Execution Choice (`Lanjut` vs `Cukup 1 Agent`)**:
   - After a manual 1-by-1 agent (or single-stage batch) finishes and reports its Handoff (unless the user already ran a full `->` pipeline or passed `once`/`cukup`):
     Call `ask_question` (`is_multi_select: false`) to let the user decide:
     1. **`Cukup 1 agent ini saja (Selesai)`** — stop here and return control to the user.
     2. **`Lanjut ke <recommended_next_agent> (sesuai Handoff)`** *(include if Handoff `Next:` names a specific agent)* — invoke that next agent, passing the previous Handoff Report as context.
     3. **`Lanjut, pilih agent lain dari 14 agent`** — re-open the 14-agent `ask_question` picker (+ effort & next task/context), passing the previous Handoff Report forward. Repeat until the user chooses to stop.
