# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Nature of this repo

This is a **Claude Skills Pack** — a collection of prompt-based skill definitions. Each skill is a markdown file (`SKILL.md`) in its own folder. There is no build system, test runner, or linter.

## Structure

```text
{skill-name}/SKILL.md          # Skill entry point (YAML frontmatter + instructions)
{skill-name}/*.md              # Supporting docs, formats, or deep-dive modules
clarity-council/               # Monorepo for the clarity-council skill suite
clarity-council/skills/        # Nested sub-skills (persona_consult, council_consult, etc.)
clarity-council/skills/personas/  # Persona contracts
```

## Skill format

Each `SKILL.md` begins with YAML frontmatter:

```yaml
name: skill-name
description: >
  When to trigger this skill. Use "when user says X" or "Use when Y."
  Keep to 1-2 sentences.
version: 2.5.1          # optional
license: MIT            # optional
compatibility: claude-code opencode  # optional
allowed-tools:          # optional
  - Read
  - Write
```

## Skills inventory

> **Public skills only.** Private skills live in `_private/` and surface as junctions in the root (e.g. `cpf`, `mr-*`, `setup-*`). They are gitignored and **must not** be listed here. To check what's private: `cmd //c "dir /AL"` lists junctions, and `.gitignore` is the canonical list.

| Skill | Purpose |
| :--- | :--- |
| branch-rebase | Safely rebase the current branch onto its upstream target; auto-resolves trivial conflicts |
| break-it-down | Decode messages into plain language with tone/intent analysis |
| clarity-council | Persona-based consultation (single/multi-persona, iterative) |
| codebase-churn | Git-history treemap (SVG): area = lines changed, color = commit frequency — find unstable, bug-prone files |
| codebase-explain | High-level context and module mapping for unfamiliar code areas |
| codebase-improve-architecture | Find refactor/deepening opportunities, informed by CONTEXT.md and ADRs |
| daily-briefing | Outlook-driven daily briefing: 24h email + calendar recap, today's schedule, action items, focus blocks |
| daily-standup-prep | Per-team standup report: gathers Jira/GitLab/Confluence/Git activity over N days, maps to roster, renders Mermaid kanban + talking order, writes to vault |
| decision-breaker | Force a pick between options to defeat analysis paralysis. Council: senior-architect + devils-advocate + personal-assistant vote, majority wins |
| defuddle | Extract clean markdown from web pages via Defuddle CLI (use instead of WebFetch for HTML) |
| energy-budget | Spoon-theory accounting for the calendar: score today's load, flag burnout risk, suggest defers. Council: personal-assistant + psychologist |
| good-morning | Morning kickoff wrapper: runs sprint-snapshot (daily tag) → daily-standup-prep (with burndown) → daily-briefing (report only, no focus blocks). Idempotent — overwrites today's artifacts on re-run |
| grill-me | Stress-test plans through iterative questioning |
| grill-with-docs | Grilling against domain docs + updating CONTEXT/ADR artifacts inline |
| hyperfocus-recovery | Reconstruct context after a deep session or interruption from git/file artifacts → suggested re-entry point |
| interest-capture | Fast capture for hyperfixations so they don't derail today's work but aren't lost. Files into Obsidian inbox, returns user to task |
| issue-dup-find | Scan open Jira issues (default project from memory) for likely duplicates → markdown report with probability + reason (read-only) |
| issue-estimate-sp | Story point estimation via Jira context + scrum-poker council session (read-only) |
| issue-feature-breakdown | Read-only Jira/Confluence context gathering + council analysis → actionable breakdown |
| issue-suggest-component | Suggest Jira components for a ticket or sweep a project (default project from memory, max 250). Confirms before each add; can create new components |
| issue-triage | Triage Jira ticket or free-form bug → ranked root-cause hypotheses + solution paths (read-only) |
| meeting-decompression | Process a meeting dump: separate facts/action-items/social ambiguities. Sorts ruminations into "worth following up on" vs "RSD noise". Council: psychologist + personal-assistant |
| obsidian-bases | Create and edit Obsidian Bases (.base files) — views, filters, formulas, summaries |
| obsidian-canvas | Create and edit Obsidian .canvas files (nodes, edges, groups, connections) |
| obsidian-cli | Interact with Obsidian vaults via CLI: notes, tasks, properties; supports plugin/theme dev |
| obsidian-markdown | Create and edit Obsidian Flavored Markdown (wikilinks, callouts, frontmatter, embeds) |
| obsidian-vault | Search, create, and manage notes in the Obsidian vault with wikilinks and index notes |
| request-refactor-plan | Build incremental refactor plan via interview → file as GitHub issue |
| skill-create | Create new skills with structured prompts, resources, and packaging conventions |
| sprint-plan | Convert the start-of-sprint canvas into a planning markdown report (committed scope split carry-over vs new commit, capacity vs commitment, WIP-saturation risk, observations, risks). Assumes all unclosed items from the previous sprint's `end.canvas` carry into this one. Date-stamped output: same-day re-runs refresh silently, prior days preserved. Auto-runs clarity-council. Port of `Prompts/Sprint Plan.md`. |
| sprint-review | End-of-sprint stakeholder report: compares `start.canvas` to `end.canvas`, fits the standard SM template (Accomplishments / Status / Sprint Commitment / PI Confidence / Impediments). Auto-runs clarity-council. Port of `Prompts/Sprint Review.md`. |
| sprint-snapshot | Point-in-time snapshot of a scrum team's sprint board: writes Obsidian Canvas + companion markdown summary + append-only JSONL trend log into the team's vault folder. Auto-detects sprint phase; supports `--as-of` for historical snapshots. Port of `Sprint-Planner.ps1`. Hosts the shared `_team-rules.md` schema used by all `sprint-*` skills. |
| sprint-sos-report | Weekly scrum-of-scrums report comparing two sprint snapshots: scope changes, status transitions, per-member workload delta, wedge balancing, council findings/observations/trouble-areas/trends. Auto-detects which two snapshots to compare (`--from`/`--to` overrides for re-planning churn). Port of `Prompts/Scrum of Scrums.md`. |
| task-initiation | Defeat executive-function stalls with one ≤30-second physical action — not a plan, not a breakdown. Inline-only (no council, speed matters) |
| time-reality-check | Counter time blindness with a calibrated three-point estimate (best/likely/worst) accounting for setup, interruptions, re-entry. Council: statistics-expert + devils-advocate |
| writing-apology-calibrator | Calibrate a drafted apology: strip reflexive over-apology, keep warranted accountability. Council: psychologist + devils-advocate |
| writing-beats | Shape an article as a journey of beats, choose-your-own-adventure style |
| writing-cold-open | Produce just the first sentence of a message when the blank cursor is winning. Three openings (direct/warm/contextual), then steps out |
| writing-draft-article | Guide from raw idea/draft to a finished, polished article through iterative questioning |
| writing-fragments | Mine the user for raw writing fragments before imposing structure |
| writing-humanize | Remove signs of AI-generated writing from text |
| writing-rejection-sensitivity-check | Calibrate a stung-by message: separate evidence from interpretation, score warranted sting 1–5. Council: psychologist + devils-advocate (always) |
| writing-shape | Shape raw material into an article paragraph by paragraph through a conversational session |
| writing-social-script | Generate a literal script for a dreaded social scenario. Three phrasings (direct/neutral/softened) + exit line. Council: psychologist + customer-advocate |
| writing-tone-check | Pre-send tone reviewer: paste a draft, get a landing prediction (cold / passive-aggressive / over-apologetic) + rewrite suggestions. Sibling to break-it-down. Council: psychologist |

## Adding a new skill

1. Create `{skill-name}/SKILL.md` with proper YAML frontmatter (name, description).
2. Add a row to the **Skills inventory** table above in this file — **but only for public skills**. If the skill lives under `_private/` (junction in the root), skip the inventory row and add it to `.gitignore` instead. (README.md links to the inventory — do not duplicate it there.)
3. Keep supporting docs inside the skill folder.
4. Folder names are stable — do not rename once published.

## Editing an existing skill

- The `SKILL.md` at the folder root is the entry point. Never move it.
- Supporting files (formats, deep-dive modules, persona contracts) stay in the same folder.
- Update the skill's README if the workflow or reference files change.
