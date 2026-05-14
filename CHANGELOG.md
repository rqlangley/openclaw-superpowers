# Changelog

All notable changes to openclaw-superpowers will be documented in this file.

## [v0.1.0] - 2026-05-14

Initial release — native OpenClaw port of obra/superpowers.

### What's included

- 3-agent architecture: `coding` (orchestrator), `implementer` (leaf), `reviewer` (leaf)
- 10 skills: `software-dev`, `writing-plans`, `subagent-driven-development`,
  `test-driven-development`, `requesting-code-review`, `receiving-code-review`,
  `dispatching-parallel-agents`, `using-git-worktrees`, `finishing-a-development-branch`,
  `systematic-debugging`, `verification-before-completion`
- Full supporting files: prompt templates, testing anti-patterns reference,
  debugging technique docs, code reviewer template
- Config snippet for openclaw.json integration

### Differences from upstream obra/superpowers

- Translated from Claude Code (`Task` tool, `TodoWrite`) to OpenClaw (`sessions_spawn`,
  `update_plan`, `sessions_yield`)
- Agent roles made explicit (`coding`, `implementer`, `reviewer`) rather than generic
- Skill discovery via OpenClaw YAML frontmatter (`description:` field)
- No bootstrap/session-start mechanics (OpenClaw triggers skills via description matching)
- Removed Claude Code / Anthropic-specific governance content
- Model names replaced with tier placeholders (`<fast-model>`, `<mid-model>`, `<top-model>`)
