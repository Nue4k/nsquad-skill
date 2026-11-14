---
name: panggil-agent
description: >-
  Invoke one or more specific sub-agents (1-by-1 cherry-pick or parallel batches)
  from the 14-agent squad (researcher, planner, hr-scouter, orchestrator, architect,
  data-architect, reviewer, backend, ui-builder, debugger, refactorer, test-writer,
  a11y-style-checker, docs-release) with per-agent effort (low/medium/max), optional
  owns/workspace overrides, or hr-scouter selections. Use for /panggil-agent,
  "call specific agent", "run agent", or "panggil agent".
argument-hint: "[option: low|medium|max] <agent> [(effort)] [|| <agent2> [(effort)]] <task>"
---

# Specific Sub-Agent Caller (`panggil-agent`)

Use this skill when the user wants to **call any agent 1-by-1 on demand**, launch **parallel batches (`||`)**, apply a **global effort preset (`low` / `medium` / `max`)**, or execute specific rows from `/hr-scouter` or `/planner`.

---

## 1. Input Modes (5 Flexible Styles)

1. **Single Agent (1-by-1 Specific Invocation)**:
   - Run exactly one agent with its own effort, optional `owns`, and task.
   - Examples:
     - `/panggil-agent debugger (max) Find the root cause of the null pointer in auth`
     - `/panggil-agent ui-builder (low) Fix button contrast and spacing on the login form`
     - `/panggil-agent data-architect (medium) Design docs/db-schema.md for multi-tenant billing`
     - `/panggil-agent backend (max) owns:src/api/auth/ Implement refresh token rotation`

2. **Parallel Batch & Chained Pipeline (`||` and `->`)**:
   - Use `||` (or `parallel:`) to run independent agents **simultaneously in the same turn** (or via `swarm` with `workspace: "share"` for concurrent writers).
   - Use `->` for sequential stages (e.g., lock contract first, then fan out in parallel).
   - Examples:
     - Parallel fan-out: `/panggil-agent [backend (max) || ui-builder (medium) || test-writer (medium)] Build user profile CRUD`
     - Staged parallel pipeline: `/panggil-agent architect (max) -> [backend (max) || ui-builder (max)] -> [a11y-style-checker (low) || reviewer (medium)] Add OAuth2 login`

3. **Custom Subset (Comma-Separated + Per-Agent Effort)**:
   - Example: `/panggil-agent backend (max), ui-builder (medium), reviewer (low) Add password reset`
   - *Rule:* If the selected agents have non-overlapping `owns` and no contract dependency between them, **run them in parallel automatically** rather than sequentially!

4. **Global Effort Preset (`opsi low` / `opsi medium` / `opsi max` or `opsi maks`)**:
   - **🟢 `opsi low` (Fast & Quota-Saving)**: Sets all invoked agents to **`low`** (except `hr-scouter`, which stays **`medium`**). Default lean formation if none named: `[backend (low) || ui-builder (low)]` or `debugger (low)`.
   - **🟡 `opsi medium` (Balanced / Standard)**: Sets all invoked agents to **`medium`**. Default formation if none named: `[backend (medium) || ui-builder (medium)] -> reviewer (medium)`.
   - **🔴 `opsi max` / `opsi maks` (Maximum / Production-Grade)**: Sets all invoked agents to **`max`** *(maximum platform capability, except `hr-scouter` fixed at `medium`)*. Default formation if none named: `architect (max) -> [backend (max) || ui-builder (max) || test-writer (max)] -> reviewer (max)`.

5. **Pick From `/hr-scouter` or `/planner` Output**:
   - Examples:
     - `/panggil-agent run #1 and #3 from hr-scouter` (cherry-pick specific rows 1-by-1)
     - `/panggil-agent use hr-scouter recommendation`
     - `/panggil-agent use Opsi Max from hr-scouter`
   - Read the latest `hr-scouter` or `planner` output in this conversation and execute the selected rows/preset.

---

## 2. Quick Reference: 14 Agents, Models & Effort

| Agent | Design Tier | Valid Effort | `invoke_subagent` `Model` | `EnableWriteTools` | Default Ownership (`owns`) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `researcher` | `hydrogen` | `low`, `medium`, `max` | `flash` *(or `inherit`)* | `false` *(Read-Only)* | `[]` — Runtime/SDK versions, compatibility, & architecture patterns |
| `planner` | `argon` *(optional)* | `low`, `medium`, `max` | `inherit` / `pro` | `true` (`docs/project-brief.md`) | `docs/project-brief.md` — MVP Q&A, stack, & roadmap *(prefer `/planner` in main chat)* |
| `hr-scouter` | `hydrogen` | **`medium` *(fixed)*** | `flash` *(or `inherit`)* | `false` *(Read-Only)* | `[]` — Repo audit, team formation, `owns`, & 1-by-1 commands *(prefer `/hr-scouter` in main chat)* |
| `orchestrator` | `hydrogen` | `low`, `medium`, `max` | `flash` *(or `inherit`)* | `true` (`docs/decisions.md`) | `docs/decisions.md` — Pipeline selection, parallel scheduling, & conflict prevention |
| `architect` | `argon` | `low`, `medium`, `max` | `inherit` / `pro` | `true` | `docs/plan.md`, shared types — Technical blueprint & module contracts |
| `data-architect` | `argon` | `low`, `medium`, `max` | `inherit` / `pro` | `true` | `docs/db-schema.md` — DB tables, relations, constraints, & indexes |
| `reviewer` | `argon` | `low`, `medium`, `max` | `inherit` / `pro` | `false` *(Read-Only)* | `[]` — Quality, security, logic audit, & `APPROVE`/`REVISE` verdict |
| `backend` | `hydrogen` | `low`, `medium`, `max` | `flash` *(or `inherit`)* | `true` | Core modules/services/API/data, DB migrations, & `docs/api-contract.md` |
| `ui-builder` | `hydrogen` | `low`, `medium`, `max` | `flash` *(or `inherit`)* | `true` | UI components, pages, view states, & styling |
| `debugger` | `hydrogen` | `low`, `medium`, `max` | `flash` *(or `inherit`)* | `true` | Bug-related files — Reproduction, root cause, & minimal fix |
| `refactorer` | `hydrogen` | `low`, `medium`, `max` | `flash` *(or `inherit`)* | `true` | Target files — Structural cleanup with 100% identical external behavior |
| `test-writer` | `hydrogen` | `low`, `medium`, `max` | `flash` *(or `inherit`)* | `true` | `tests/`, `**/*.test.*`, `**/*.spec.*` — Automated tests without editing prod code |
| `a11y-style-checker` | `hydrogen` | `low`, `medium`, `max` | `flash_lite` (`low`) / `flash` (`medium`/`max`) | `false` *(Read-Only)* | `[]` — Accessibility & visual consistency audit |
| `docs-release` | `hydrogen` | `low`, `medium`, `max` | `flash_lite` (`low`) / `flash` (`medium`/`max`) | `true` | `README.md`, `CHANGELOG.md`, `VERSION.md`, `docs/` — Documentation & release |

> **Effort & Alias Rules:**
> - Accept both `max` and `maks` as maximum effort.
> - **`hr-scouter` always runs at `medium`.** All other agents support `low`, `medium`, or `max` per user preference or `hr-scouter` recommendation.
> - If the user omits effort, ask briefly whether they want **Low**, **Medium**, **Max**, or **`hr-scouter` recommendation** (or default to `medium`).
> - Include explicit reasoning-depth instructions in each sub-agent's `Prompt` (`low`: fast checklist/minimal edit; `medium`: thorough spec-aligned work; `max`: deep analysis, all edge cases, strict verification).

---

## 3. Execution Protocol (Parallel-First & Adaptive)

1. **One-Line Formation Confirmation**:
   State the mode, agents, models, efforts, and parallel stages (e.g., *"Running [Custom Parallel]: `architect` (pro - max) -> [`backend` (flash - max) || `ui-builder` (flash - medium) || `test-writer` (flash - medium)] -> `reviewer` (pro - max)..."*).
2. **Parallel Scheduling & Workspace Isolation**:
   - **Launch independent agents simultaneously** in a single turn via multiple `invoke_subagent` calls (or `swarm`).
   - Use `Workspace: "share"` whenever 2+ writer agents run in parallel so their builds/tests do not collide. Use `Workspace: "inherit"` for Read-Only agents (`researcher`, `hr-scouter`, `reviewer`, `a11y-style-checker`).
   - Respect contract gates only when dependent: `data-architect` locks `docs/db-schema.md` before `backend` writes DB queries/migrations; `backend` locks `docs/api-contract.md` before `ui-builder` and `test-writer` integrate real endpoints (otherwise `ui-builder` can run in parallel using mocks).
3. **Graceful Fallback & Role Absorption**:
   - If `planner` is invoked, run interactive Q&A in the main chat to produce `docs/project-brief.md`.
   - If `architect` is skipped, the first active agent works directly from `docs/project-brief.md` or user instructions.
   - If `a11y-style-checker` is skipped while `ui-builder` is active, fold basic a11y checks into `ui-builder`'s prompt.
   - If `test-writer` is skipped while `backend`/`debugger` is active at `medium`/`max`, ask them to include unit/regression tests.
4. **Delegate via `define_subagent` & `invoke_subagent`**:
   - Set `EnableWriteTools: false` for `researcher`, `hr-scouter`, `reviewer`, and `a11y-style-checker`.
   - Require every code writer to run a *Pre-Handoff Check* (typecheck/lint/build) before finishing.
5. **Standard Handoff Report**:
   Every agent ends with:
   ```markdown
   ## Handoff: <sender_agent> -> <next_agent_or_User>
   - Effort     : <pro/flash/flash_lite> (<low/medium/max>)
   - Task       : <what was requested>
   - Result     : <what was completed + file paths>
   - Verified   : <typecheck/lint/test results>
   - Decisions  : <key technical decisions>
   - Pending    : <risks or deferred items>
   - Next       : <next step>
   ```
