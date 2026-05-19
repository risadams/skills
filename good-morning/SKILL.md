---
name: good-morning
description: Morning kickoff wrapper that runs sprint-snapshot (daily tag), daily-standup-prep (with sprint burndown), and daily-briefing (report only — no focus-block suggestions) in order. Re-runs idempotently — existing same-day artifacts are overwritten in place. Use when user says "good morning", "morning routine", "kick off my day", "start my day", or invokes /good-morning.
allowed-tools:
  - Skill
  - AskUserQuestion
  - Bash
  - Read
  - Glob
---

# Good Morning

Single-command wrapper that runs the user's three morning skills in order. Owns no business logic of its own — every artifact is produced by the delegated skill. This skill exists so the user types one command instead of three.

## Quick start

```text
/good-morning                 → run all three phases for default team(s)
/good-morning Aurora          → pass team override down to sprint-* skills
/good-morning --skip-snapshot → skip phase 1 (e.g. snapshot already captured today)
```

Any extra args (team names, `--inc`, `--sprint`, `--days`, etc.) are forwarded to the sub-skills that recognise them. The wrapper itself only consumes `--skip-snapshot`, `--skip-standup`, `--skip-briefing`.

## Workflow

```text
- [ ] Phase 1: sprint-snapshot --phase daily            (Jira → vault canvas + md + jsonl)
- [ ] Phase 2: daily-standup-prep --include-sprint-pulse (per-team standup report + burndown)
- [ ] Phase 3: daily-briefing --no-focus-blocks         (Outlook 24h recap + today's schedule)
- [ ] Phase 4: One-line summary of artifacts written
```

Run phases **sequentially, never in parallel**. Phase 2 depends on Phase 1's JSONL trend row (burndown forecast reads it). Phase 3 is independent but ordered last by design.

**Auto-continue between phases.** When a phase completes, immediately invoke the next one — do not pause for user acknowledgment, do not summarise mid-flight, do not ask "shall I continue?". The user invoked /good-morning expecting the whole chain to run unattended. The only legitimate stop points are:

1. A hard failure in Phase 1 (abort the wrapper).
2. A sub-skill raising a genuinely blocking `AskUserQuestion` that the wrapper cannot answer on the user's behalf (e.g. missing roster CSV with no source path). In that case, forward the prompt as-is.

Overwrite prompts, "continue?" prompts, and any other yes/no the wrapper can answer for the user should be auto-answered with the continue/overwrite option — never paused for input.

### Phase 1 — Sprint snapshot (daily tag)

Invoke via `Skill`: `sprint-snapshot` with `--phase daily` (preserves literal `daily` in filenames per sprint-snapshot's phase rule). Forward any team / inc / sprint args the user supplied.

Tell the sub-skill the overwrite behavior up-front so it doesn't prompt: *"This run is from /good-morning — if `daily.canvas` or `daily.md` already exist for today, overwrite in place. Do not prompt."* The JSONL trend log is append-only — sprint-snapshot already handles that correctly.

If sprint-snapshot fails (Jira down, roster missing), surface the error and **abort the wrapper** — phases 2 and 3 expect the snapshot to exist.

### Phase 2 — Standup report (with sprint burndown)

Invoke via `Skill`: `daily-standup-prep`. Pass `IncludeSprintPulse=true` explicitly so the clarity-council burndown chart + statistics-expert forecast + scrum-master suggestions run (this is the "sprint burndown" the user wants). Forward team / days args from the wrapper invocation.

Tell the sub-skill: *"Overwrite today's standup file(s) without prompting — /good-morning expects idempotent re-runs."*

If daily-standup-prep partially fails (e.g. GitLab unreachable), let it produce what it can — its degraded-mode output is still useful. Surface any warnings in Phase 4's summary.

### Phase 3 — Daily briefing (report + rollups, no focus blocks)

Invoke via `Skill`: `daily-briefing`. Phase 3 must produce **all three artifacts**:

1. **Daily report** — `{{vault_root}}\📅\YYYY\MM\YYYY-MM-DDD.md` (daily-briefing step 4)
2. **Weekly rollup** — `{{vault_root}}\📅\YYYY\MM\YYYY-W{NN}.md` (daily-briefing step 5)
3. **Monthly rollup** — `{{vault_root}}\📅\YYYY\YYYY-M{MM}.md` (daily-briefing step 6)

**Skip step 9 (focus-block proposal) entirely.** Tell the sub-skill: *"/good-morning is the caller — run steps 1 through 6 (daily report + weekly rollup + monthly rollup). Skip step 9 (focus blocks). Do NOT propose calendar blocks, do NOT create any meeting drafts. Stop after the monthly rollup is written."*

Same overwrite rule: if any of the three notes already exist for today, overwrite in place — no prompts.

### Phase 4 — Summary

Print one block summarising what was produced:

```text
=== Good morning — 2026-05-18 ===
✓ Snapshot:  [daily.canvas]  [daily.md]              (+1 trend row)
✓ Standup:   [Aurora 2026-05-D18.md]                 (burndown ↘ on-track)
✓ Briefing:  [2026-05-D18.md]  [2026-W20.md]  [2026-M05.md]  (12 unread, 4 meetings today)
```

Use markdown links so the user can click straight into the vault.

**Then run a personal-assistant pass.** Phase 3 (daily-briefing) was invoked with `--no-focus-blocks`, so the user never sees a focus-block proposal during `/good-morning`. To preserve the "what should I actually do next" signal without re-introducing calendar drafts, invoke the `clarity-council` skill via `Skill` with `persona_consult` mode, persona pinned to `personal-assistant`. Pass:

- **user_problem:** *"Given today's standup and briefing artifacts, what are the 3 most important next actions for the user?"*
- **context:** the executive-summary bullets from the daily briefing (read from `{{vault_root}}/📅/YYYY/MM/YYYY-MM-DDD.md`), the burndown verdict from the standup report, and any impediments/blockers surfaced in Phase 2.
- **desired_outcome:** *"A `## Next actions` block — 3 bullets max, each naming a concrete action, timing/deadline, and any dependency or follow-up. No calendar drafts (that's owned by `/daily-briefing` step 9 when invoked standalone)."*
- **depth:** `brief`

Append the persona's `## Next actions` block to the Phase 4 console output. This makes the wrapper output actionable in its own right rather than just an artifact-link manifest.

## Rules

- **Sequential, never parallel.** Phase 2 reads Phase 1's JSONL; Phase 3 is independent but ordered last by design.
- **Auto-continue between handoffs.** When one phase finishes, the next starts immediately. No pause, no acknowledgment prompt, no mid-flight summary. Only stop for a hard Phase-1 failure or a genuinely blocking sub-skill question the wrapper can't auto-answer.
- **Idempotent.** Re-running on the same day overwrites today's artifacts in place. Never prompt — the user has already opted in by invoking /good-morning. The JSONL trend log is the only exception (append-only, by design).
- **Phase 3 = three artifacts.** Daily report + weekly rollup + monthly rollup. All three. Skip only step 9 (focus blocks).
- **No focus blocks, no calendar drafts.** Phase 3 is report-only. Do not propose blocks, do not create meeting drafts.
- **Forward, don't reinterpret.** Team names, `--inc`, `--sprint`, `--days`, etc. are passed through verbatim to whichever sub-skill recognises them. The wrapper doesn't validate them.
- **Abort on Phase 1 failure.** A missing snapshot breaks Phase 2's burndown. Phases 2 and 3 may degrade gracefully; Phase 1 may not.
- **No new memory writes.** All config (vault root, default team, Jira project, etc.) is resolved by the sub-skills from their existing memory keys. The wrapper introduces no new memory files.

## Edge cases

- **`--skip-snapshot`** — skip Phase 1 only if the user explicitly opts out (e.g. they already ran it from `/sprint-snapshot` earlier). Phase 2 will still attempt to read the most recent JSONL row.
- **`--skip-standup` / `--skip-briefing`** — same pattern; document in the Phase 4 summary which phases ran.
- **Sub-skill prompts despite the override instruction** — answer with the overwrite-equivalent option (`Overwrite`) on the user's behalf, since /good-morning already implies consent. Do not re-ask.
- **Today is a weekend or holiday** — still run all three phases; the artifacts are useful for catch-up. daily-briefing's window logic already handles gap days.
