# samstack

Coding principles for Claude Code, packaged as one plugin: stability, verification loops and kits, git, security, code quality, parallel work, design and fan-out, state-machine checking, catch-ups, writing, TypeScript, measuring, and reframing and sharpening a loose ask. Each skill states the principle, and its `lessons.md`, where it has one, holds the mistakes that taught it.

The way of working behind it: a change is proven by running the real thing on made-up data, not by writing tests (`samstack:testing`); a PR merges once its pre-PR hook has passed, with no CI gate in front of it; a red `main` is reverted at once; a correction becomes a check, not a paragraph. Take what fits and cut what doesn't: the skills are plain Markdown.

| Path | What it is |
|---|---|
| `.claude-plugin/marketplace.json` | The marketplace, listing one plugin |
| `plugins/samstack/.claude-plugin/plugin.json` | The plugin |
| `plugins/samstack/skills/<name>/SKILL.md` | One skill, with its `references/` and `lessons.md` beside it |
| `scripts/check` | What the pre-push hook runs |
| `scripts/public-check` | The owner's gate before publishing: no private names in any file or any commit |
| `LICENSE` | MIT |
| `NOTICE` | Credit for the adapted skills |

Skills load as `samstack:<name>`, for example `samstack:stability`.

## When to reach for which

Their descriptions carry these triggers, so Claude picks them up on its own. Say the name to force one.

| Situation | Skill |
|---|---|
| A build adds a module, an interface or a schema, and the first shape that comes to mind would stick | `architect` |
| Two shapes you can't rank without seeing them built, or a one-way door | `arena` |
| A large migration or an ambitious multi-part change with no narrower playbook, or work reviewed after stepping away | `figure-it-out` |
| The same edit or check across many files, repos or cases | `swarm` |
| A High change (auth, billing, secrets, signing, CI and hooks, migrations, crypto) needs its adversarial pass, or "tear this apart" | `interrogate` |
| A flow with retries, races or a protocol, or an ordering bug that shipped twice | `statechart` |
| Coming back to a long or multi-agent session: "catch me up", "what's waiting on me" | `ketchup` (`ketchup-video` for a slide video) |

`statechart` ships its own scripts: they need Node 22.18 or later (or Bun or Deno), and TLC with Java, from `mise` or from `TLA2TOOLS` and `JAVA`.

## Install

```
claude plugin marketplace add samuelbailey123/samstack
claude plugin install samstack@samstack
```

Both manifests leave out `version`, so every commit to `main` is a new version, and `claude plugin update samstack@samstack` picks it up.

To work on the skills, add the marketplace from your clone instead (`claude plugin marketplace add <path to the clone>`). A marketplace added from a local folder loads its files fresh every session, so a merged change is live with no update step.

## Where it comes from

These are one developer's working rules, so some read as opinionated: an agent does the whole job without stopping to ask, and the checks that gate a change run on the author's machine before the PR. [CONTRIBUTING.md](CONTRIBUTING.md) says how to change them.

`reframe`, `sharpen`, `writing`, `typescript`, `measuring`, `verification-kit`, `architect`, `arena`, `swarm`, `interrogate`, `figure-it-out`, `statechart`, `ketchup` and `ketchup-video` are adapted from [pstack](https://github.com/cursor/plugins) by Lauren Tan (MIT, licence kept beside each skill), and points from other pstack skills are folded into `code-quality`, `parallelism`, `stability` and `testing`. [NOTICE](NOTICE) lists each one. The repo layout follows [pstack-claude](https://github.com/michael-denyer/pstack-claude).

MIT licensed: see [LICENSE](LICENSE) and [NOTICE](NOTICE).
