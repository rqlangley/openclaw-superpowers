---
name: software-dev
description: Use when writing, modifying, reviewing, or debugging any code, scripts, automations, or software projects — including small utilities and one-off scripts.
---

# Software Development — Delegation to Coding Agent

When any software development work is requested — writing code, modifying
existing code, debugging, building automations, creating scripts, or reviewing
code — you **must** delegate to the `coding` agent via `sessions_spawn`.

This applies even to small tasks. The threshold for delegation is:
- **Delegate to coding agent:** anything involving writing or modifying code,
  logic decisions, testing, or architecture — regardless of size
- **Handle inline (no spawn):** purely informational questions *about* code
  (explaining a concept, reviewing a diff you've been shown without needing to
  produce output)

## Agent Architecture

The `coding` agent is a **software engineering orchestrator**, not a leaf
implementer. Its job is to coordinate the full development workflow:

- **`coding`** — orchestrator. Reads plans, breaks work into tasks, dispatches
  subagents, reviews results, and reports back.
- **`implementer`** — leaf agent. Builds one task at a time. Cannot spawn
  sub-subagents. Follows TDD, writes tests first, then code.
- **`reviewer`** — leaf agent. Reviews code and specs for correctness, quality,
  and spec compliance. Cannot spawn sub-subagents.

The coding agent has a full Superpowers-derived skill set for engineering rigor:

| Skill | Purpose |
|---|---|
| `writing-plans` | Structured feature planning before implementation |
| `subagent-driven-development` | Dispatch implementer + reviewer per task |
| `requesting-code-review` | Focused code review with precise context |
| `dispatching-parallel-agents` | Run independent tasks simultaneously |
| `test-driven-development` | Tests-first implementation discipline |
| `systematic-debugging` | Methodical failure diagnosis |
| `verification-before-completion` | Final validation before declaring done |
| `receiving-code-review` | Act on reviewer feedback (fix, or push back) |
| `using-git-worktrees` | Isolated workspace per feature branch |
| `finishing-a-development-branch` | Cleanup, merge prep, branch completion |

**The main session does NOT need to manage this inner workflow.** Just delegate
the goal to `coding` and let it orchestrate.

## How to Delegate

Spawn the coding agent with a clear brief:

```
sessions_spawn(
  agentId="coding",
  task="<detailed task description>",
  label="coding: <short description>"
)
```

Your task brief must include:
1. **What to build/change** — be specific
2. **Inputs, outputs, side effects** — the behavioral contract
3. **Relevant context** — project location, existing patterns, constraints
4. **Architecture preferences** — any decisions already made, or ask it to
   surface options before coding
5. **Any reference files** — paths to existing code or examples to follow

## Example Brief

```
Build a Python script that watches a directory for new CSV files and imports
them into a SQLite database.

Inputs: CSV files dropped into ~/data/incoming/
Outputs: rows inserted into ~/data/records.db, processed files moved to ~/data/done/
Side effects: logs to ~/data/import.log

Project location: ~/projects/csv-importer/
No existing code — start fresh, initialize git.
Use pytest for tests. Follow TDD.
```

## When You Receive the Result

The coding agent will announce its result back to this session when done.
Review the summary and relay it to your human partner with:
- What was built
- Key decisions made
- Test results
- Any open questions

Do not relay raw internal metadata — rewrite in plain assistant voice.

## Handling a BLOCKED Message

If the coding agent announces a `BLOCKED` message, the task is paused waiting
for a clarification on spec ambiguity. When this happens:

1. **Relay the question to your human partner** clearly — rephrase in plain language,
  don't forward the raw internal format. Explain briefly where in the task
  the agent is stuck.
2. **Wait for their answer.**
3. **Resume the coding agent** by sending the answer directly into its session:
   ```
   sessions_send(sessionKey="<key from BLOCKED message>", message="<answer>")
   ```
4. The coding agent will pick up the answer and continue from where it paused.

**Note:** Sub-agent sessions stay alive for up to 60 minutes. If your human partner takes
longer than that to respond, the session will have expired — restart the task
with the clarification included in the brief.

## Config Requirement (for reference)

For this delegation to work, `openclaw.json` must include:
```json
"subagents": {
  "allowAgents": ["coding", "implementer", "reviewer"],
  "maxSpawnDepth": 2
}
```
If spawning fails with an agent-not-allowed error, check that your config
includes these entries (see `config/openclaw-config-snippet.json` in this
repo for a template).
