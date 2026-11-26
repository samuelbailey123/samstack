---
name: testing
description: "How a change is proven: verification loops that drive the real build on made-up data, not tests. Use when proving a change works, fixing a bug, deciding what happens to an existing test, building a repo's loops, or setting up its pre-PR hook, post-merge run or nightly run."
---

# Testing: verification loops

Every change is proven by a verification loop, not a test. A loop launches the real build on made-up data, drives only what changed, and keeps evidence that review reads. This holds in every repo you build, client repos included, unless the client asks to keep their tests; then the client's rules hold. (The author, 2026-11-23.)

## The rules

- **Every change gets a loop.** It runs on made-up data, drives only what changed, reuses the change's incremental build, and keeps evidence that review reads.
- **No new tests of any kind.** No unit, integration, snapshot, UI or end-to-end test, and no new case in an existing test file. A bug fix's proof is a loop that is red before the fix and green after.
- **Existing tests go area by area, as loops land.** An area's tests are deleted in the PR that lands the loops proving what they proved, never before. A test that asserts nothing goes on sight.
- **Core keeps its tests until its CLI loops on a scratch home exist.** Core is auth, permissions, signing, the updater, billing, secrets and crypto, and stored or migrated data. Until then, a core change runs the core tests it touches, and its proof is still a loop.
- **CI and hooks are not core.** They get loops of their own (see "Loops for hooks and CI").
- **A past correction becomes architecture, types or a lint, never a test.** See `samstack:code-quality`.
- **No coverage figure, ever:** no gate, report or badge. Remove coverage gates and tooling when you meet them.

## The flow in every repo

1. **Before the PR,** a hook on the author's machine runs the lints, an incremental build of only what changed (a change to no compiled code builds nothing) and the change's loops. Once the hook passes, the PR opens and merges at once: PR CI gates nothing. The PR names the evidence, and review reads it, except for typos, docs, copy, lint fixes and assets.
2. **After a merge,** `main`'s CI runs the lints, the build and about ten main-flow loops. Re-run a red one once, so a flake reverts nothing. Red again: revert the merge at once, and the author re-lands it with the fix.
3. **Nightly,** every loop runs, plus fuzzing: each loop's driver fed generated input (empty, huge, malformed, out of order).

A check runs only what the change touches. A check that does more, or sits stuck, is a bug in the check: find where the time goes and fix that, never by dropping a loop. The first agent working in a repo that lacks a piece of this flow builds it, in its own small PR.

**Transition:** until a repo's hook runs loops, run its existing tests before the PR, write none, and prove the change with a loop as well.

## What a loop is

A loop is a script in the repo, run by one command, that every agent reuses. `scripts/verify/` is the usual home. Each run:

1. **Reuses the build.** It takes the build the hook or the author already made, with a flag such as `--bin <dir>`. It never cleans, and never builds the whole app.
2. **Makes a scratch home.** That is a fresh temp folder holding the run's own data, config, ports and app identity, so two runs never collide and nothing reads or writes a real user's data. It refuses to start if any configured database, URL or credential points at production.
3. **Seeds made-up data** shaped like production's and written through the product's own write path or CLI. The data is big and awkward enough to exercise the change: enough rows to scroll, a name with an apostrophe, an empty list. Made-up data never reaches production.
4. **Launches the real artifact** through its shipping entry point, and waits until it is usable, not merely up.
5. **Drives only what changed,** through the surface a user or caller touches. It uses stable handles (accessibility labels, roles, routes, command arguments, request paths), never coordinates.
6. **Checks the outcome a user sees,** and reads any side effect back from a second view: the CLI, an API, the stored file or row.
7. **Keeps evidence** in a named folder that cleanup never deletes.
8. **Cleans up** only what the run started, by the PIDs and paths it recorded, never by process name.

Any harness can drive a loop: Playwright, XCUITest, a PTY, curl. What makes it a loop is the real build, made-up data, only the change, and kept evidence.

**A loop counts only once it has been seen to fail.** Run it against the unfixed code or a deliberate break, and check that it goes red for the reason it names. A step that can't pass reports FAIL with its evidence. Never loosen a step so that it passes.

To build a kit for a shipped app (launch, doctor, drive, cleanup and a feature map), use a verification-kit skill if you have one, or build those five pieces by hand. Before building a loop, read the section for your kind of software in [references/loops-by-kind.md](references/loops-by-kind.md). It covers Mac and iOS apps, CLIs, web apps, services, and scripts and hooks, with a worked example.

## Verify only what changed

- Each loop names the paths it proves, in the feature map or a table the hook reads. The hook maps the diff's changed paths to loops and runs those, no others.
- A changed area with no loop gets one in the same PR. The new loop covers the changed behaviour, not the whole feature.
- A diff that changes no behaviour (docs, comments, config the build ignores) owes the lints and the build. Copy or an asset a user sees also owes one loop step that shows it.

## Incremental builds

- The hook builds once, incrementally, in the worktree's own build folder, and hands that build to every loop. A loop never builds.
- Build only the targets the loops launch.
- Keep each worktree's build cache (DerivedData, `.build`, `target/`, `.next/cache`) between runs. Never clean it to speed up a slow hook.
- Cache the build, never the result. A loop always runs, because a cached pass looks the same as a pass that never ran.

## Evidence

Evidence is what review reads instead of the diff. Every run keeps, in its named folder:

- the revision (the commit plus a dirty flag) and the digest of the binary it launched;
- one log of each step's command and output, and ok or FAIL with the time;
- for a screen, its accessibility outline or DOM text, and a picture taken after it settled;
- for stored data, the read-back from the second view;
- for a bug fix, the red run before the fix and the green run after.

A claim rests on text or numbers in the evidence, not on a picture alone: quote the text, count the rows, measure the geometry. A picture is for noticing a problem; a measurement is for claiming one.

## Core loops on a scratch home

Core is proven through a CLI on a scratch home, because a CLI is fast and scriptable, and has no screen to wait for.

- Every core operation is reachable from a CLI that takes a home or config folder. Where one isn't, add the command; it is product code, not a test.
- Each loop runs the operation on made-up data, with known good and known bad input. It checks that only the good input succeeds, and that the bad input fails closed:
  - **Auth and permissions:** a valid session, an expired one, a forged one, and a request for another user's resource.
  - **Signing and the updater:** a correctly signed update, a tampered one, one signed by another key, and a downgrade. Only the first installs.
  - **Billing:** a made-up customer through charge, refund, failed payment and retry, in the provider's test mode. The totals read back match.
  - **Secrets and crypto:** a round trip, a wrong key, and truncated input. Afterwards, grep the whole evidence folder for the secret; any hit fails the loop.
  - **Stored and migrated data:** seed the old schema with made-up rows, migrate, and read every row back through the new code. Migrate again to prove it is idempotent, then run the rollback.
- A loop generates its keys, tokens and accounts in the scratch home for that run. Never a real credential or a live key.
- An area's core tests go once its loops pass, have been seen red against a deliberate break, and run in the hook.

## Loops for hooks and CI

A hook or a CI job is proven by handing it a broken change and watching it refuse:

1. Make a throwaway repo in the scratch home (`git init`, with a bare repo beside it as its remote), or a detached worktree.
2. Commit a deliberately broken change of each kind the check guards: a lint violation, a type error, a failing loop, a planted fake secret.
3. Run the hook, or the CI job's own script, against it. It must exit non-zero, and its message must name the fix.
4. Commit a clean change, and check that it passes and ran only what that change touches.
5. Keep the transcript of every run as the evidence.

Read the hook's own exit code, never one through a pipe. A guard that errors must refuse rather than pass; the shell traps are in [lessons.md](lessons.md).

A one-off script nobody depends on owes less: `bash -n`, `shellcheck` or `py_compile`, one real or dry run, and a line in the PR saying how you checked it.

## Review reads evidence, not the diff

- A shipped app keeps a feature map: one file per user-facing feature, saying how to reach it, how to drive it, and what end state proves it works. The main-flow loops come from it.
- **A review finding needs the command that proves it,** a loop or repro command and its output. A finding nobody ran is dropped. A claim that a change is safe names the one fact it rests on, and runs code to prove it.
- Model checking (TLA+) earns its cost only where a flow has shipped the same class of bug twice.
- A client's website also gets a Lighthouse pass (accessibility, performance, SEO) before it ships and after a change to its pages, with the scores in the PR.

## Reproduce before you fix

A bug report, whether from crash reporting, a teammate or a user, is first replayed with a loop on current `main`:
- still broken: that loop, red, is the fix's proof, and the fix turns it green;
- already fixed on main: the job is to ship a build, not a code change;
- can't reproduce: report the exact steps tried and what's missing.

## Merging on autopilot

When PRs land without a human reading them:
- **Before merge,** the change's loops run, and the PR carries their evidence. A reviewer judges that evidence.
- **After merge,** the main-flow loops drive `main` again, as in the flow above. A loop that is still red on a second run reverts the merge at once, and the author re-lands it with the fix.
- **Rules change on patterns, not one-offs.** Only when several agents repeat the same mistake does it become a check (see `samstack:code-quality`).

## What reaches the screen

Verification is mandatory to run, not to narrate. Report the result, not the mechanism. Name a failure mode only when it is both likely and non-obvious, in one line.

## Figures a screen shows

For every number, status or time a UI or tool displays:
- Name its source of truth and read that directly. Never infer it from a neighbouring signal or match readings to owners by value.
- A reading that isn't live carries its age ("as of 3h ago"). Unknown renders as unknown ("—"), never 0 or a default.
- Before shipping, compare each figure with an independent reading of the same fact: the vendor's own UI or CLI, a second API, or a hand calculation. Record the comparison in the PR, and ship at 99% confidence or don't ship.
- A limit or state that can block the user (rate limits, quotas, "blocked until") shows the window that actually binds, with its reset.

## Lessons

Past mistakes in this area, one line each: [`lessons.md`](lessons.md). Grep it for the tool or symptom in front of you.
