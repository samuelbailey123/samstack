# samstack

Coding principles for Claude Code, packaged as one plugin: stability, verification loops, git, security, code quality, parallel work, and reframing and sharpening a loose ask. Each skill states the principle, and its `lessons.md`, where it has one, holds the mistakes that taught it.

The way of working behind it: a change is proven by running the real thing on made-up data, not by writing tests (`samstack:testing`); a PR merges once its pre-PR hook has passed, with no CI gate in front of it; a red `main` is reverted at once; a correction becomes a check, not a paragraph. Take what fits and cut what doesn't: the skills are plain Markdown.

| Path | What it is |
|---|---|
| `.claude-plugin/marketplace.json` | The marketplace, listing one plugin |
| `plugins/samstack/.claude-plugin/plugin.json` | The plugin |
| `plugins/samstack/skills/<name>/SKILL.md` | One skill, with its `references/` and `lessons.md` beside it |
| `scripts/check` | What the pre-push hook runs |
| `scripts/public-check` | The owner's gate before publishing: no private names in any file or any commit |
| `LICENSE` | MIT |

Skills load as `samstack:<name>`, for example `samstack:stability`.

## Install

```
claude plugin marketplace add samuelbailey123/samstack
claude plugin install samstack@samstack
```

Both manifests leave out `version`, so every commit to `main` is a new version, and `claude plugin update samstack@samstack` picks it up.

To work on the skills, add the marketplace from your clone instead (`claude plugin marketplace add <path to the clone>`). A marketplace added from a local folder loads its files fresh every session, so a merged change is live with no update step.

## Where it comes from

These are one developer's working rules, so some read as opinionated: an agent does the whole job without stopping to ask, and the checks that gate a change run on the author's machine before the PR. [CONTRIBUTING.md](CONTRIBUTING.md) says how to change them.

`reframe` and `sharpen` are adapted from [pstack](https://github.com/cursor/plugins) by Lauren Tan (MIT, licence kept beside each skill). The repo layout follows [pstack-claude](https://github.com/michael-denyer/pstack-claude).

MIT licensed: see [LICENSE](LICENSE).
