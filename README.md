# Ink and Agency Skill Pack

An opinionated library of reusable agent skills.

The pack is built to work well with Claude-compatible skill loading, but most of the content is deliberately plain Markdown and easy to adapt to other prompt-driven agents. Each folder is a self-contained skill, so you can browse the pack like a catalog instead of reading a giant manual.

![Ink and Agency Skill Pack](_meta/banner.svg)

## What’s in the pack

- Practical skills for writing, planning, triage, architecture, and workspace workflows
- One `SKILL.md` entry point per skill folder
- Supporting docs where a skill needs examples, formats, or deeper guidance
- A structure that works directly with Claude and can usually be adapted to other agents with light changes

## Skill Map

| Area | Example skills | What they help with |
| --- | --- | --- |
| Writing | `writing-humanize`, `writing-draft-article`, `writing-shape` | Turning rough text into something clearer and more natural |
| Analysis | `break-it-down`, `codebase-explain`, `issue-triage` | Making sense of a codebase, analyzing messages, and diagnosing bugs faster |
| Collaboration | `clarity-council`, `grill-me`, `grill-with-docs` | Getting sharper decisions through structured discussion |
| Planning | `sprint-plan`, `sprint-review`, `daily-briefing` | Organizing work, progress, and reporting |
| Workspace tools | `obsidian-vault`, `obsidian-markdown`, `obsidian-canvas` | Managing notes, structure, and visual knowledge maps |

![Skill Map](_meta/skill-map.svg)

## Quick Start

For Claude, clone this repository into your skills directory:

```bash
git clone git@github.com:risadams/skills.git "$HOME/.claude/skills"
```

That’s it. Once the repo is in place, Claude can discover the skills directly from the folder structure.

If you are using another agent, the same folder layout is still useful, but the install location and invocation rules may differ.

## Check the install

After cloning, you should see paths like these:

- `$HOME/.claude/skills/writing-humanize/SKILL.md`
- `$HOME/.claude/skills/clarity-council/SKILL.md`
- `$HOME/.claude/skills/codebase-explain/SKILL.md`

## Using skills

There is no separate run command.

In Claude, skills are triggered in chat by asking for the behavior you want. A few examples:

- Use `writing-humanize` on this paragraph.
- Use `codebase-explain` for this module.
- Triage `PROJ-1234`.
- Run a `clarity-council` on this design.

In other agents, the same prompts often still work, but you may need to wire the skill files into that agent's own prompt, skill, or tool-loading system.

## Compatibility

This pack is Claude-compatible first, but it is not Claude-only.

Most skills in this repo are prompt assets with a stable folder structure, so they usually port cleanly to other agents. In practice, you may need to adjust:

- Install location, because other agents may not read from `.claude/skills`
- Frontmatter fields, if another agent expects different metadata keys
- Tool names or allowed-tool lists, if the target agent exposes a different tool model
- Invocation patterns, if the target agent uses commands, slash prompts, or manifests instead of folder discovery

If an agent can read Markdown prompt files and route on instructions, the content itself usually needs little or no rewriting. The main work is usually integration, not rewriting the skill.

## Browse the catalog

The full inventory lives in **[CLAUDE.md](CLAUDE.md#skills-inventory)**. That file is the source of truth for the skill list and descriptions, which keeps this README short and prevents drift.

## Release notes

- **v1.0** (2026-05-18)
  - Initial public release of the skills pack

## Contributing notes

This is an unsupported personal project. Fork freely, but no PRs are accepted.

For project conventions, naming schemes, and layout rules, see **[CONTRIBUTING.md](CONTRIBUTING.md)**.
