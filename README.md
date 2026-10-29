# spec → build → review

Three Claude Code skills that work as one loop:

1. **`/spec`** — interviews you until the requirements are clear, then writes a numbered spec to `specs/<slug>.md`.
2. **`/build`** — implements exactly what the approved spec says. Nothing more.
3. **`/review`** — audits the build against the spec line by line and hands a fix list back to `/build`.

The spec file is the single source of truth for all three steps.

## Install

Copy the three folders into your Claude Code skills directory:

```bash
git clone https://github.com/jeff-claw/spec-build-review.git
cp -R spec-build-review/skills/* ~/.claude/skills/
```

Restart Claude Code. Then run `/spec <what you want built>`.

## Usage

```
/spec a CLI that converts CSV files to JSON
/build specs/csv-to-json.md
/review specs/csv-to-json.md
```

## Origin

The original author of these skills is unknown. This repo shares copies as they were found in a local Claude Code setup. If you wrote them, open an issue and credit will be added.
