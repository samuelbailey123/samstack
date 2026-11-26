---
name: interrogate
description: "Adversaries try to break a change or a design, and only findings proven by a run survive. Use as the adversarial step of the security pass on a High change (auth, billing, secrets, signing, CI and hooks, migrations, crypto), and for 'tear this apart', 'adversarial review', 'stress test this', 'find blind spots' or 'challenge this'."
---

# Interrogate

Adversaries attack a change or a design. A finding survives only when a run proves it. Adapted from pstack's interrogate ([cursor/plugins](https://github.com/cursor/plugins)), MIT, Copyright (c) 2026 Lauren Tan: see `LICENSE`.

Report the findings; fixing them is the caller's job. For a High change this is the adversarial step of the security pass in `samstack:security`, run on the merged change as the last step of its work.

## Frame

The scope is what the user names. Otherwise it is the change's diff from its merge base with the base branch, through the working tree; for a change that already merged, the diff of its merge.

State the intent in your own words, from the user's message, the commits and the PR description: what the change is for and what it must never do. Ask the user when the intent is unclear, since only they know it.

## Fan out

Launch three adversaries by default, never fewer than two, in one message as `general-purpose` agents with `isolation: "worktree"`. They are live sessions, so count them against the cap in `samstack:parallelism`. Copy the scope's uncommitted changes into each worktree. One model given one brief finds one adversary's worth of failures, so vary the `model` across them when the `Agent` tool offers more than one, and give each adversary a different angle:

- **Input and correctness:** empty, huge, malformed and out-of-range input, off-by-one and encoding edges.
- **State and order:** concurrency, retries, repeats, out-of-order and partial runs, restarts mid-operation.
- **Abuse and trust:** another user's resource, a forged or expired credential, a hostile value reaching a shell, a query or a URL.
- **Failure and recovery:** a dependency that is down, slow or wrong; a rollback; a half-applied migration.

Each adversary gets the scope, the intent, its angle and this brief:

> Break this change. Prove each finding with a run against the real code: a repro script, or command output that shows the wrong behavior, on made-up data and throwaway keys. For a design or plan, run a prototype or probe the current tree. Keep repros in your own worktree, and never commit, push or open anything. Report each finding with a title, the exact command, the output it printed, and the output the intent requires. Leave out anything you did not run: style, naming, structure, and risks that need a future change to matter. No findings is a valid result.

## Verify

Rerun each finding's command yourself. Keep a finding when the output shows the failure and the failure breaks the intent. Drop the rest. Merge findings that share a root cause. Adversaries find different failures, so the verdict is the union of the verified findings, not one adversary's list.

Remove every adversary worktree by the name you made it under once the findings are in.

## Output

- **Intent:** the restatement.
- **Findings:** for each verified finding, the command, its output, how it breaks the intent, and the adversaries that found it. Say "no verified findings" when none survive.
- **Dropped:** for each adversary, the count of dropped findings and why.

The caller fixes each verified finding in the change's own PRs, with the repro as the fix's red-then-green loop (`samstack:testing`), or drops it. None is filed for later.

Your reply opens with the verdict: the count of verified findings, or "no verified findings".
