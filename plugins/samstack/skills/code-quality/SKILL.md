---
name: code-quality
description: "Standards for shipped code: error handling, observability levels, idempotency, turning a correction into a check, and keeping repos lean. Use when writing non-trivial code, adding logging, fixing a mistake that has happened before, or adding, moving or removing files, docs or scripts."
---

# Code quality

## Design

**KISS**: the simplest solution that is correct and secure. Readability and maintainability first. **YAGNI**: immediate requirements only, no speculative features. SOLID applies, but where it collides with KISS on a small change, KISS wins — an abstraction layer added to satisfy a principle is the thing both rules exist to prevent.

**Subtract before you add.** Delete dead paths, one-caller wrappers and adapters with no second implementation before building on top. Leave the code simpler than you found it.

For the vocabulary of *how* to shape a module (depth, seam, adapter, leverage, locality), use a codebase-design skill if you have one.

## Standards

- Structured error handling with specific failure modes. Verify preconditions before critical or irreversible operations. Long-running operations get a timeout and cancellation.
- **No comments.** Names, types and structure carry the meaning. Nobody reads comments, and agents use them to justify workarounds.
  - A comment survives only for a why the code cannot say. That means behaviour forced by something outside our control (a platform, vendor, protocol or dependency quirk), with its issue link where one exists, or a licence header.
  - A surprise in our own code gets a rename, an extracted function or a type, not prose.
  - A comment never narrates, justifies a workaround or tells the next agent "don't do X". Fix the cause, or encode the constraint as a check (below).
  - Incident history and rejected designs go in the commit message or PR body.
  - A lint suppression is held to the same bar: it stays only for a style-only or faulty rule.
  - Code you touch loses the comments that break this rule. A no-comments skill, if you have one, sweeps a whole diff.
- File and path operations verify existence and permissions first.
- Production code runs on real data and verifiable evidence: stubs, fallbacks and made-up data stay in verification loops.
- **Idempotency**: favour code and commands safe to run twice. Where a solution isn't idempotent, say so explicitly and offer an idempotent alternative.

## Keep the codebase worth copying

Agents copy the nearest example, so a workaround, deprecated path or dead code left standing becomes the pattern. When a mistake is corrected, make the next one impossible at the highest level that works:

1. Architecture or API: one supported way. Delete the old way.
2. Types.
3. A lint whose error names the fix.
4. A verification loop that goes red on the mistake.
5. Docs, last, and only for a judgement call no check can make.

Never a test: proof of a change is a verification loop (`samstack:testing`), and a correction is a check that refuses the mistake. Prove the new check fails on the real past mistake, then cut any prose rule down to a pointer at it. In practice:

- Fix the root cause, migrate the callers and delete the old way in the same PR. Never patch a symptom with a timer, a retry or a sleep that hides an architecture clash.
- When two fixes resting on one premise have failed the same check, write that premise down and prove it directly, with a loop or a probe, before trying a third.
- An API that takes nuance to use well (React `useEffect`, imperative timers, raw threading) is banned in product code by a lint, behind one sanctioned wrapper.
- A boundary the architecture relies on (main vs renderer, UI vs engine) is a directory, and an import across it fails a lint or dependency check.
- A new lint's error names the file, type or function to use instead. Where the pattern is already common, fail only on new occurrences, then clean up the old ones.

## Lean, AI-forward repos

Every repo holds only what it needs, laid out so an agent finds the one right example first. Lean never costs quality: a check, a loop, or a doc that an agent needs stays.

- **The root** holds only:
  - what the toolchain requires;
  - a short, current README;
  - CONTRIBUTING, CHANGELOG and licence notices;
  - the dot-configs a tool really reads.

  Every other doc goes in `docs/`.
- **No CLAUDE.md, AGENTS.md or GEMINI.md.** CONTRIBUTING and the files that enforce it say how to change, test, push and merge the repo. An agent reads those first.
- **Each platform's app sits under one clear parent**, such as `mobile/android` and `mobile/ipad`. Platforms are never scattered across the root.
- **A committed script stays only if something calls it:** CI, a hook, the release, or CONTRIBUTING. Anything else is deleted.
- **Agent scratch is gitignored** (`samstack:git`).
- **The README has no ASCII art and no stale sections.** A section that no longer matches the code is fixed or cut in the PR that finds it.
- **Old code is replaced, not kept beside the new.** Where an area has an old way and a new way, the agent working there deletes the old one in a small PR of its own.
- **Before removing or moving anything,** find everything that uses it: imports, scripts, CI, hooks, docs and links. Update each one, then prove the build or loop still runs.

## Observability

Every error and caught exception, every external/network/DB/process call, every outcome-affecting branch, and every significant state change is logged with structured context (timestamps, identifiers, outcomes), at the right level:

| Level | For |
|---|---|
| DEBUG | flow |
| INFO | operations |
| WARN | recoverable |
| ERROR | failures |

Structured (JSON or key-value) where the surface allows it. Entry/exit tracing belongs at the service, request and CLI boundaries of long-lived deliverables — in a throwaway script, a trivial helper or a hot loop it buries the signal instead. Frontend logs errors and key user-flow events, never verbose render or lifecycle output.

**Secret, token, credential and PII values are redacted or omitted from every log.**

The goal: any issue diagnosable from logs alone.

**Logging vs auditing**: in-code logs diagnose the running program; an audit trail records who changed what, across sessions. Secrets and PII belong in neither, and audit entries reference targets by path, never by payload.

## Self-check before finalising

Run this internally and print only what it finds. Silence means it passed.

- Syntax validated with the right tool (`terraform validate`, `py_compile`, `jq .`).
- Logical consistency; unsafe defaults, destructive operations and secrets audited for.
- Idempotency confirmed, or the non-idempotency flagged.
- Environment and version assumptions named.
- The likely failure mode anticipated.

## Explaining the code

Explain only what the user can't read off the code: a non-obvious assumption, a surprising trade-off, a gotcha. A sentence or two, tied to the code.

## Lessons

Past mistakes in this area, one line each: [`lessons.md`](lessons.md). Grep it for the tool or symptom in front of you.
