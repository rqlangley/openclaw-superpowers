# openclaw-superpowers

A native port of [obra/superpowers](https://github.com/obra/superpowers) for the [OpenClaw](https://openclaw.dev) AI assistant platform.

---

## What this is

This repo packages a **native OpenClaw implementation** of the Superpowers software development framework — a 3-agent architecture with 10 specialized skills that brings structured, TDD-driven development discipline to your OpenClaw instance.

### How is this different from existing ClawHub bridges?

Existing ClawHub bridges for Superpowers are thin proxies that route Claude Code skill invocations through the ClawHub gateway. They work, but they don't take advantage of OpenClaw's native capabilities.

This port is built from the ground up for OpenClaw:

- Uses `sessions_spawn` / `sessions_yield` for agent dispatch and result collection (not Claude Code's `Task` tool)
- Uses `update_plan` for task tracking (not `TodoWrite`)
- Designed for OpenClaw's specific agent roles (`coding`, `implementer`, `reviewer`)
- No Claude Code dependency — fully standalone

---

## Architecture

### 3 Agents

| Agent | Role | Can Spawn? |
|---|---|---|
| `coding` | Orchestrator — plans, dispatches, coordinates | ✅ Yes |
| `implementer` | Leaf — builds one task at a time, follows TDD | ❌ No |
| `reviewer` | Leaf — reviews specs and code, read-only | ❌ No |

The `main` agent (your daily assistant) delegates software tasks to `coding`, which handles the full inner development loop.

### 10 Skills

| Skill | Purpose |
|---|---|
| `software-dev` | Entry point — delegates dev tasks from `main` to `coding` |
| `writing-plans` | Structured implementation planning before any code |
| `subagent-driven-development` | Executes plans via implementer + two-stage review per task |
| `test-driven-development` | TDD discipline for implementer subagents |
| `requesting-code-review` | Code review dispatch template |
| `receiving-code-review` | How to act on review feedback |
| `dispatching-parallel-agents` | Run independent tasks concurrently |
| `using-git-worktrees` | Isolated workspace per feature branch |
| `finishing-a-development-branch` | Merge, push, cleanup after all tasks done |
| `systematic-debugging` | Root-cause-first debugging methodology |
| `verification-before-completion` | Evidence-before-claims gate before marking work done |

---

## Installation

### Step 1: Create agent workspaces

OpenClaw agents each have a dedicated workspace directory. Create the three required for this system:

```bash
mkdir -p ~/.openclaw/workspace-coding
mkdir -p ~/.openclaw/workspace-implementer
mkdir -p ~/.openclaw/workspace-reviewer
```

### Step 2: Install agent config files

Copy each `AGENTS.md` to its workspace:

```bash
cp agents/coding/AGENTS.md ~/.openclaw/workspace-coding/AGENTS.md
cp agents/implementer/AGENTS.md ~/.openclaw/workspace-implementer/AGENTS.md
cp agents/reviewer/AGENTS.md ~/.openclaw/workspace-reviewer/AGENTS.md
```

> **Customize paths:** The AGENTS.md files contain `<username>` placeholders for home
> directory paths. Replace `<username>` with your actual Unix username, and update
> any workspace paths that differ from the defaults.

### Step 3: Install skills

Copy the skills directory into your OpenClaw plugin-skills directory:

```bash
cp -r skills/* ~/.openclaw/plugin-skills/
```

This installs all 10 skills. OpenClaw will auto-discover them via the `description:` field
in each `SKILL.md` frontmatter.

### Step 4: Configure agents in openclaw.json

Add the agent definitions to your OpenClaw config. See `config/openclaw-config-snippet.json`
for the required JSON structure, and the instructions inside for how to apply it.

The minimum required additions are:
1. Three agent entries in `agents.list` (coding, implementer, reviewer)
2. Subagent permissions in `subagents` (allow these agents, set max depth ≥ 2)

### Step 5: Restart OpenClaw gateway

```bash
openclaw gateway restart
```

### Step 6: Verify

Ask your main assistant to do something involving code. It should delegate to the `coding`
agent automatically (via the `software-dev` skill) and you'll see subagent activity in
your OpenClaw dashboard.

---

## Model Selection

The agents work best with a tiered model setup:

- **`<fast-model>`** — your fastest/cheapest model, for mechanical implementation tasks
- **`<mid-model>`** — a capable mid-tier model, for integration and judgment tasks
- **`<top-model>`** — your best model, for architecture, design, and review

Replace these placeholders in the `AGENTS.md` files and skill files with your actual
model identifiers. If you're unsure where to start, a single capable model in all
three slots works fine — the tier guidance is an optimization, not a requirement.

---

## Attribution

This project is a port of [obra/superpowers](https://github.com/obra/superpowers),
which is Copyright (c) Jesse Vincent, licensed under the MIT License.

The original Superpowers framework was designed for Claude Code. The structure,
methodology (plan-first, TDD, two-stage review loops, systematic debugging),
and much of the prose in the skill files derive from that upstream work.

See `UPSTREAM_LICENSE` for the full upstream copyright notice.

---

## License

MIT — see `LICENSE`.
