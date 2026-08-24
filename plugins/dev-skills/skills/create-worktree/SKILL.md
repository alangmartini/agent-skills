---
name: create-worktree
description: Create an isolated git worktree and switch the session into it. Use when the user asks to start, create, or work in a worktree, or when project instructions require the current task to run in one.
argument-hint: "[name]"
---

Create a new git worktree and move the session into it, so the task runs on an isolated checkout and branch instead of the shared working copy.

## When to Use

- The user explicitly says "worktree" (start, create, use, work in a worktree).
- Project instructions (`AGENTS.md`, `CLAUDE.md`, memory) require the task to run in a worktree, typically because several agents work the same repo in parallel.

Do not create a worktree for an ordinary branch or feature request. A plain `git checkout -b` is the right tool unless isolation was explicitly asked for.

## Process

1. Treat any argument as the worktree name. With no argument, let the tool generate a random one.
2. In Claude Code, call the `EnterWorktree` tool. Pass `name` when the user supplied one, otherwise call it with no parameters. Never pass `path`: this workflow only creates new worktrees, it never enters an existing one.
3. In agents without that tool, run `git worktree add .claude/worktrees/<name> -b <branch>` from the repository root and treat that directory as the working directory for the rest of the task.
4. Report back in two short lines, `Path: <absolute path>` and `Branch: <branch name>`, then one line saying the session is now inside the worktree and that leaving it (keep or remove) happens at the end of the work.
5. Run every later command from the worktree. Do not change directory back to the original checkout.

Create the worktree immediately. Do not ask follow-up questions first, and do not run other tools in the same step.

## Working Inside the Worktree

- The git stash stack is shared with the main checkout and every other worktree. Prefer a temporary WIP commit over `git stash`. If a stash is unavoidable, push it with a unique message and restore it by SHA rather than popping it.
- Commit the work on the worktree branch before leaving, otherwise removing the worktree discards it.

## Leaving

- `ExitWorktree` with `keep` preserves the directory and branch for later.
- `ExitWorktree` with `remove` deletes both, and refuses while uncommitted changes or unmerged commits exist unless the discard is confirmed.
- Outside Claude Code the equivalent is `git worktree remove <path>` once the branch is merged or no longer needed.

## Red Flags

- Creating a worktree nobody asked for.
- Passing `path` to `EnterWorktree` and silently reusing an existing worktree.
- Running the task from the original checkout after the worktree exists.
- Removing a worktree that still holds uncommitted work.
