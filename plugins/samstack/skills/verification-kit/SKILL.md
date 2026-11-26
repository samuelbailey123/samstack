---
name: verification-kit
description: "Build or audit a repo's verification kit: the scripts that launch the real app on a scratch home, drive it as a user does and keep evidence, plus a map of its features. Use when a shipped app has no kit, when a review needs proof that a feature works, or to audit a kit that may have drifted."
---

# Verification kit

The kit is what every verification loop in a repo reuses (`samstack:testing`): a way to launch the real app, check that the instance is worth driving, drive it as a user does, keep evidence and clean up, plus a map of the features to drive. A loop is one proof, and the kit is the harness under all of them. Write it for the next agent, who reads it cold, mid-task, having never seen the app.

The kit lives in `scripts/verify/` (or the repo's existing home for loops), with its entry document at `scripts/verify/README.md`. CONTRIBUTING names both.

## Build a kit

**1. Interview the repo, not the user.** Answer from the codebase, and ask the user only what you cannot observe.

- **Surface:** what a user touches: a web UI, a CLI or TUI, a desktop app, an API, a mobile app, a library. A repo can have several. Pick the primary one and note the rest.
- **Run:** how the app starts locally. Prefer the repo's own documented command (package scripts, Makefile, README quickstart). Note ports, environment variables, seed data and auth.
- **Drive:** how an agent can interact with it. Existing harnesses first (Playwright or Cypress specs, expect scripts, PTY helpers, curl-able endpoints, a debug port), then a generic recipe: browser or CDP for web and Electron, a PTY or tmux for a CLI or TUI, plain HTTP for services. [loops-by-kind.md](../testing/references/loops-by-kind.md) has one per kind of software.
- **Observe:** the evidence available: screenshots, terminal transcripts, response bodies, logs, exit codes, stored state.
- **Isolate:** whether two instances can run side by side (ports, data folders, profiles). If not, the kit says so, because refusing to drive a shared instance beats corrupting the user's session.

If the checkout doesn't build or start as it is, fix that first, or report it precisely: a kit written against a broken base teaches wrong steps. When an irrelevant missing asset blocks start-up (a static folder the API never serves, a sample config), the kit may create it, marked as verification scaffolding, and remove it in cleanup.

**2. Write the pieces,** each grounded in what the interview found, with no placeholders left.

- **Launch:** the exact command that starts the app for verification, and how to tell it is ready (a log line, a prompt). It reaches a usable state from an empty scratch home, migrations and seed data included. Ready means usable, not merely answering: a database health check can pass while its init scripts still run. It takes the build the hook or the author already made and never builds. A short-lived CLI or TUI has no server to keep alive, so launch starts each drive in its own isolated PTY or tmux session. Include teardown.
- **Doctor:** one read-only check of "is this instance worth driving?": process up, right build, port owned by this run, auth valid. An agent runs it first whenever anything looks off.
- **Drive:** the harness recipe with this repo's real selectors and commands, not examples. Stable handles (ARIA labels, data attributes, prompt strings, routes) over coordinates and tab order.
- **Evidence:** what to capture and where it goes (the Evidence section of `samstack:testing` lists what each run keeps). State the proof standards. Exercise the real user path, never internal setters or test-only endpoints. Capture the action and the resulting state, not just the final screen. Check side effects (files written, rows inserted, messages sent) beside what is visible. Mock only where a production boundary already isolates the external system. Where the safe path is a dry run or a test mode, observe what it skips (files, network, git refs) instead of trusting its name, because some dry runs still touch the network or open a browser.
- **Cleanup:** tear down what the run started, by the PIDs and paths it recorded, never by process name. Cleanup removes instances and scratch state and never the evidence: proof artifacts survive teardown, in a location the kit names.
- **Helpers:** every script the kit ships is executable, and the README shows its invocation. A helper the reader has to reverse-engineer is not a helper.

**3. Seed the feature map.** Create `scripts/verify/features/README.md` and one file per user-facing feature you can identify, starting with the top 3 to 5 from routes, commands, menus or docs. Follow the shape in [references/feature-map-example/](references/feature-map-example/README.md), an invented notes app and harness. Each file answers from the user's point of view what the feature is, how to reach it, how to drive it with the harness, and what observable end state proves it works. Its four H2s are `Sub-features`, `How to get to it (user POV)`, `Driving it with <harness>` and `Gotchas`. If the seed data lacks a precondition that a journey needs, the journey reports a failure and the reason goes in that feature's `Gotchas`: never weaken the journey to make it pass. The map is the repo's maintained source for verification, so a proof that drives one convenient entry point is incomplete when the map lists others.

**4. Prove the kit before you hand it over.** Run its own instructions end to end once: launch, doctor, drive one mapped feature (one is enough, because the map lets later runs cover the rest), capture evidence, clean up. After cleanup, confirm the evidence still exists at its named location, since a cleanup that eats the proof fails this step. Fix what fails, and run cleanup after every failed iteration so broken attempts strand no processes or ports. Then break that feature on purpose and watch the drive go red for the reason it names. A kit that was never executed is a draft, not a deliverable. If you proved it in a stand-in for the real target (a container in place of a fresh VM, a local stack in place of a remote one), list every setup step the stand-in did that the target will not. Those steps stay unverified until a run on the real target covers them.

## Maintain a kit

A feature map rots the moment the app changes. This pass keeps the kit honest. The unit of rigour is the feature: check every feature file against source and exercise every feature live, without proving every bullet.

Pick one outcome, and say which:

- **clean:** every feature got source and live coverage, and nothing is worth shipping. No branch, no PR.
- **changed:** one PR ships proven corrections to the map, the README or the harness.
- **blocked:** coverage could not finish, or a proven fix could not ship safely. Say exactly what blocked it.

Edit only the kit's own folder, and never product code during a pass. A behaviour the map describes that the app no longer has is either doc drift (fix the map) or a product regression (report it, and don't paper over it in the docs).

1. **Locate the kit.** With several candidates, ask which. With none, build one (above) instead of inventing a target.
2. **Index hygiene.** Read the map's README and list its sibling files. Fix missing, extra, duplicate and dead entries.
3. **Source wave.** Launch one read-only subagent per feature file, together. Each explains from source how its feature works, flags likely doc drift with citations, and returns one live recipe, in this shape: feature summary, source entry points, likely drift or none, one recipe. Subagents never drive the app and never edit files.
4. **Reconcile.** Every feature file has a returned summary. Merge overlapping recipes into as few app states as practical, spot-check the cited drift without re-proving the clean claims, and sweep recent churn for user-facing surfaces missing from the map. Call a surface missing only with a concrete source path.
5. **Live pass,** required even when the source looks clean. You own all the driving, following the kit's own launch model: one long-lived instance driven serially for servers and UIs, a fresh isolated session per drive for short-lived CLIs. Exercise every feature at least once, and hold three invariants whatever fails:
   - Run doctor before the first drive, on each fresh session, and after any failed drive. Where doctor can't see the failure (a wedged UI on a healthy process), reset to a known state or relaunch.
   - Evidence captured so far survives every cleanup, checked at its named location.
   - Nothing a drive started outlives that drive. Clean up the residue of failed iterations whether the session is stuck, exited or shared (for a shared instance, clean the residue, not the instance).

   A doctor failure caused by kit drift is drift: fix it and retry once, restarting only what the fix invalidated, before calling the pass blocked. A feature that can't be reached is `verified-unreachable` only with the concrete prerequisite (auth, entitlement, OS, external state) and the route attempted. If the map omits that prerequisite, that is drift. Re-drive every harness fix live before it ships. Final teardown comes after the last drive, re-proofs included.
6. **Triage.** A wrong or missing description from the user's point of view is doc drift: fix it. Working behaviour the harness can't drive is a harness gap: fix it. App behaviour that is actually broken is a product gap: report it, and keep it out of this PR.
7. **Ship or stop.** For changed, open one PR of proven corrections, after re-reading every changed file. For clean or blocked, open no PR, and report the outcome and the coverage honestly. Keep run notes (features covered, unreachable prerequisites, confirmed drift, outcome) in a scratch folder, and don't commit them.

Adapted from pstack's create-verification-skill and maintain-verification-skill ([cursor/plugins](https://github.com/cursor/plugins)), MIT, Copyright (c) 2026 Lauren Tan: see `LICENSE`.
