---
name: stability
description: Change safety, blast radius, resilience, and disk. Use before deploying, restarting, or reconfiguring anything running, or when a service is degraded.
---

# Stability

Availability is a release criterion. A change is not done if it leaves a user-facing surface degraded, and "the tests were green" is no defence: verify what actually ships.

## Change safety

- **Prefer additive and backward-compatible.** Migrations go expand → migrate → contract, never in-place destructive. A new path behind a flag beats editing the live one.
- **Verify the artifact, not the harness.** Where the build bundles, inlines or resolves differently than the tests do (esbuild externals, Vercel env inlining, streaming behaviour), build and boot the real artifact before shipping. Green unit tests have shipped a dead prod twice. A verification loop launches that artifact (`samstack:testing`).
- **Health-check before and after** any change to a running system, and capture the before-state so "it was already broken" is provable. Re-check after a settling delay: a status read seconds after a signal proves nothing.
- **A rollback plan counts only once exercised** — dry-run or self-test. State the rollback path before a risky change.
- **Pin versions.** Keep preview and beta tooling off release and deploy paths. Upgrades are deliberate.
- **Dependencies are chosen, not collected.** Lockfiles are committed. A new dependency gets a look at its maintenance, use and licence first, and CI runs the ecosystem's audit (`npm audit`, `pip-audit`, …).

## DNS and domains

A DNS, nameserver or registrar change reaches every client of the domain at once and undoes slowly, so before one: export the current zone (the export is the rollback), lower the record's TTL ahead of time where you can, then make the change and check it with `dig` against the authoritative nameservers and a public resolver (`@1.1.1.1`). State the rollback before you start.

## Blast radius

- **Enumerate dependents before touching shared infrastructure** — a host, a runner label, a database, a secret. One "what uses this?" beforehand beats an outage after.
- **Config shared by N services changes on all N atomically**, with a coordinated redeploy. A secret rotated on one service of three is an outage that presents as something else.
- **Diagnose before recycling.** A restart, kill or redeploy is a remedy after the root cause is named, never a first resort. Scope lifecycle commands to exact targets (PIDs filtered by age or parent, never a broad name pattern) and check what else matches first.
- **Separate contention from capacity before scaling.** Look for starvation, lock contention and concurrency caps first; adding capacity to a contended system changes nothing.

## Runtime resilience

- **Retry transient dependency failures** with bounded, jittered backoff. Never cache or persist a degraded result.
- **Bound everything that can grow.** Batch unbounded deletes, set client timeouts shorter than the platform's limit, keep work inside the execution window.
- **Silent failure is an outage.** Every degradation path emits a signal that actually pages, and the alerting survives the failure that triggers it (a heartbeat gated on `if: success()` is not a heartbeat).

## Disk is a shared resource

Developer tooling never reclaims anything. An iOS worktree accretes gigabytes it will never free (DerivedData, `Packages/*/.build`, `node_modules`). Simulator *devices* are worse and invisible in worktree accounting: on one dev Mac they filled the volume to under a gigabyte free. ENOSPC then failed builds and test runs, presenting as phantom code errors that cost more time than the disk did.

- **Check before fanning out**: `df -h /System/Volumes/Data`. Under ~50 GB free, reclaim first or reduce concurrency.
- **Simulators are the biggest and least obvious.** Delete devices for runtimes nothing targets. `xcrun simctl delete` takes device UDIDs or names, `unavailable` or `all`; a runtime identifier fails with `Invalid device`, so enumerate UDIDs from `simctl list devices -j`. `xcrun simctl erase <udid>` reclaims a bloated in-use device.
- **DerivedData is fully regenerable**, but scope deletion to the project unless the user agrees to a machine-wide purge.
- **A merged worktree is garbage.** Remove it by name once its branch is merged and on the remote: `git worktree remove <path>`, then `git branch -d <branch>`. `git worktree prune` clears entries whose folders are gone. `git worktree remove` refuses a tree with uncommitted changes; that refusal is the check, so leave it standing.
- **Another agent's worktree is off limits** whatever its state; check who is live before removing a tree you did not create.

Sweep before ending a session that spawned worktrees, and report what was freed.

## Lessons

Past mistakes in this area, one line each: [`lessons.md`](lessons.md). Grep it for the tool or symptom in front of you.
