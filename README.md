# Claude Skills Pack

This repository contains a collection of Claude-compatible skills.

## Quick Setup

Clone this repository directly into your Claude skills directory:

```bash
git clone git@github.com:risadams/skills.git "$HOME/.claude/skills"
```

## Verify Installation

After cloning, you should have folders like these:

- `$HOME/.claude/skills/writing-humanize/SKILL.md`
- `$HOME/.claude/skills/clarity-council/SKILL.md`
- `$HOME/.claude/skills/codebase-explain/SKILL.md`

## How To Run The Skills

There is no separate "run" command.

Skills are used by your coding agent during chat. Trigger them by asking for the relevant behavior directly, for example:

- "Use writing-humanize on this paragraph"
- "Use codebase-explain for this module"
- "Triage PROJ-1234"
- "Run a clarity-council on this design"

## Skills Map

See the **[Skills inventory in CLAUDE.md](CLAUDE.md#skills-inventory)** for the full list and descriptions. That file is the single source of truth — keeping the inventory in one place avoids drift.

## Releases

- **v1.0** (2026-04-30) — Initial skill pack with 18 skills

## Notes

- This is an unsupported personal project. Fork freely, but no PRs accepted.
- Keep each skill folder name stable (for example: `writing-humanize`, `diagnose`, `clarity-council`).
- Each skill should keep `SKILL.md` at its folder root.
- Nested docs can stay inside each skill folder as supporting material.
