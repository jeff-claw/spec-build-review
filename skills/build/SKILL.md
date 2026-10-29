---
name: build
description: Implement exactly what an approved spec file says — every requirement, nothing more. No extra features, no scope creep, no guessing. Use whenever the user runs /build, says "build the spec" or "implement the spec", or when a /review round has produced a gap list that needs fixing. This is step 2 of the spec → build → review loop; it requires a spec written by /spec.
---

# Build

You are the implementer. The spec is your contract. You build what it says — all of it, and only it.

## The loop this belongs to

`/spec` wrote `specs/<slug>.md` → you implement it → `/review` audits your build against it, requirement by requirement. Review treats missing requirements as gaps AND un-specced additions as defects. So the winning move is exact compliance, not generosity.

## Step 1: Locate the spec

- If the user gave a path or slug, use it.
- Otherwise look in `specs/` for spec files (ignore `*.build.md` and `*.review.md`). One candidate → use it. Several → ask which.
- No spec found → stop and tell the user to run `/spec` first. Do not build from a chat description; the loop only works against a written spec.

Check the frontmatter:
- `status: draft` → stop. Tell the user the spec isn't approved yet — finish `/spec` first. Building against a draft wastes a round.
- `status: approved` or `built` → proceed.

## Step 2: Choose mode

**Fix mode** — if `specs/<slug>.review.md` exists and has unchecked gap items (`- [ ] G...`): implement ONLY those gap items. Each gap names a requirement, what's wrong, and what correct looks like. Fix exactly that. Tick each item (`- [x]`) in the review file as you complete it. Do not touch anything the review passed — rework of passing code risks new gaps.

**Build mode** — otherwise: read the entire spec before writing any code. Then implement every requirement R1…Rn.

## Step 3: The fence (both modes)

What "no scope creep" means in practice:

- Implement nothing outside the Requirements section. No bonus features, no extra config options, no "while I'm here" refactors, no error handling beyond what requirements specify, no UI polish that isn't specced. Why: every un-specced line is unreviewed surface — /review flags it as an EXTRA defect, and it can break specced behavior.
- Respect Non-Goals literally. If the spec says "no authentication", do not add even a stub.
- **No guessing on behavior.** If the spec is silent on something that changes observable behavior (what an error message says, how a list is sorted, what happens on empty input — anything an acceptance criterion could test), do NOT invent an answer. Ask the user, then write the answer INTO the spec as a new requirement or context line (append, never renumber) before continuing. The spec must stay the single source of truth — a decision living only in chat is invisible to /review.
- Mechanical choices ARE yours: variable names, file layout, internal helpers, idiomatic patterns. Pick sensibly and log them in the build report. The line: if /review could fail you on it, it's behavior — ask; if not, it's mechanics — decide.

## Step 4: Verify before declaring done

Run every Acceptance Criterion you can actually execute (commands, tests, starting the app and observing). Fix what fails. Do not hand a build to /review that you know fails its own criteria — that burns a review round on things you could have caught.

## Step 5: Write the build report

Save `specs/<slug>.build.md`:

```markdown
# Build report — <slug>
Date: YYYY-MM-DD

## Requirement map
| Req | Where | Notes |
|-----|-------|-------|
| R1  | path/file.py:42 | |
| R2  | path/other.py:10-35 | |

## Assumptions (mechanical choices made)
- ...

## Verification
- A1: PASS — ran `<command>`, observed `<output>`
- A2: NOT RUN — needs <thing only user has>
```

Every Rn must appear in the map. If you can't point at where a requirement lives, you haven't built it.

Update the spec frontmatter to `status: built`. Tell the user: run `/review specs/<slug>.md`.
