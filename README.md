# openclaw-superpowers

A native port of [obra/superpowers](https://github.com/obra/superpowers) for the [OpenClaw](https://openclaw.dev) AI assistant platform.

---

## Hey, human! 👋

**The easiest way to install this is to let your OpenClaw instance do it for you.**

Clone this repo, then say to your OpenClaw assistant:

> "Please read the README.md in `~/projects/openclaw-superpowers` and help me install the Superpowers framework on my OpenClaw instance. Walk me through the configuration choices first, then set it up."

Your assistant will read the rest of this file, ask you a few questions about how you want things configured, and handle the installation. You don't need to read anything below this section unless you want to.

---

## What this is

This repo packages a **native OpenClaw implementation** of the Superpowers software development framework — a 3-agent architecture with 10 specialized skills that brings structured, TDD-driven development discipline to your OpenClaw instance.

When installed, your main OpenClaw assistant will automatically delegate software development tasks to a dedicated `coding` orchestrator agent, which manages the full inner development loop using `implementer` and `reviewer` subagents. You get plan-first development, test-driven implementation, and two-stage code review — without thinking about any of it.

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

## Installation Guide (for OpenClaw agents)

> **This section is written for OpenClaw agents performing the installation.**
> Walk through the pre-flight checks and customization questions with the user before
> touching anything. Then execute the installation steps in order.

---

### Phase 1: Pre-flight checks

Before making any changes, verify the environment:

- [ ] Check OpenClaw gateway config supports `agents.list` and `maxSpawnDepth` (use `gateway config.schema.lookup`)
- [ ] Check if agent workspaces already exist — `~/.openclaw/workspace-coding`, `workspace-implementer`, `workspace-reviewer` — and warn the user if so (avoid clobbering an existing install)
- [ ] Confirm the plugin-skills directory (default: `~/.openclaw/plugin-skills/`)
- [ ] Check for skill name conflicts with any already-installed skills
- [ ] Confirm the user's Unix username and home directory for path substitution

---

### Phase 2: Ask the user these questions

Ask all of these before making any changes. Collect all answers, then proceed to Phase 3.

#### Models — what to use for each role?

This framework uses three conceptual model tiers. The user doesn't have to pick three different models — using one model for everything works fine. But if they have a fast/cheap model and a powerful one, the tiers let them optimize.

| Tier | Used for | Suggested starting point |
|---|---|---|
| **Fast** | Mechanical implementation tasks | Cheapest model with solid coding ability |
| **Mid** | Orchestration, integration tasks | A capable mid-tier model |
| **Top** | Architecture, planning, code review | Their best available model |

Questions to ask:
- "Do you want to use different models for different tiers, or the same model for everything?"
- If tiered: "Which model should I use for each tier?" (show them what's configured in their OpenClaw instance if possible)
- "Should the top-tier model use extended thinking / reasoning when available?" (only relevant if their top model supports it)

#### What should happen when a development branch is finished?

The `finishing-a-development-branch` skill runs at the end of every feature. Ask the user what they want:

- **Open a pull request** — asks for: target remote (default: `origin`), base branch (default: `main`)
- **Merge directly** — asks for: target branch (default: `main`)
- **Push the branch and stop** — the user handles merge/review manually
- **Commit locally only** — no push, no merge (good for solo/local projects)
- **Deploy after merge** — if they want auto-deploy, ask how (deploy script path, command, service name, etc.)

#### Code review intensity?

- **Full 2-stage review** (spec review before coding + code review after) — most thorough, recommended default
- **Code review only** — skip spec review, faster
- **Light / single-pass** — fastest, least overhead

#### Git worktrees?

Ask: "Do you use git worktrees?" If no, note that `using-git-worktrees` is installed but won't be invoked by default — no action needed, but good to know.

---

### Phase 3: Installation steps

Execute in order:

1. **Create agent workspace directories**
   ```bash
   mkdir -p ~/.openclaw/workspace-coding
   mkdir -p ~/.openclaw/workspace-implementer
   mkdir -p ~/.openclaw/workspace-reviewer
   ```

2. **Install agent config files**
   ```bash
   cp agents/coding/AGENTS.md ~/.openclaw/workspace-coding/AGENTS.md
   cp agents/implementer/AGENTS.md ~/.openclaw/workspace-implementer/AGENTS.md
   cp agents/reviewer/AGENTS.md ~/.openclaw/workspace-reviewer/AGENTS.md
   ```
   Then substitute path placeholders: replace `<username>` with the actual Unix username in all three files.

3. **Install skills**
   ```bash
   cp -r skills/* ~/.openclaw/plugin-skills/
   ```

4. **Patch `openclaw.json`**
   Use the `gateway` tool (not manual file editing) to apply the agent configuration. See `config/openclaw-config-snippet.json` for the required structure. The minimum required additions:
   - Three agent entries in `agents.list` (coding, implementer, reviewer), with correct workspace paths
   - `agents.defaults.subagents.allowAgents: ["coding", "implementer", "reviewer"]`
   - `agents.defaults.subagents.maxSpawnDepth: 2`

5. **Apply model choices**
   In each `AGENTS.md` and in relevant skill files, replace the tier placeholders (`<fast-model>`, `<mid-model>`, `<top-model>`) with the actual model identifiers the user chose. If they chose a single model for everything, use it in all three slots.

6. **Apply branch-completion preference**
   Edit `~/.openclaw/plugin-skills/finishing-a-development-branch/SKILL.md` to reflect the user's chosen strategy (PR, merge, push-only, local-only, deploy).

7. **Apply review intensity preference** (if not using full 2-stage)
   Edit `~/.openclaw/plugin-skills/subagent-driven-development/SKILL.md` to reflect the chosen review configuration.

8. **Restart the OpenClaw gateway**
   ```
   openclaw gateway restart
   ```
   ⚠️ Do this last — a restart ends the current turn. Finish all file edits and confirmations before restarting.

---

### Phase 4: Verify

After restart, confirm with the user:

- Ask the main assistant to run a trivial coding task (e.g. "write a hello world script in Python")
- Confirm it delegates to the `coding` agent (subagent activity should appear in the OpenClaw dashboard)
- Confirm the result comes back to the main session

If it works, the framework is live. Let the user know what was installed and give them a brief summary of the choices made.

---

## Customization Reference

This table covers everything a user might want to change after initial setup, and where to find it.

| What to change | Where it lives |
|---|---|
| Model used for fast/mechanical tasks | `~/.openclaw/workspace-implementer/AGENTS.md` (model field), `subagent-driven-development/SKILL.md` |
| Model used for orchestration/mid tasks | `~/.openclaw/workspace-coding/AGENTS.md` |
| Model used for review/top tasks | `~/.openclaw/workspace-reviewer/AGENTS.md` |
| Branch completion behavior (PR vs merge vs push vs local) | `~/.openclaw/plugin-skills/finishing-a-development-branch/SKILL.md` |
| Review intensity (2-stage vs code-only vs light) | `~/.openclaw/plugin-skills/subagent-driven-development/SKILL.md` |
| Whether extended thinking is used | Respective `AGENTS.md` files (thinking level field in agents.list config) |
| Adding/removing agents from allowed subagent list | `openclaw.json` → `agents.defaults.subagents.allowAgents` |
| Max subagent depth | `openclaw.json` → `agents.defaults.subagents.maxSpawnDepth` |

To re-run any part of the setup (e.g. to change your model choices), just tell your OpenClaw assistant what you want to change and it can apply the edit directly.

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
