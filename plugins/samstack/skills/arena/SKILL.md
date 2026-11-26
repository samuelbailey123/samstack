---
name: arena
description: "Several independent candidates at one task, then pick a base and graft the best parts of the rest. Use for 'arena this', 'try it a few ways' or 'compare approaches', and for a non-trivial artifact (a design, an API, a schema, a prompt, a hard fix) where one attempt would lock in the wrong shape and you can't rank the options without seeing them built."
---

# Arena

N candidates attempt the same task, one reads them all, builds on the strongest and grafts in what the others did better. Adapted from pstack's arena ([cursor/plugins](https://github.com/cursor/plugins)), MIT, Copyright (c) 2026 Lauren Tan: see `LICENSE`.

An arena costs N attempts. Skip it when the shape is already settled or the work is mechanical, and use `samstack:architect` for a design you can rank by thinking alone. Keep a checklist with one entry per phase (Frame, Fan out, Cross-judge, Pick, Graft, Verify).

## Phase A: Frame

Every candidate gets the same prompt, so the prompt is the contract.

1. State the artifact each candidate produces.
2. Derive the rubric: 3 to 6 concrete, gradeable criteria for what success means on *this* task. It is for the picker and the judge. Candidates see only the task.
3. Choose the runners: three by default, never fewer than two. They are live sessions, and so is the judge, so count them against the cap in `samstack:parallelism`; near the cap, run fewer candidates, not no judge. Vary the `model` across candidates when the `Agent` tool offers more than one, because one model given one brief tends to converge on one answer. With one model, give each candidate a different design direction in its brief, or repeat the same brief when the work is generation-bound and not judgment-sensitive.
4. Give each candidate its own output location. For code, that is `isolation: "worktree"` on the `Agent` call. For anything else, a scratch folder outside the repo. Candidates never share a writable path, and never push, open a PR or file a ticket.

## Phase B: Fan out

Launch all N candidates in one message. Each brief stands alone: the task, the path to the shared grounding, its own output path, and an instruction to produce the artifact plus a short rationale naming the alternatives it considered and what it rejected. A subagent can't ask you anything, so an ambiguity in the brief comes back as a guess.

If a candidate produces nothing, go on with N-1 and note the dropout.

## Phase C: Cross-judge

Once every candidate is in, launch one read-only judge: a subagent of the `Explore` type, with "do not edit files" in its brief, on a different model from the candidates when there is one. It scores each candidate, by path label and not by model, on each rubric criterion, and recommends a base. It runs while you read in Phase D. If no second opinion is available, say the judge was skipped.

## Phase D: Pick a base

Read every candidate end to end before picking. Score each against the rubric one criterion at a time, not on holistic feel, and compare with the judge. Agreement on the base confirms the pick. Disagreement means one of you is biased or the rubric was ambiguous: read both rationales before deciding.

Pick the base a future maintainer can extend most easily without breaking invariants. When two feel tied, prefer the cleaner boundary or the smaller API.

## Phase E: Graft

Walk each losing candidate once more for what is worth porting into the base: one or two things each, not most of it. Fold each graft in by hand, so the result stays coherent under one mental model. Pasting a graft in mechanically gives a patchwork.

If the candidates converge on one shape, that is strong agreement: ship the consensus and skip the graft. If they diverge wildly, Phase A was under-specified. Reframe the task and re-run once rather than averaging the divergence.

## Phase F: Verify

Prove the synthesized artifact as you would any other change: a verification loop on the real thing (`samstack:testing`). If verification finds a problem the arena missed, either Phase A was wrong (reframe and re-run) or a candidate caught it and you missed the graft (back to Phase E).

Then remove every candidate worktree and scratch folder by the name you made it under, and keep nothing but the synthesized artifact.

## Landing it

The arena belongs to one piece of work with one ticket or note. Its synthesized artifact lands in that work's PRs, small and flat, never more than two stacked, or it is dropped. No candidate branch outlives the run, and no ticket is filed for an idea that lost.

## Output

One synthesized artifact, and a short synthesis note in the PR description or the ticket: the base and why, the judge's verdict or that it was skipped, each graft with its source candidate, what was rejected and why, any convergence or dropouts, and the verification result.

Your reply to the user opens with the answer, which shape won and where it is, and stops there unless they ask for the note.
