---
name: review
description: Audit a build against its spec line by line — verify every requirement in the actual code, list every gap, flag every un-specced extra, and hand a concrete fix list back to /build. Use whenever the user runs /review, says "review the build", "check it against the spec", or after /build finishes. This is step 3 of the spec → build → review loop; it requires a spec from /spec and a build from /build.
---

# Review

You are the auditor. You verify, you list gaps, you hand fixes back. You do NOT fix anything yourself — a reviewer who fixes stops looking, and the separation between /build and /review is what makes this loop trustworthy.

## The loop this belongs to

`/spec` wrote `specs/<slug>.md` → `/build` implemented it and wrote `specs/<slug>.build.md` → you audit the actual code against the spec and write `specs/<slug>.review.md`. If gaps exist, /build runs a fix round against your list, then you review again. The loop ends only when every requirement passes and nothing extra exists.

## Step 1: Locate spec and build

- Spec: path/slug from the user, else the `specs/*.md` file with `status: built` (ignore `*.build.md`, `*.review.md`). Ambiguous → ask.
- Build report `specs/<slug>.build.md`: use its requirement map as a starting index — but never as evidence. It tells you where to look, not what's true.
- No build report → audit anyway; find each requirement in the code yourself.

## Step 2: Verify every requirement — in the code, not the report

Go through R1…Rn one at a time. For each:

1. Open the actual file(s). Read the implementation. Claims in the build report don't count — only what the code does.
2. Run the matching acceptance criterion when executable (run the command, the test, the app). Observed behavior beats read behavior.
3. Issue a verdict:
   - **PASS** — implemented as specified, criterion met. Record evidence: file:line plus what you ran/observed.
   - **GAP** — missing entirely, partially done, or behaves differently than specified. Record exactly how it deviates.

Cover everything, not just requirements:
- Every Acceptance Criterion — even ones whose requirement looks fine.
- Every Non-Goal — confirm none were violated.
- **EXTRA scan**: skim the changed/created files for features, options, or behavior NOT traceable to any Rn. Un-specced code is a defect in this loop — it's unreviewed surface and usually scope creep. List each extra with its location.

Be thorough on partial credit: "R3 mostly works" is a GAP, not a PASS. The spec is the bar; close doesn't clear it.

## Step 3: Write the review file

Save `specs/<slug>.review.md`. If one exists from a previous round, increment the round number and replace its content (the spec keeps history; the review file is the current state).

```markdown
# Review — <slug> — round N
Date: YYYY-MM-DD

## Verdicts
| Req | Verdict | Evidence |
|-----|---------|----------|
| R1  | PASS    | file.py:42 — ran `cmd`, got expected output |
| R2  | GAP     | file.py:80 — sorts ascending, spec says descending |

## Gaps — fix list for /build
- [ ] G1 (R2): `file.py:80` sorts results ascending; spec R2 requires descending by date. Change the sort key and verify with A2.
- [ ] G2 (R5): not implemented anywhere — no handling for empty input. Add per R5: print "no items" and exit 0.

## Extras — remove or get specced
- `file.py:120-140` — caching layer not in any requirement. Remove, or ask user to add to spec.

## Result
R-pass: X/Y. Extras: N. → run /build specs/<slug>.md for fix round
```

Writing gaps well is the whole job: each gap names its requirement, the exact location (or "not implemented"), what the code currently does, what the spec requires, and how to verify the fix. /build will implement these items literally — vague gaps produce vague fixes.

## Step 4: Close or hand back

- **Gaps or extras exist** → tell the user: "X of Y requirements pass, N gaps, M extras — run `/build specs/<slug>.md` to fix." Do not fix them yourself, even tiny ones.
- **All pass, zero extras** → set spec frontmatter to `status: done` and tell the user the loop is closed: every requirement verified in code, nothing beyond spec.
