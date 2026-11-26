---
name: measuring
description: "Vet a measurement before you report or act on it: a speedup, regression, throughput, latency or eval result. Find what limits it, and rule out that it measured something other than the work. Use when you run a benchmark, report a number you measured, or choose between options on one."
---

# Measuring

A number is a claim about the work. Before you report or act on one, find what limits it and rule out that it measured something else: errors, skipped or cached work, code that never ran, an untuned side, noise, or a piece too small to matter. Answer each question with evidence from a run, not a guess from reading the code.

For a ballpark the user asked for, one run is enough. Still check questions 4 and 7, and say it is one run. Skip the rest unless that run looks wrong. A choice between options is never a ballpark.

## Before you run

- Write the claim you expect to ship in the words you would ship it ("export is 30% faster at p50 on the 60k-row dataset"), the result that would change your decision, and the noise floor. Stop when the run-to-run spread is larger than the effect. The questions test that sentence.
- Read the measurement script. Note what it times, counts and ignores.
- Check the load with `uptime` and the core count with `nproc`. If the machine is busy, find what is running. If you cannot stop it, interleave the sides so both see the same noise, and say so in the report.

## The questions

1. **Why not double?** Name the limiter. Profile in a run you do not report, because profilers slow the work down. Use CPU per process (`top`, `pidstat`), a runtime profiler (`node --cpu-prof`, `py-spy`, `perf`), I/O wait and syscall counts (`strace -c` on Linux), then map the hot spot to source. Watch the load generator too: if it saturates first, you measured the load generator. When a change did not move the number, the limiter says why, so find it before you call the change useless.
2. **Was it tuned?** Run every side the way production runs it: release builds, production flags and environment, batching and transaction settings, connection pools, caches as warm or cold as production sees them, and the same versions and data. A side on defaults means you compared configurations, not implementations. A limiter that is a setting (a commit per row, a debug build, a missing index) means that side is untuned: tune it and measure again before you pick a winner. If you cannot tune it, pick no winner from that run. Narrowing the claim to the code as it ships today does not help when the user is choosing what to adopt, because they adopt the option, not today's settings.
3. **Did it break limits?** Do the arithmetic. Compare bytes per second with disk and network bandwidth, operations per second times cost per operation with your cores, and the time saved with the time the changed piece took. Removing a piece that takes 10% of the run makes the run at most about 11% faster. A result past a limit measured something other than the work: a cache, a no-op or a bug.
4. **Did it error?** Count failures and non-success responses, and check that outputs are correct, not just present. Rejections are often fast, and timeouts and retries are slow. If the script does not count errors, add the count.
5. **Does it reproduce?** Run each side at least 5 times, alternating (A, B, A, B, and so on) so warmup, lazy initialisation, caches and drift favour neither. Report the median and range. A gap smaller than the run-to-run variation is no measurable difference. When the call is close, use a rank-sum test or the harness's own statistics.
6. **Does it matter?** Next to any micro result, measure the end-to-end path a user waits on, with realistic data sizes and concurrency, and report the micro result as a share of the whole. A helper that takes 1% of a request makes the request at most 1% faster, however fast the helper gets.
7. **Did it even happen?** Confirm the work ran inside the timed region: the request reached the server, the rows were written, the bytes were read, and the code used the result. Lazy code (generators nobody iterates, promises nobody awaits, results the JIT can discard) and timeouts produce numbers for work that never happened.

For an eval result, ask whether every run did the task, whether the gap holds across trials and models, and whether the scenario matters.

## Report

- Lead with the verdict: faster, slower, no measurable difference, or inconclusive.
- Give the number with its unit, the run count, the range and the limiter. For example: "p50 41 ms to 33 ms, median of 7 runs per side, range 32 to 35 ms after, bound by JSON parsing on one core."
- Call the verdict inconclusive when you claim a difference but cannot name the limiter, when a side ran untuned, or when you could not check questions 4 and 7. Name the gap.
- Keep a PR body to one primary number. Put the runs, range and limiter evidence in a linked artifact.

Adapted from pstack's benchmark-checklist and explain-the-number skills ([cursor/plugins](https://github.com/cursor/plugins)), MIT, Copyright (c) 2026 Lauren Tan: see `LICENSE`.
