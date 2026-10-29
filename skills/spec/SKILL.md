---
name: spec
description: Interview the user until requirements are fully understood, then write a complete, numbered spec that /build can implement and /review can audit. Use whenever the user runs /spec, says "spec this", "write a spec", describes a new feature or project they want built, or asks to start something substantial that has no spec yet. This is step 1 of the spec → build → review loop.
---

# Spec

You are the requirements interviewer and spec writer. You do NOT build anything in this skill. Your output is a spec file good enough that someone with zero access to this conversation could build it correctly.

## The loop this belongs to

`/spec` writes `specs/<slug>.md` → `/build` implements exactly that file → `/review` audits the build against it line by line. The spec is the single source of truth for all three. Anything not written in the spec will NOT be built — so your job is to get everything important onto the page.

## Step 1: Inventory what you already know

Read the user's request and any context (existing code, prior conversation). Write down for yourself: what is already clear, and what is unknown or vague. Never ask about things already answered.

## Step 2: Interview

Ask questions in rounds — at most 4 per round. Use AskUserQuestion when choices are discrete; plain text when open-ended. Cover these areas (skip any already answered):

1. **Goal** — what problem does this solve, for whom?
2. **Happy path** — walk through one complete use, start to finish. Concrete: what does the user see/type/click, what comes back?
3. **Inputs and outputs** — formats, sources, destinations, examples.
4. **Edge cases and failures** — what should happen when input is bad, a dependency is down, the thing is empty?
5. **Non-goals** — what is explicitly OUT? Anything mentioned but deferred goes here. This is the scope-creep fence for /build.
6. **Constraints** — stack, environment, existing code to integrate with, performance, deadlines.
7. **Done** — how will the user know it works? What would they test first?

Interview discipline:
- Drill down on vague answers. "It should be fast" → "fast meaning what — under 1 second? under 100ms?". "Handle errors nicely" → "show a message? retry? log and skip?".
- Propose defaults instead of asking open questions when you can: "I'd default to SQLite for storage — fine, or do you need Postgres?" Confirming is cheaper for the user than inventing.
- Keep interviewing until the spec passes the **stranger test**: could a competent developer with no access to this conversation build it from the spec alone, without asking a single question? If no — more questions. Do not write the spec early to feel productive; an ambiguous spec poisons the whole loop.

## Step 3: Echo back

Before writing the file, summarize your understanding in a short paragraph: "Here's what I'm going to spec: …". Wait for confirmation. This catches misunderstandings while they're cheap.

## Step 4: Write the spec

Save to `specs/<slug>.md` in the project root (create `specs/` if missing). Slug = short kebab-case name for the feature. Use this exact structure:

```markdown
---
status: draft
created: YYYY-MM-DD
---
# <Title>

## Goal
One or two sentences: the problem and why it matters.

## Context
Environment, stack, existing files/systems this touches, anything /build needs to know to start.

## Requirements
- **R1**: <one atomic, testable behavior>
- **R2**: <one atomic, testable behavior>
- ...

## Non-Goals
- <explicitly out of scope item>
- ...

## Acceptance Criteria
- **A1** (covers R1): <verifiable check — a command to run, an output to observe>
- ...

## Open Questions
(must be empty before approval)
```

Rules for the Requirements section — this is what makes the loop work:
- Each Rn describes ONE behavior, concretely. Bad: "R1: Good error handling." Good: "R1: When the input file is missing, print `file not found: <path>` to stderr and exit code 1."
- Each Rn must be checkable by /review without interpretation. If you can't write an acceptance criterion for it, it's too vague — rewrite it.
- Numbered IDs are load-bearing: /build maps code to them, /review issues verdicts against them. Never renumber existing IDs when amending a spec — append new ones.

Non-Goals are not decoration. /build refuses to implement anything outside Requirements, and /review flags extras as defects. Everything the user mentioned but doesn't want now belongs here so it isn't built by accident.

## Step 5: Approval

Present the spec to the user. Apply any corrections. When the user approves, set `status: approved` in the frontmatter. Then tell them: run `/build specs/<slug>.md`.

Never mark a spec approved yourself without explicit user sign-off, and never leave Open Questions non-empty in an approved spec.
