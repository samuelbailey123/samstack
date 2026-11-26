---
name: figure-it-out
description: "Design a playbook when no narrower skill fits, then run it with an audit trail. Use for 'figure it out' and 'handle this end to end', for a large migration or an ambitious multi-part change, and for work a human reviews after stepping away. Rigor scales to the stakes, units run as hypothesis loops, and every decision is logged."
---

# Figure it out

When the task matches no playbook, design one. The first deliverable, before any code, is the workflow itself: phases that scale rigor to the task, run the scientific method, and leave a decision trail a human can audit after stepping away. Adapted from pstack's figure-it-out and show-me-your-work ([cursor/plugins](https://github.com/cursor/plugins)), MIT, Copyright (c) 2026 Lauren Tan: see `LICENSE`.

A narrower skill wins when one fits: `samstack:architect` for one design, `samstack:swarm` for a sweep, `samstack:arena` for candidates at one artifact, `samstack:interrogate` for an adversarial pass. This skill is the playbook that calls them. Keep a checklist, and add the phases below to it as items.

## Phase A: Frame

Ground first, then commit. Don't start the run until you can state:

- **The definition of done as a falsifiable predicate:** a result a command or a loop can show true or false on the real product (`samstack:testing`).
- **Scope, quantified:** rough units and effort, and the blockers that grounding turned up.
- **The rigor level, biased high.** A one-way door or a wide blast radius (`samstack:stability`) gets more, and a reversible low-stakes step gets less. Rigor is gates and artifacts, not trying harder.

Write the framing in a few lines and go on. A long run needs no go-ahead and has no checkpoint short of done: reversible calls are yours, stated in a line so the user can redirect them. Ask only what only the user knows, name the unblocking question, and keep working on everything that doesn't depend on it.

## Phase B: Design the workflow

Decompose into atomic units that each land on their own. Sequence the riskiest unknown first, and build the scaffold and the verification before the features.

- **Build the verification harness before the work,** with a baseline captured from the pre-change state, so each check reads as old value against new value.
- **Run `samstack:architect` for each one-way-door design decision.** It runs `samstack:arena` when the decision is contested. Skip it for mechanical work whose shape is already concrete.
- **Decide what fans out.** Parallelize only across seams, each worker on its own worktree or branch, and don't over-fan (`samstack:parallelism`). For a sweep, call the Skill tool with "samstack:swarm".
- **Plan the landing:** small PRs, one per independent unit, never more than two stacked. Everything the run produces lands in the PRs of the one piece of work this is, or is dropped.
- **Write the phase list down.** The list is what the human reviews.

Then execute the design: put each step on the checklist between the phases here and run it under Phase C, with Phase D's log written as each step lands, not saved for the end.

## Phase C: Run the loop

Each unit is an experiment. State the hypothesis, make the smallest change, measure against the predicate on the real artifact, keep the change if it advanced the predicate, and revert it if it didn't. Verify each unit before starting the next, not in a batch at the end.

- Verify by inspecting the artifact, never a self-report. When something passes too easily, suspect the observation method before the system.
- Pair delegated work with a judge: a read-only subagent that checks the artifact against the predicate. If a worker games the gate, reset and harden the contract. If the gate itself is wrong, fix the gate in its own change rather than routing around it.
- A verdict is `VERIFIED`, `NOT VERIFIED` or `INCONCLUSIVE`. Inconclusive is not a pass, and a negative is never hidden.

## Phase D: Keep the audit trail

Keep one append-only log, a TSV with one row per decision and single-line cells: `ts`, `phase`, `decision`, `why`, `evidence`, `result`. Write each row as you would tell a teammate what you did, in plain words. Evidence is a pointer that resolves (a commit, a PR, `file:line`, an artifact path), never a paragraph.

- Log the fork chosen, the unit finished with its verification result, the pivot or revert with its trigger, the blocker surfaced and the gate fixed. Skip the trivial.
- A wrong call gets a new row that supersedes it. Never edit or delete history.
- The log is a working artifact, kept in a scratch folder out of git. Put it in the PR description or the ticket when a reviewer needs the trail to trust the result.
- Before handing back, read the log against what actually happened: every row maps to a real decision, each row's evidence resolves, and a fork or abandoned approach that shaped the work is logged.

The trail plus the diff is what lets the human come back and trust the work.

## Phase E: Verify and hand back

Check the whole against the Phase A predicate on the real product, not just the harness. Turn any correction that recurred into a check, a lint or a script (`samstack:code-quality`).

Your reply opens with the answer: verified or not against the predicate. Then the playbook you designed, the rigor level and why, the trail's location, and what is still open.
