---
name: finishing-a-development-branch
description: Use when implementation is complete, all tests pass, and you need to integrate the work - automatically merges to base branch, pushes to remote, and cleans up the worktree
---

# Finishing a Development Branch

## Overview

Guide completion of development work by presenting clear options and handling chosen workflow.

**Core principle:** Verify tests → Merge to base → Push to remote → Clean up worktree.

**Announce at start:** "I'm using the finishing-a-development-branch skill to complete this work."

## The Process

### Step 1: Verify Tests

**Before presenting options, verify tests pass:**

```bash
# Run project's test suite
npm test / cargo test / pytest / go test ./...
```

**If tests fail:**
```
Tests failing (<N> failures). Must fix before completing:

[Show failures]

Cannot proceed with merge/PR until tests pass.
```

Stop. Don't proceed to Step 2.

**If tests pass:** Continue to Step 2.

### Step 2: Detect Environment

**Determine workspace state before presenting options:**

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
```

This determines the cleanup behavior:

| State | Cleanup |
|-------|--------|
| `GIT_DIR == GIT_COMMON` (normal repo) | No worktree to clean up |
| `GIT_DIR != GIT_COMMON`, named branch | Path-based (see Step 5) |
| `GIT_DIR != GIT_COMMON`, detached HEAD | No cleanup (externally managed) |

### Step 3: Determine Base Branch

```bash
# Try common base branches
git merge-base HEAD main 2>/dev/null || git merge-base HEAD master 2>/dev/null
```

Or ask: "This branch split from main - is that correct?"

### Step 4: Merge, Cleanup, and Push

**Do not ask your human partner for a workflow choice.** Perform the following automatically:

#### 4a. Merge to Base Branch

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"

git checkout <base-branch>
git pull
git merge <feature-branch>
```

Run tests on the merged result. If they fail, **stop and report** — do not proceed to push.

#### 4b. Clean Up Worktree

Run Step 5 (Cleanup Workspace) now, while on the base branch.

#### 4c. Delete Feature Branch

```bash
git branch -d <feature-branch>
```

#### 4d. Push to Remote

```bash
git push origin <base-branch>
```

**If no remote exists:** Note it in your report and skip. Flag for your human partner.

**If push fails (e.g., diverged history):** Report the error. Do not force-push without an explicit request.

#### 4e. No PR by default

Do not open a PR unless the original task brief explicitly requested one. Code review already happened inside the review loops — a PR adds friction with no benefit for solo development.

### Step 5: Cleanup Workspace

**Only runs for Step 4b** (merge path). Skipped if we are already in a normal repo (no worktree).

```bash
GIT_DIR=$(cd "$(git rev-parse --git-dir)" 2>/dev/null && pwd -P)
GIT_COMMON=$(cd "$(git rev-parse --git-common-dir)" 2>/dev/null && pwd -P)
WORKTREE_PATH=$(git rev-parse --show-toplevel)
```

**If `GIT_DIR == GIT_COMMON`:** Normal repo, no worktree to clean up. Done.

**If `GIT_DIR != GIT_COMMON` and `WORKTREE_PATH` is under `.worktrees/` or `worktrees/` at the project root:** This worktree was created by the `using-git-worktrees` skill — clean it up:

```bash
MAIN_ROOT=$(git -C "$(git rev-parse --git-common-dir)/.." rev-parse --show-toplevel)
cd "$MAIN_ROOT"
git worktree remove "$WORKTREE_PATH"
git worktree prune  # Self-healing: clean up any stale registrations
```

**If `GIT_DIR != GIT_COMMON` and the path is elsewhere:** Do NOT remove the worktree. Leave it in place.

## Quick Reference

| Step | Action |
|------|--------|
| 1. Verify tests | Hard stop if failing |
| 2. Detect environment | Normal repo vs. worktree |
| 3. Determine base branch | Usually `main` |
| 4a. Merge to base | Automatic |
| 4b. Cleanup worktree | If applicable |
| 4c. Delete feature branch | After merge + cleanup |
| 4d. Push to remote | Automatic |
| 4e. PR? | Only if task brief explicitly requested it |

## Common Mistakes

**Skipping test verification**
- **Problem:** Merging broken code
- **Fix:** Always verify tests before merging

**Asking the human to choose a workflow**
- **Problem:** Creates unnecessary friction and blocks async runs
- **Fix:** The workflow is automatic: merge → push. Only open a PR if explicitly requested.

**Opening a PR by default**
- **Problem:** Adds friction and implies human code review is needed, when it already happened
- **Fix:** Default is merge + push. PR is an opt-in, not the default.

**Pushing before cleanup**
- **Problem:** No real harm, but cleanup should happen first for logical ordering
- **Fix:** cleanup worktree → delete branch → push

**Deleting branch before removing worktree**
- **Problem:** `git branch -d` fails because worktree still references the branch
- **Fix:** Remove worktree (Step 5) first, then delete branch

**Running git worktree remove from inside the worktree**
- **Problem:** Command fails silently when CWD is inside the worktree being removed
- **Fix:** Always `cd` to main repo root before `git worktree remove`

**Force-pushing**
- **Problem:** Rewrites shared history
- **Fix:** Never force-push without an explicit request from your human partner

## Red Flags

**Never:**
- Proceed with failing tests
- Merge without verifying tests on result
- Ask the human to choose a workflow (it's automatic)
- Open a PR unless the task brief explicitly asked for one
- Force-push without an explicit request
- Remove a worktree before confirming merge success
- Clean up worktrees you didn't create (path check)
- Run `git worktree remove` from inside the worktree

**Always:**
- Verify tests before merging
- Detect environment before cleanup
- Merge automatically — don't pause and ask
- Push after every successful merge
- `cd` to main repo root before worktree removal
- Run `git worktree prune` after removal
