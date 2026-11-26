---
name: parallelism
description: When subagents and worktrees earn their cost, and the concurrency caps that bite. Use when planning parallel work or creating a worktree.
---

# Parallelism

The floor: **inline beats a subagent for anything you can do faster yourself.** Parallelism is for genuinely independent units, not for looking busy.

## Subagents

Dispatch for **2+ genuinely independent units** — research, audits, sweeps, separate features. Launch them in one message so they run concurrently.

- Briefs are **complete and self-contained**: a subagent cannot ask the user a question. Ambiguity comes back in its results rather than being guessed at.
- Fan-out subagents write **disjoint files** and **never run git operations**. The orchestrator commits. A helper that owns a whole ask, on its own worktree and branch, runs its own git through to the merge.
- Hand review work to a specialist when you keep one (for example a `security-auditor` or `code-reviewer` agent in `~/.claude/agents/`); otherwise brief a general subagent for it.

Before fanning out, check that the brief is complete and that the tools the agent needs are actually available to it. Most subagent failures are an underspecified brief or a missing tool, not a flaky harness.

## Worktrees

A worktree earns its cost only when parallel work could **collide**, or to isolate a risky change. Single-file work and read-only work never need one.

Create one on its own branch from a named base (`git worktree add <path> -b <branch> <base>`), and remove it by name once its branch has merged.

- Independent tasks: one worktree, one flat PR each. A dependent chain: **one agent, one worktree, stacked PRs**, never more than two stacked, and never spread across worktrees.
- Set one cap on live sessions, leads and helpers included, and let only the owner change it (the author runs at most 30). At the cap, new work waits and idle sessions close first, each keeping its branch and a note. Builds are the tighter limit: run at most 3 pre-PR builds at once on one machine. Shared `.git` lock contention is real: on `index.lock` errors, retry with short backoff.
- **Remove by identity, never by history.** A worktree goes only when `git worktree list` shows it on your branch and its commits are on the remote; never another agent's, whatever its state.
- **A new worktree gets no ignored files.** `.env` and friends are not copied in, so a worktree that needs them will fail to boot until you place them yourself.
- A worktree of `~/.claude` goes **outside** `~/.claude`.

Disk cost per worktree and the reclamation routine: call the Skill tool with "samstack:stability".

## Agent teams

`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`. Experimental, token-heavy, and only for genuinely large multi-track efforts. Not a default.

## Budget

After 2–3 honest attempts at the same failing thing, stop repeating it. Write down the premise the attempts share and prove it directly with a loop or a probe (`samstack:code-quality`), then change approach. Ask the user only for what only they can supply (an account, a decision on taste), name the unblocking question, and keep working on everything that doesn't depend on it.

A large run needs no go-ahead and has no stopping point short of done. Enumerate every item first, work all of them until done equals enumerated, and name any blocked item with its unblocking question while the rest finishes. Length is never a reason to stop, defer or check in.

## Lessons

Past mistakes in this area, one line each: [`lessons.md`](lessons.md). Grep it for the tool or symptom in front of you.
