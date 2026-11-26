---
name: writing
description: "Write docs, READMEs, RFCs, PR descriptions and commit messages that a tired engineer follows on the first read, and cut the tells of machine-written prose from any text. Use when writing or reviewing prose a human will read, or when asked to tighten a draft."
---

# Writing

Write so a tired engineer understands on the first read. Lead with the point: the answer or the decision first, then the reason. When editing, keep the meaning and match the tone the text is meant to have.

- Cut every word that does no work. Use the plain word: "use", not "utilize" or "leverage"; "to", not "in order to".
- Write the real symbol, file, flag or command, not a synonym or a description of it. Call each thing by one name and repeat it.
- Say what it does, not how it feels: name the mechanism or the number. "A column rename fails the build", not "schema changes can cause issues". A sentence that could appear unchanged in another project's docs goes.
- Use the words a developer would say aloud, and say what a named pattern means the first time.
- When a rule here makes a sentence worse, fix the sentence another way.

## Pick the mode first

One document, one mode (Diátaxis). Split and link instead of mixing.

- **Tutorial**: learning by doing. Say what the learner will build. Every step produces a visible result, and says what to expect. Explanation gets one clause and a link.
- **How-to**: steps to a goal, for a reader who is competent. Action only, forks allowed, named by the task.
- **Reference**: facts for lookup. Dry, complete and sure, in the same structure as the thing it describes.
- **Explanation**: understanding and why, for one bounded topic: decisions, history, constraints, alternatives. Opinion belongs here and nowhere else.

## Sentences

- Talk to the reader as "you", in the present tense and the active voice. Name the actor unless it is unknown or beside the point.
- Write instructions as commands, one per sentence, the common case first. Put a condition or warning before the step it guards.
- Give every other sentence one thought. Split an instruction longer than about 20 words and any other sentence longer than about 25, and any sentence the reader has to reread. Mix the lengths.
- Keep the small words ("the", "a", "that") and spell out arrows and abbreviations.
- Keep "only" and "not" next to the word they change. Make every "it", "they" and "this" point at one obvious thing, never at a whole clause. Break up long noun strings.
- Give each word one meaning and each action one word. Skip idioms, Latin abbreviations, metaphor, aphorism and personified code.
- Skip "please", and "simply", "easy" and "quickly" in a procedure. Cut adverbs or use a stronger verb, and give the number. Hedge once at most, with "may".

## Tells to cut

Machine-written prose gives itself away. Scan for these and rewrite.

- Trailing "-ing" phrases that add no fact ("highlighting...", "ensuring..."). Vague attributions ("experts believe"): name the source or delete.
- AI vocabulary: additionally, crucial, delve, enhance, foster, landscape, pivotal, showcase, testament, underscore, vibrant. Abstract metaphor nouns: substrate, wedge, vector, nexus, primitive, harness, surface, scaffolding, paradigm, ratchet, north star, flywheel. Use the concrete word.
- "Serves as", "stands as", "boasts" and "features" where "is" or "has" works.
- "Not just X, but Y": state the point. Ideas forced into threes. "From X to Y" where X and Y sit on no real scale.
- A generic conclusion: state the specific plan or fact.
- Chatbot phrases ("I hope this helps!", "Let me know if...") and praise or agreement openers: answer directly.
- Formatting tells:
  - bold on every proper noun or acronym;
  - a bold label and colon that restates its line ("**Performance:** Performance improved..."), where a bold lead-in that ends in a period and adds detail is fine;
  - decorative emoji and curly quotes;
  - a colon as a mid-sentence connector, where a colon goes before a list or an example.
- Headings in sentence case that carry the point, one h1 per page, no skipped levels. Numbered lists for sequences and bullets for the rest, each introduced by a complete sentence, with parallel items. Code in code font, UI elements in bold, serial commas, no "etc.". Link text says where the link goes.

## Commits, PRs and docs

- A PR body is a briefing a reviewer reads in under a minute: what changed, why, and the evidence it rests on. Link logs, SHA lists and metric tables instead of pasting them. A commit message follows the same rules, except the mode.
- Write real paths. Make every count or tree claim true at the commit that lands it, and include the command that regenerates it.
- Product UI strings are not documentation. Use the product's own copy guidelines for those.

Adapted from pstack's unslop and technical-writing skills ([cursor/plugins](https://github.com/cursor/plugins)), MIT, Copyright (c) 2026 Lauren Tan: see `LICENSE`.
