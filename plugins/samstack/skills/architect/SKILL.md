---
name: architect
description: "Design before code: sketch the types, signatures and module boundaries, then build inside that shape. Use for 'design this' or 'how should this be structured', and before any build that adds a module, an interface, a schema or a flow across files, where the first shape that comes to mind would get locked in. A one-file or mechanical edit goes straight to work."
---

# Architect

Shape the change before you write it. Adapted from pstack's architect ([cursor/plugins](https://github.com/cursor/plugins)), MIT, Copyright (c) 2026 Lauren Tan: see `LICENSE`.

1. **Trace every system the change touches:** callers, stored data, other services, config and docs. Read them; a remembered picture of them is how a design misses a caller.
2. **Sketch two different shapes** as types, signatures and module boundaries, in stubs or a short outline. Two shapes differ in who owns a piece of state or where a seam falls, not in naming.
3. **Pick the shape where each piece of state has one owner, each task has one way to do it, and callers need to know least.** Write one line on why it beat the other. The line goes in the PR description or the ticket.
4. **Build inside the shape, and keep the sketch current as the code fills it in.** When the code fights a boundary, or the same workaround shows up twice, the shape is wrong: redesign from the boundary instead of patching around it.

What a good shape means, in types, boundaries and domain terms, is in `samstack:code-quality`. Prove the result with a loop (`samstack:testing`).

## When the design is bigger than a sketch

- **Contested or a one-way door,** and you can't rank two shapes without seeing them built: call the Skill tool with "samstack:arena". Its candidates are the shapes.
- **An ambitious multi-part change,** where the design is the whole playbook, not one decision inside it: call the Skill tool with "samstack:figure-it-out". It calls this skill for each one-way-door decision.
- **A shape that is already concrete,** such as a mechanical migration: skip this skill. A second design pass over a settled design is overbuilding.
