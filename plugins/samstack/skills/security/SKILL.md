---
name: security
description: The security audit gate and its trigger surfaces. Use when a diff touches auth, secrets, storage, uploads, headers, CORS, raw SQL or user-controlled URLs, or before a destructive operation.
---

# Security

## The gate

**Every change that touches a security-relevant surface gets a dedicated security pass after it merges, as the last step of its own work.** It is not a PR gate: the pre-PR hook and the change's loops decide when a PR merges (`samstack:testing`), and the pass follows on `main`. The pass exists because security debt slips in where nobody thought to look.

### Trigger surfaces

- **Data exposure / storage** — buckets and CDN (public vs private, signed URLs), file uploads, publicly-fetchable URLs, PII in responses or logs, `NEXT_PUBLIC_` and other client-shipped values.
- **AuthN / AuthZ** — new or changed routes and handlers, ownership and permission checks (IDOR), admin gates, session/cookie/JWT config, proxy and trust boundaries, cron and webhook secrets.
- **Secrets & credentials** — token, key and password handling, env wiring, signing and compare logic.
- **Money and trust** — billing, signing and updaters, crypto, and migrations of stored data.
- **Pipelines** — CI and hooks, which run with the repo's credentials and decide what ships.
- **Injection** — raw SQL (`$queryRawUnsafe`), `dangerouslySetInnerHTML`, user-controlled URLs reaching `fetch` (SSRF), deserialization, shell calls.
- **Security config** — CSP and headers, CORS, cookie flags, rate limiting, dependency bumps carrying known CVEs.

### How to satisfy it

- **Size floor.** A diff under ~50 lines touching none of the surfaces above gets an inline self-review against this file, noted explicitly. A diff that changes a trigger surface gets an independent reviewer: a security-review subagent or command (a `security-auditor` agent, if you keep one). A substantial or cross-cutting change gets an adversarially-verified audit: multi-lens scan, then refute-by-default verification, not a rubber stamp. Call the Skill tool with "samstack:interrogate" for it: adversaries try to break the change, and only findings a run proves survive.
- **Brief the auditor on designed behaviour** — what the system deliberately collects and stores — so intended behaviour isn't reported as a leak. A finding names concrete exposure; it does not re-litigate a design decision.
- **A finding's proof is a verification loop** on a scratch home that shows the exposure before the fix and its absence after, with made-up data and throwaway keys (`samstack:testing`, "Core loops on a scratch home").
- **Findings are fixed in the work that finds them.** Anything severe is fixed before anything else ships. The rest go into that same work's PRs, or are dropped: a finding is never filed for later.
- **Pre-existing debt counts.** A weakness you find while working is a finding the moment you see it, even though you didn't introduce it.

## Credentials

Secrets live in env vars, a secret manager (Vault, AWS, GCP), or properly-permissioned config. Recommend secure handling by default, unprompted.

Keep `$GITHUB_TOKEN` and every other credential as an unexpanded variable in commands. Never echo, interpolate, log, or write a credential value anywhere — and never into notes, tickets or a git-backed knowledge base, where every write is permanent history.

Validate, sanitise and type-check inputs before processing. File and process operations follow least privilege and verify existence and permissions first. Log sensitive operations, never sensitive values.

## Operational safety

**Act as the last safety gate.** Assume no permission prompt will stop a bad command.

Before anything destructive or irreversible: explain what it will do, prefer the dry run (`terraform plan`, `kubectl diff`, `--dry-run`), verify preconditions, and get explicit confirmation. Look at the target before deleting or overwriting it.

Availability-degrading operations (restarts, shared-config changes, deploys) are governed by `samstack:stability`.

## Lessons

Past mistakes in this area, one line each: [`lessons.md`](lessons.md). Grep it for the tool or symptom in front of you.
