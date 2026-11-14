# NSquad (`skill-nsquad`)

**Modular 14-Sub-Agent Architecture & Multi-Agent Collaboration Protocol for Software Development.**

`skill-nsquad` is a collection of **6 skills** and **14 sub-agent definitions (`agent.md`)** designed to coordinate AI coding teams in a structured way—from zero-to-one (0→1) project initiation, architecture and database schema design, parallel backend–frontend implementation, to quality review and release.

It is stack-agnostic (Web, API, CLI, Library/SDK, Desktop, Mobile, or data pipelines) and **portable**: ready to use out of the box in **Antigravity (AGY)** or adapted to any other AI platform that supports sub-agents (such as **Claude Code**, **OpenCode**, **Cursor**, **Windsurf**, or custom orchestrators).

NSquad stands for Nue Squad (Nue its from my nickname)

---

## Key Features

- **14 Specialized Roles with Clear Boundaries**: Every agent has explicit file ownership (`owns`) and *Out of Scope* rules to prevent conflicting edits.
- **Contract Readiness Gates (`STATUS: DRAFT | LOCKED`)**: `backend` waits for `data-architect` to lock `docs/db-schema.md`, and `ui-builder` waits for `backend` to lock `docs/api-contract.md` before running in parallel.
- **Flexible Effort Selection (`low` / `medium` / `high`)**: `hr-scouter` runs at a **fixed `medium` effort**, while all other agents can be set to `low`, `medium`, or `high` (maximum available on the platform) based on user preference or `hr-scouter` recommendations—without rigid artificial caps.
- **1-by-1 Specific Calling & High Parallelism**: Call any agent individually with custom `owns:` and effort, or launch parallel batches (`[backend || ui-builder || test-writer]`) using isolated `share` workspaces.

---

## Repository Structure

```text
skill-nsquad/
├── agents/                  # 14 standard sub-agent definitions (frontmatter + system prompt)
│   ├── architect/agent.md
│   ├── backend/agent.md
│   ├── data-architect/agent.md
│   ├── debugger/agent.md
│   ├── docs-release/agent.md
│   ├── hr-scouter/agent.md
│   ├── orchestrator/agent.md
│   ├── planner/agent.md
│   ├── refactorer/agent.md
│   ├── researcher/agent.md
│   ├── reviewer/agent.md
│   ├── test-writer/agent.md
│   ├── ui-builder/agent.md
│   └── a11y-style-checker/agent.md
├── nsquad/SKILL.md          # Core architecture, shared rules, contract gates, & pipelines
├── initiate-project/SKILL.md # 0-to-1 funnel (researcher → planner + data-architect → hr-scouter)
├── planner/SKILL.md         # Interactive MVP & roadmap discussion (docs/project-brief.md)
├── data-architect/SKILL.md  # DB schema, relations, constraints, & indexes (docs/db-schema.md)
├── hr-scouter/SKILL.md      # Repo audit, team formation, & 3 effort presets (Low/Medium/High)
└── summon-agent/SKILL.md    # Selective agent runner + global preset options
```

---

## The 14 Sub-Agents & Universal Model Tiers

`skill-nsquad` uses 3 vendor-neutral model tiers (`reasoning`, `standard`, `fast`) so it works on any AI platform:

| No | Agent | Universal Tier | Effort | Write Access | Default Ownership (`owns`) | Primary Role |
| :- | :--- | :--- | :--- | :--- | :--- | :--- |
| 00 | `researcher` | `standard` | `low` / `medium` / `high` | Read-Only | `[]` | Verify runtime/SDK versions, compatibility, & breaking changes |
| 0A | `planner` | `reasoning`* | `low` / `medium` / `high` | Write | `docs/project-brief.md` | Interactive Q&A for MVP scope, stack, & roadmap |
| 0B | `hr-scouter` | `standard` | **`medium` (fixed)** | Read-Only | `[]` | Scan repo, recommend team formation, `owns`, & 3 effort presets |
| 1 | `orchestrator` | `standard` | `low` / `medium` / `high` | Write | `docs/decisions.md` | Pick pipeline, delegate tasks, prevent file conflicts, cap revise loop at 2x |
| 2 | `architect` | `reasoning` | `low` / `medium` / `high` | Write | `docs/plan.md`, `src/types/`, `shared/` | Technical blueprint, shared type contracts, & YAGNI |
| 2B | `data-architect` | `reasoning`* | `low` / `medium` / `high` | Write | `docs/db-schema.md` | Table design, relations, constraints, indexes, & lock `STATUS: LOCKED` |
| 3 | `reviewer` | `reasoning` | `low` / `medium` / `high` | Read-Only | `[]` | Audit diff against contracts, logic, security, & issue `APPROVE`/`REVISE` |
| 4 | `backend` | `standard` | `low` / `medium` / `high` | Write | Server/API/data, DB migrations, `docs/api-contract.md` | Lock API contract, implement business logic, validation, & data access |
| 5 | `ui-builder` | `standard` | `low` / `medium` / `high` | Write | UI components, pages, view states, styling | Build responsive UI matching locked contract (`loading`, `empty`, `error`, `success`) |
| 6 | `debugger` | `standard` | `low` / `medium` / `high` | Write | Bug-related files only | Reproduce bug, prove root cause, & apply minimal safe fix |
| 7 | `refactorer` | `standard` | `low` / `medium` / `high` | Write | Target refactoring files | Reduce duplication & clean structure with 100% identical external behavior |
| 8 | `test-writer` | `standard` | `low` / `medium` / `high` | Write | `tests/`, `**/*.test.*`, `**/*.spec.*` | Write deterministic tests without modifying production code |
| 9 | `a11y-style-checker` | `fast` | `low` / `medium` / `high` | Read-Only | `[]` | Audit accessibility (contrast, ARIA, keyboard) & visual consistency |
| 10 | `docs-release` | `fast` | `low` / `medium` / `high` | Write | `README.md`, `CHANGELOG.md`, `VERSION.md`, `docs/` | Sync documentation, changelog, & propose Conventional Commits |

> \* **Quota-Saving Tip:** For small-to-medium projects, run `/planner` and `/hr-scouter` directly in the main chat session (without spawning a new sub-agent) and merge `data-architect` into `architect`/`backend`.

---

## Installation & Usage

### 1. In Antigravity (AGY)

Copy the 6 skill folders and 14 agent definitions to your global config directory (or into `.agents/` inside your project repository):

**Windows (PowerShell):**

```powershell
# 1. Ensure target directories exist
$skillsDir = "$env:USERPROFILE\.gemini\config\plugins\nsquad\skills"
$agentsDir = "$env:USERPROFILE\.gemini\config\agents"
New-Item -ItemType Directory -Force -Path $skillsDir, $agentsDir | Out-Null

# 2. Copy 6 skills to global nsquad plugin
$skills = @('data-architect','hr-scouter','initiate-project','nsquad','planner','summon-agent')
foreach ($s in $skills) {
  Copy-Item -Path ".\$s" -Destination $skillsDir -Recurse -Force
}

# 3. Copy 14 sub-agent definitions to global agents
Copy-Item -Path ".\agents\*" -Destination $agentsDir -Recurse -Force
```

**macOS / Linux:**

```bash
mkdir -p ~/.gemini/config/plugins/nsquad/skills ~/.gemini/config/agents
cp -r data-architect hr-scouter initiate-project nsquad planner summon-agent ~/.gemini/config/plugins/nsquad/skills/
cp -r agents/* ~/.gemini/config/agents/
```

**Example Commands in AGY Chat:**

- Start a new project (0→1): `/initiate-project Build a CLI POS app with SQLite`
- Interactive blueprint only: `/planner Create a project plan for a notes app`
- Audit repo & size the team: `/hr-scouter Check which agents are needed to add auth`
- Cherry-pick 1-by-1: `/summon-agent backend (high) owns:src/api/auth/ Implement refresh token rotation`
- Parallel batch: `/summon-agent [backend (high) || ui-builder (medium) || test-writer (medium)] Build user profile CRUD`
- Run with a global preset: `/summon-agent preset high Refactor the payment module` *(options: `preset low`, `preset medium`, `preset high`)*

---

### 2. In Other AI Platforms (Claude Code, OpenCode, Cursor, Windsurf, etc.)

All instructions in `skill-nsquad` are standard Markdown files, making them easy to adapt to any multi-agent setup.

#### A. Model Tier Mapping

The `agents/*/agent.md` frontmatter defaults to Antigravity model keys (`pro`, `flash`, `flash_lite`). For other platforms, map the 3 universal tiers as follows:

| Universal Tier | Purpose | Antigravity (AGY) | Claude Code | OpenAI / Codex |
| :--- | :--- | :--- | :--- | :--- |
| `reasoning` | Architecture design & critical QA | `pro` / `inherit` | `opus` / `inherit` | Reasoning / flagship (e.g., `o3`, `gpt-5`) |
| `standard` | Coding, debugging, research, coordination | `flash` | `sonnet` | Standard fast coding (e.g., `gpt-4o`, `codex`) |
| `fast` | A11y checklist & documentation updates | `flash_lite` (`low`) / `flash` (`med`/`high`) | `haiku` | Lightweight / mini (e.g., `gpt-4o-mini`) |

#### B. Tool Name Mapping (`tools:` in `agents/*/agent.md`)

If your platform uses different tool names in agent frontmatter, map them as follows:

| Name in `skill-nsquad`       | Purpose                          | Common Equivalent (Claude Code / Others) |
| :----------------------------- | :------------------------------- | :--------------------------------------- |
| `view_file`                  | Read file (Read-Only)            | `Read` / `read_file`                 |
| `write_file`, `edit_file`  | Create & edit files              | `Write`, `Edit`                      |
| `run_command`                | Run terminal / tests / build     | `Bash` / `terminal`                  |
| `search_web`                 | Web documentation search         | `WebSearch`                            |
| `invoke_subagent`, `swarm` | Spawn sub-agents / parallel work | `Agent` / `Task`                     |
| `ask_question`               | Interactive user prompts         | `AskUserQuestion` / direct chat        |

#### C. Running Without Slash Commands

1. **Claude Code:** Copy `agents/*/agent.md` into `.claude/agents/<name>.md` (adjust `model` and `tools` per the tables above), and reference `nsquad/SKILL.md` in `CLAUDE.md`.
2. **Cursor / Windsurf / Custom Orchestrators:** Add `nsquad/SKILL.md` as a Project Rule (e.g., `.cursor/rules/`), and instruct the primary AI to delegate tasks using the system prompts from `agents/<name>/agent.md`.

---

## Standard Workflow

```text
[New Project 0→1]
researcher ──▶ planner (+ data-architect) ──▶ hr-scouter ──▶ architect
                                                                 │
    ┌────────────────────────────────────────────────────────────┘
    ▼
data-architect (locks docs/db-schema.md)
    │
    ▼
backend (locks docs/api-contract.md) ──▶ [backend ∥ ui-builder] ──▶ [a11y ∥ test-writer] ──▶ reviewer ──▶ docs-release
```

- **Lean Mode (2–4 agents):** `backend` → `ui-builder` → `reviewer` (or `debugger` → `reviewer` for bugfixes).
- **Standard Handoff Report:** Every sub-agent ends its task with a structured summary covering `Effort`, `Task`, `Result`, `Verified`, `Decisions`, `Pending`, and `Next`.

---

## Related Documentation

- [nsquad/SKILL.md](nsquad/SKILL.md) — Full architecture rules, contract gates, & 14-agent matrix.
