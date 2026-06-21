---
name: good-morning
description: Morning kickoff wrapper that runs sprint-snapshot (daily tag), daily-standup-prep (with sprint burndown), and daily-briefing (report only — no focus-block suggestions) in order. Re-runs idempotently — existing same-day artifacts are overwritten in place. Use when user says "good morning", "morning routine", "kick off my day", "start my day", or invokes /good-morning.
related-agents:
  - scrum-master
  - project-manager
related-skills:
  - sprint-snapshot
  - daily-standup-prep
  - daily-briefing
loop-eligible: false
recurrence-hint: daily
allowed-tools:
  - Skill
  - AskUserQuestion
  - Bash
  - Read
  - Edit
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
3. **`daily-briefing`'s stale-carryover prompt or recently-closed re-surface prompt** — these are intentional user decisions about specific action items (still open vs done vs cancel vs defer; re-spawn vs false re-surface). Forward as-is; never auto-answer "Still open" on the user's behalf, since silently keeping a stale item is exactly the failure mode these prompts exist to prevent. See `daily-briefing/SKILL.md` § "Recently-closed dedup rule" and § "Stale carryover rule".

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

**Then run the daily-note section-completeness check (belt-and-suspenders).** `daily-briefing`'s § "Pre-save section checklist" should guarantee these, but the chart silently vanished from 4 consecutive June 2026 notes before the checklist existed — so the wrapper re-verifies. `Read` today's daily note (`{{vault_root}}/📅/YYYY/MM/YYYY-MM-DDD.md`) and confirm it contains all four mandatory surfaces (see [[feedback_daily_note_mandatory_sections]]):

1. A `#### Day at a glance` Mermaid `gantt` block.
2. A `#### Email triage at a glance` Mermaid `pie` block.
3. A `### 👥 Team Daily Notes` standup wikilink — unless `Glob("{{vault_root}}/Scrum Teams/**/YYYY-MM-DD.md")` returns zero matches for today.
4. A `### 📊 Cross-day context` block with the `![[Daily Notes Dashboard.base#Recent (cards)]]` + `![[TODO#By tag]]` embeds.

If any is missing, `Edit` it into the note in place (reconstruct the gantt from the schedule section, the pie from the email-triage counts, the standup link from the `Glob` result) and append a one-line note to the Phase 4 summary: `↻ Repaired missing daily-note sections: <list>`. Do NOT prompt — the sections are mandatory, so repair silently. If this check repairs sections on >1 run, that signals `daily-briefing`'s pre-save checklist is being skipped and needs tightening.

**Then run the action-item post-filter pass (belt-and-suspenders).** `daily-briefing` should have applied the verb test and addressed-to-user classifier from its § 3a and the phantom-carryover guard from § 6, but those rules occasionally let an FYI slip through as a `- [ ]` item. Re-read today's daily note and audit the `## ✅ Open Action Items` section:

1. `Read` `{{vault_root}}/📅/YYYY/MM/YYYY-MM-DDD.md` and extract every `- [ ]` line from `### Carrying over (still open)` and `### New / from window`.
2. For each item, re-apply the heuristics from `daily-briefing/SKILL.md` § 3a:
   - **Verb test** — does the bullet name an actionable verb directed at the user (reply / review / approve / merge / sign / decide-with-deadline / schedule / draft / fix / investigate / RSVP)? Status verbs (is, was, has been) fail.
   - **Addressed-to-user test** — is the ask aimed at the user specifically, or is it broadcast / informational / conditional?
   - **OOO / FYI patterns** — "reach out to X backups" (X is OOO, no actual task), "read [meeting notes]" with no decision pending, "verify Y was sent" with no time-criticality, "decide: [topic]" with no external deadline or stakeholder waiting.
   - **Phantom test** — does the bullet contain a person/MR#/Jira ticket that does NOT appear anywhere in today's `📨 Email Triage` section or `🗓️ Today's Schedule`? If so, there's no fresh signal backing it and it's likely stale.
3. If any items fail the audit, `Edit` the daily note:
   - Remove the bullet from `## ✅ Open Action Items`.
   - Insert it as a plain bullet (no checkbox) under `## 📝 Notes & Follow-ups` if the item still has informational value (OOO notice, status awareness).
   - Drop it entirely if it's a phantom with no information content.
4. Append a one-line console note in the Phase 4 summary listing what was reclassified: `↻ Demoted N action items to Notes & Follow-ups: <one-line list with reason per item>`. Do NOT prompt — the heuristics are conservative enough to demote silently per user feedback. If the user wants to keep something demoted, they can manually restore it.

This is a sanity net, not the primary classifier. If this step is consistently demoting >2 items per run, that's a signal that `daily-briefing/SKILL.md` § 3a needs to be tightened further.

**Then run a personal-assistant pass.** Phase 3 (daily-briefing) was invoked with `--no-focus-blocks`, so the user never sees a focus-block proposal during `/good-morning`. To preserve the "what should I actually do next" signal without re-introducing calendar drafts, invoke via `Skill` agent: with `council-single-persona` agent, persona pinned to `personal-assistant`. Pass:

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
