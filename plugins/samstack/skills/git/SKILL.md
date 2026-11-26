---
name: git
description: Branch, commit, PR and push discipline. Use when committing, pushing, opening a PR, or creating a repo.
---

# Git

## The one gate on a push

Is this `main` or `master`? If yes, stop: that work goes through a PR. The one exception is the revert of a merge that turned `main` red (`samstack:testing`), which goes straight to `main`. If no, push it. **Pushing a non-main branch needs nobody's go-ahead**: feature branches, follow-ups to an open PR, opening the PR itself, and force-pushes to amend or re-sign with `--force-with-lease --force-if-includes` (flags on the command, so the user's git config stays as it is). Do them, and never describe one as pending approval. A bare `--force`, which overwrites whatever another push put there, asks first.

That governs *pushing*, not *what is pushed*. Signed commits, only the user's name, no secrets, and holding a PR that carries a known regression all still apply.

## Before pushing to a branch that already has a PR

Read the PR's state with the repo's own ship or status command, else `gh pr view <n> --json state`. Confirm `OPEN`. Pushing to a branch whose PR is merged or closed silently orphans the work; branch fresh from the default branch and cherry-pick instead. This is a check you run, not a question you ask.

Start new work from the default branch.

## Commits

- **Logical units.** One commit per coherent change, not one per file: a three-file change with one purpose is one commit. Unless the repo squash-merges, these land in `main`'s history as they are.
- **One committer: the user.** No co-author trailers, no tool attribution, in the message or anywhere else.
- **Signing.** Commits are signed so they show Verified. Let the signing agent prompt, and leave its socket and the user's git config as they are.

## Pull requests

Fill the description from the commit messages, link the ticket the work belongs to, name the verification-loop evidence the change rests on (`samstack:testing`), and add fitting labels.

PR CI gates nothing. The pre-PR hook is the gate, so a PR is done when its hook has passed and it has merged:

```bash
gh pr merge --merge --match-head-commit <head sha>
```

`--match-head-commit` makes the merge refuse if anything was pushed after the hook ran. Add `--admin` where branch protection demands reviews or checks this flow doesn't use. Don't wait on a PR's CI or decide from it; where the repo has its own ship or status command, ask that where a PR stands. After the merge, read the merged content on `origin/main` ([lessons.md](lessons.md), "A PR can merge before your last push"), and watch `main`'s own run: a red `main` reverts the merge that made it red (`samstack:testing`).

## Repositories

Creating a repo, or running `git init`, needs the user to say so first. New repos are **private** unless they explicitly ask otherwise. A throwaway repo that a verification loop makes in its scratch home, and deletes at cleanup, is not a new repo and needs no go-ahead.

Add `.claude/` and other agent-scratch directories to `.gitignore` on first commit to any repo.

## Dependent chains

When change B builds on unmerged change A, stack them, never more than two deep: if your setup has a stacked-PRs skill, call the Skill tool with "stacked-prs". Independent changes are never a stack: one branch, one flat PR each.

## Lessons

Past mistakes in this area, one line each: [`lessons.md`](lessons.md). Grep it for the tool or symptom in front of you.
