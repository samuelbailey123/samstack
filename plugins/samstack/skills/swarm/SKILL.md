---
name: swarm
description: "Fan N workers out over independent slices, drain them, and return one report. Use for a sweep or migration across many files or repos ('do this for every X', 'apply it across the codebase'), parallel coverage or audits, races between approaches, and gauntlets. Inline wins when you can do the work faster yourself, and so does one script when the edit is mechanical."
---

# Swarm

Workers each take a slice, you drain them, one report comes back. Adapted from pstack's swarm ([cursor/plugins](https://github.com/cursor/plugins)), MIT, Copyright (c) 2026 Lauren Tan: see `LICENSE`.

Before fanning out, check the two cheaper options in `samstack:parallelism`: do it inline, or write the one script that does or proves the job. A deterministic tool beats fan-out. Keep a checklist with one entry per phase (Frame, Fan out, Aggregate, Report).

## Phase A: Frame

1. State the done predicate and the artifact or report the swarm must return.
2. Enumerate every item first: each file, repo, case or check the job covers. Done means done equals enumerated, so the list is the coverage bar and every item belongs to exactly one slice.
3. Choose the shape: partition into slices, race N workers on identical briefs, or mix both. For a race or a mixed shape, declare `first pass`, `rank all` or `best-of` before launching. For a model race, name each arm's `model` up front.
4. Size the swarm against the cap on live sessions in `samstack:parallelism`. When the slices outnumber the room under it, run waves: drain one wave before launching the next. Never cut slices to fit, because a slice that never ran is a gap.
5. Give each worker that writes its own writable output: `isolation: "worktree"` on the `Agent` call, and files no other worker touches. When workers verify or measure commits, each brief names the exact SHAs, and a measurement brief also names the method (sample count, what one sample is, order). The worker records both in its result.

## Phase B: Fan out

Launch all of a wave's workers in one message, as `general-purpose` agents. Every brief stands alone, because a subagent can't ask you anything: the goal, the scope, its exact slice or race arm, how to verify, and what to report. Reports are `PASS`, `ISSUES` or `BLOCKED`, each with evidence. A worker that can prove a defect reports `ISSUES` and lists every issue it can prove, not only the first.

Workers never run git: the orchestrator commits. If a worker drops out, go on with the rest and note it.

## Phase C: Aggregate

Read the results. Drop one that doesn't record the SHAs and method its brief names, and relaunch that worker once. A second miss is a gap, and a gap is not a pass. Every enumerated item needs a result. For a race, apply the selection rule declared up front. Never paste raw worker output.

A blocked slice is named with the question that would unblock it, and the rest of the swarm finishes meanwhile.

## Phase D: Land and report

The swarm belongs to one piece of work with one ticket or note. Each slice's changes land in that work's PRs: one commit per slice, in small flat PRs that group slices only while a PR stays small, never more than two stacked, and each proven by the loops it touches (`samstack:testing`). A finding that isn't fixed there is dropped, not filed for later.

Return one consolidated report, answer first: how many of the enumerated items passed. Then a result table, one-line issues, gaps or dropouts, and the race rule when one was used.
