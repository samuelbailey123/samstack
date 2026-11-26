---
name: reframe
description: Reframe a loose ask in the expert's words. You MUST call this before any other tool or reply when the user describes a want in plain or unsure words ("I'm not sure how"), names a symptom or a feeling instead of a goal ("seems suspicious", "taking way too long"), or could mean more than one problem. A clear ask goes straight to work, and so does one you can answer outright.
---

# Reframe

Find the expert words for what the user is asking for, borrow words from neighbouring fields that solved the same problem, then rewrite the ask with them. Adapted from pstack's reframe ([cursor/plugins](https://github.com/cursor/plugins)), MIT, Copyright (c) 2026 Lauren Tan: see `LICENSE`.

When the user typed `/samstack:reframe`, give them the whole method's output. Otherwise the reframe is your own brief, and the user sees one line of it: `Reading this as: <the ask, rewritten in the expert words>`.

## The method

Start from the user's own files, code and words, not a summary.

If the ask could name more than one problem, first list the two or three problems it could be, each with the one term that names it and the sign that tells it apart from the others. Then work the most likely one, and say why you picked it.

1. Give 8 to 12 terms, ranked by how much each would change what gets built. For each, give the word and its meaning in one plain sentence, the part of the ask it names, one real project, paper or system that uses it that way, with a link, and what it lets you ask an agent for.
2. Pick 2 or 3 neighbouring fields that solved the same shape of problem under other names. For each, say what carries over, where the analogy breaks, and which terms come back.
3. Name what the ask didn't mention but should care about.
4. Rewrite the ask with the new words.

Open every link and confirm it uses the term that way. Cut any citation you can't confirm. No padding.

## When it fired on its own

1. Work the method for yourself, from the user's words and what is already in context. The rewritten ask, with the gaps from step 3, is the brief you work from.
2. Call the Skill tool with `samstack:sharpen` on the rewritten ask.
3. Before any other tool call, write the user one line: `Reading this as: <the sharpened ask, in the expert words>`. It is the one line for both skills. When the candidate problems lead to different results, write one question instead, your pick first, so "yes" answers it.
4. Do the work from the brief, reading their files as the method says. The term list, its links opened and confirmed, goes in the work's ticket or note, or in a spec kept out of the repo.
5. Your final reply opens with the answer. The `Reading this as:` line is the one you wrote before the first tool call; do not repeat it.
