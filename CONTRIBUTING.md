# Contributing

## Where to work

Claude loads these skills straight from a local clone added as the marketplace, so keep that clone on `main`. Make every change in a worktree on its own branch, then open a pull request.

## A skill

- A skill is `plugins/samstack/skills/<name>/SKILL.md`. Its frontmatter `name` is the folder's name.
- Write for the agent that will read it: say what to do and why in plain sentences, put the rule where it is needed, and cut what a model would do anyway.
- A skill's `lessons.md` holds the general coding how-to learned in use, one line per lesson. Add a lesson by pull request like any other change.
- samstack holds principles anyone could use. A rule or lesson tied to one machine, org, client or product goes in a private skill instead.
- A skill leans on nothing the reader may not have: another plugin's skills, agents under `~/.claude/` and a tracker are all optional, and the text says what to do without them.
- Leave `version` out of both manifests: each commit is a version.

## Checks

Link the pre-push hook once per clone. Worktrees share it.

```
ln -sf ../../.githooks/pre-push .git/hooks/pre-push
```

The hook runs `scripts/check` in about 2 seconds. The check covers:

- every skill's frontmatter;
- every relative link in the skills' Markdown;
- `claude plugin validate` on the marketplace and on the plugin;
- gitleaks scans of the working tree and of every commit.

There is no CI workflow: this repo is Markdown and two scripts, the hook already runs the whole check, and a merge that names the head the hook passed (`gh pr merge --match-head-commit`) lands exactly the tree that was checked, so `main` cannot go red between the hook and the merge. A pull request merges once its hook passes. Anyone who adds a workflow should make it run `scripts/check` on pushes to `main`.

## Commits

Use the GitHub noreply address (`<id>+<name>@users.noreply.github.com`), so no private address lands in the history.

## Publishing

`scripts/public-check` is the owner's gate before a repo goes public. It scans every file that would ship and every commit on every branch and tag. Besides its generic patterns (home paths, tailnet addresses, any email address that is not a noreply one), it reads private patterns from `~/.config/samstack/public-check-patterns`, or from the path in `SAMSTACK_PUBLIC_CHECK_PATTERNS`: one `label<TAB>regex<TAB>sample` per line, where the sample proves the regex still matches. Without that file it refuses to run, and `--generic-only` runs the generic set alone. Contributors don't need it.
