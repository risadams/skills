---
name: daily-briefing
description: Personal daily briefing assistant. Pulls the last 24 hours of email and calendar from Outlook, summarizes today's schedule, surfaces open action items, and proposes focus blocks. Use when user says "daily briefing", "morning briefing", "daily standup prep", or invokes /daily-briefing.
allowed-tools:
  - mcp__outlook__outlook_get_inbox_emails
  - mcp__outlook__outlook_get_calendar_events
  - mcp__outlook__outlook_get_email_by_id
  - mcp__outlook__outlook_get_unread_emails
  - mcp__outlook__outlook_search_emails
  - mcp__outlook__outlook_get_folder_emails
  - mcp__outlook__outlook_create_meeting_draft
  - mcp__outlook__outlook_check_calendar_conflict
  - mcp__outlook__outlook_get_current_user
  - AskUserQuestion
  - Bash
  - Write
  - Read
  - Glob
  - Skill
---

# Daily Briefing

Personal daily briefing. Read-only by default for the report; calendar focus-block drafts require user approval before creation.

## Related skills

- **obsidian-markdown** — canonical reference for Obsidian Flavored Markdown (frontmatter, wikilinks, embeds, callouts, properties). The vault writes in this skill use OFM syntax; consult `obsidian-markdown` if you need to extend the templates with new constructs (callouts, block IDs, mermaid options, etc.) rather than reinventing the syntax inline.
- **obsidian-bases** — reference for `.base` file authoring (filters, formulas, views). Used by step 6 when the embedded dashboards are missing from the vault.

## Quick start

1. Find the most recent prior daily note in the vault to determine the briefing window (covers gaps from weekends, alternating Fridays off, holidays, vacations).
2. Pull inbox + calendar from that window-start through now, plus today's calendar.
3. Discover vault link conventions (person notes, tags) — see step 5.
4. Render the daily report (see [REPORT_TEMPLATE.md](REPORT_TEMPLATE.md)) and **save it to the Obsidian vault**.
5. Update the weekly rollup note (see [WEEKLY_TEMPLATE.md](WEEKLY_TEMPLATE.md)).
6. Update the monthly rollup note (see [MONTHLY_TEMPLATE.md](MONTHLY_TEMPLATE.md)).
7. Propose focus blocks; ask the user before creating any drafts.

## Vault path resolution

The skill needs the **Obsidian vault root** to read prior notes and write new ones. It is referenced throughout this document as `{{vault_root}}` — never hardcode a path.

Resolve `{{vault_root}}` once at the start of every run, in this order:

1. **Loaded memory.** Scan the auto-loaded `MEMORY.md` index and any reference memories already in context for a `**Vault root:**` line (or equivalent key like `vault_root:` / `obsidian_vault:`). The canonical entry is `reference_obsidian_vault.md` — its first content line reads `**Vault root:** <absolute path>`. Use that path verbatim.
2. **Memory file lookup.** If the index references a vault memory (e.g. `[Obsidian vault](reference_obsidian_vault.md)`) but the file isn't already loaded, `Read` it and extract the `**Vault root:**` value.
3. **Prompt the user.** If neither of the above yields a path, ask via `AskUserQuestion`:
   *"I need your Obsidian vault root path to save the daily briefing. What's the absolute path? (e.g. `C:\Users\you\Documents\my-vault\`)"*
   Single-select with one option — `Other` lets the user type the path.
4. **Persist the answer.** After receiving a user-provided path, save it to memory so future runs skip the prompt:
   - If `reference_obsidian_vault.md` exists, `Read` it, update the `**Vault root:**` line, and `Write` it back.
   - If it does not exist, create it with frontmatter (`name: obsidian-vault-root`, `type: reference`, short description) and a `**Vault root:**` body line, then add a one-line entry to `MEMORY.md`: `- [Obsidian vault root](reference_obsidian_vault.md) -- vault base path used by daily-briefing`.
5. **Validate.** Verify `{{vault_root}}` exists with a single `Glob("{{vault_root}}/*")`. If empty or errors, surface the failure and re-prompt — do not write to a non-existent path.

After resolution, every path literal below uses `{{vault_root}}` as the prefix.

**Daily note path:** `{{vault_root}}\📅\YYYY\MM\YYYY-MM-DDD.md` — Pittsburgh local date, with a literal `D` prefix on the **day component only** (e.g., `2026\05\2026-05-D05.md` for May 5, 2026).

**Weekly note path:** `{{vault_root}}\📅\YYYY\MM\YYYY-W{NN}.md` — ISO week-year and week number, month folder follows the week's Monday (e.g., `2026\05\2026-W19.md` for the week of May 4, 2026).

**Monthly note path:** `{{vault_root}}\📅\YYYY\YYYY-M{MM}.md` — calendar year and month. Lives in the **year folder**, not in a month subfolder (e.g., `2026\2026-M05.md` for May 2026).

## Workflow

### 1. Determine briefing window

Find the most recent prior daily note so the window covers everything since the last run — gaps from weekends, alternating Fridays off, holidays, and vacations all collapse to a single catch-up briefing.

1. `Glob("{{vault_root}}/📅/**/[0-9][0-9][0-9][0-9]-[0-9][0-9]-D[0-9][0-9].md")` — enumerate all daily notes.
2. Parse the date from each filename (`YYYY-MM-DDD.md` → `YYYY-MM-DD`). Filter to dates **strictly before today** (Pittsburgh local).
3. Pick the maximum (most recent) date — call it `last_run_date`.
4. Compute `window_start`:
   - **Default** — `last_run_date` at 06:00 America/New_York (right before that morning's briefing was generated).
   - **No prior note found**, or the most recent note is older than **14 days** — fall back to 24h before now and tell the user "no recent prior note within 14 days; using 24h fallback window."
   - **Cap at 14 days** even if a prior note exists further back, to avoid blowing context. Warn the user when capping.
5. Compute `gap_days = today - last_run_date`. If `gap_days > 1`, this is a **backfill briefing** — note that in the executive summary and label the email section with the actual date range.

State the chosen window in one short sentence before gathering data, e.g. *"Backfilling since [[2026-05-D07]] (Thu) — 4-day window."*

### 2. Gather data (parallel)

Call these in a single response, using `window_start` from step 1:

- `outlook_get_inbox_emails(since=<window_start ISO>, include_read=true, max_results=50)` — email since last briefing. If `gap_days > 2`, raise `max_results` to 100.
- `outlook_get_calendar_events(start=<window_start>, end=<now>)` — meetings since last briefing.
- `outlook_get_calendar_events(start=<today 00:00>, end=<today 23:59>)` — today's schedule.

If inbox volume is high (>30 emails), also call `outlook_get_unread_emails(max_results=25)` to ensure unread items aren't lost.

`outlook_get_inbox_emails` accepts ISO datetimes for `since` (also `today`, `yesterday`, `7d` shorthands). Prefer the explicit ISO datetime so backfill windows of arbitrary length are exact.

### 3. Triage email

For each email in the briefing window, classify into one bucket:

- **Action required** — direct ask, question, deadline, @mention, or you're the sole recipient
- **FYI** — informational, CC'd with no ask, automated digest
- **Awaiting reply** — emails *you* sent that have no response yet (check Sent Items if needed)

Only fetch full body via `outlook_get_email_by_id` when subject + preview is ambiguous. Keep it cheap.

### 4. Analyze today's schedule

Identify:

- **Conflicts** — overlapping meetings
- **Back-to-back stretches** — 3+ consecutive meetings with no gap
- **Prep gaps** — high-stakes meetings (external attendees, leadership, customer) with no buffer beforehand
- **Free blocks** — contiguous gaps ≥45 min suitable for focus work
- **No-lunch flag** — no free block between 11:30–13:30

### 5. Discover vault link conventions

Before rendering, scan the vault to find existing notes for people mentioned in the briefing data (meeting attendees, email senders). This lets the report use real `[[wikilinks]]` instead of plain names.

**People convention:** each person has their own folder under `🤼 Team/{Category}/{Display Name}/` (e.g., `Dev Teams/Alex R/`, `Dev Teams/Sam/`), and the canonical note inside is `@Firstname Lastname.md` (with literal `@` prefix). Wikilink target uses that filename without extension: `[[@Firstname Lastname]]`.

- `Glob("{{vault_root}}/🤼 Team/**/@*.md")` — enumerate all canonical person notes
- Build a name → wikilink-target map. Match flexibly: full name from email/calendar may need to map to the existing `@First Last` note
- If no match, render the name as plain text — **never invent a wikilink target** (broken links pollute the graph)
- For team/project names (e.g., "Quartz", "Garnet", "Slate" under `Scrum Teams/`), link similarly when a folder/note exists. Verify with `Glob` before linking.

**Team daily notes convention:** each scrum team has its own daily note per work date under `Scrum Teams/{Team}/Scrum 📅/{INC}/{Sprint}/YYYY-MM-DD.md`. The filename is plain ISO date (no `D` prefix — that distinguishes it from your personal daily note `YYYY-MM-DDD.md`). The three parent folders carry the team, increment, and sprint metadata.

- `Glob("{{vault_root}}/Scrum Teams/**/YYYY-MM-DD.md")` — for each date in scope (today for the daily report; each Mon–Fri for the weekly; each week's dates for the monthly), enumerate matching team notes.
- Parse the path: split on `/`, find the segment after `Scrum Teams/` (= **team**), then the segments under `Scrum 📅/` (= **INC**, then **sprint**).
- Wikilink format — use the **full path** with a display alias so Obsidian resolves unambiguously and the link text stays short: `[[Scrum Teams/{Team}/Scrum 📅/{INC}/{Sprint}/YYYY-MM-DD|{Team}]]`. Append the INC/sprint inline as plain text after the link.
- If no team notes exist for a given date, omit the team-notes line entirely (don't render an empty header — the absence is the signal).

### 6. Render the report and save to vault

Use [REPORT_TEMPLATE.md](REPORT_TEMPLATE.md). Lead with the executive summary (3-5 bullets), then the structured sections. Keep the whole report scannable in <60 seconds.

**Carryover rule (Open Action Items):** when populating the `Carrying over (still open)` subsection from the prior daily note (`last_run_date`), `Read` that note's `### ✅ Open Action Items` section and copy **only `- [ ]` lines**. Every `- [x]` line is already done — drop it. This rule applies to both subsections of the prior note (`Carrying over` and `New / from window`); merge the surviving `- [ ]` items into today's `Carrying over (still open)`. If the prior note has zero remaining `- [ ]` items, omit the subsection entirely rather than render an empty header.

**Embedded visual surfaces** — the template includes `![[Daily Notes Dashboard.base#...]]` and `![[TODO#By tag]]` embeds that auto-render when the file opens. They depend on these vault files existing:
- `📅/Daily Notes Dashboard.base` (views: `All daily notes`, `This Week`, `Recent (cards)`) — scopes to daily notes (`📅/**/YYYY-MM-DDD.md`)
- `📅/Weekly Rollups.base` (views: `All weeks`, `Recent weeks`) — scopes to weekly notes (`📅/**/YYYY-W{NN}.md`)
- `📅/Scrum Team Activity.base` (views: `Recent activity`, `By team`) — scopes to scrum team daily notes under `Scrum Teams/`
- `TODO.md` (sections: `By tag`) — Dataview-driven; out of scope for auto-creation here

**Detect missing bases on every run.** Before saving the daily report:

1. `Glob` for each of the three `.base` files at `📅/`. Build a list of which are missing.
2. If any are missing, prompt the user via `AskUserQuestion`: *"The following dashboard bases are missing from the vault: {{list}}. Create them now? They power the embedded views in the daily/weekly/monthly reports."* Options: `Create all (recommended)`, `Skip — leave broken-link placeholders`, `Choose individually`.
3. For each base the user approves, invoke the `obsidian-bases` skill via `Skill` with the target path, scope (which notes the base filters), and the named views above. `obsidian-bases` is the canonical reference for `.base` YAML grammar — defer schema decisions to it rather than authoring inline. Cross-check the generated YAML against `obsidian-bases`'s troubleshooting section (Duration math, formula quoting) before writing.
4. If the user declines, leave the embed in place — creating the missing file later will fix the rendering automatically.

This check is cheap (3 `Glob` calls) and after the first-run accept, becomes a no-op on subsequent days. `TODO.md` is Dataview-managed by the user and is *not* auto-created here — surface its absence as a warning only.

**Save to vault:**

1. Compute Pittsburgh date: `YYYY`, `MM`, `YYYY-MM-DD`.
2. Create folders if missing — single command: `mkdir -p "{{vault_root}}/📅/YYYY/MM"` (forward slashes work in bash on Windows; the 📅 emoji is preserved).
3. Target file: `{{vault_root}}/📅/YYYY/MM/YYYY-MM-DDD.md` — the `D` prefix goes on the **day component only**, not the year/month and not the folder names. Example for May 5, 2026: `2026/05/2026-05-D05.md`.
4. **Do not overwrite an existing file silently.** If it exists, `Read` it first and ask the user via `AskUserQuestion` whether to overwrite, append a new section (e.g., `## Afternoon update — HH:MM`), or skip the save.
5. Use `Write` with the full report content (frontmatter + body per template).
6. Print the saved path back to the user as a clickable markdown link.

After saving, also display the executive summary inline in chat so the user gets the briefing without opening the file.

### 7. Update the weekly rollup note

After the daily note is saved, regenerate the weekly rollup. Full template, ISO-week math, and preservation rules in [WEEKLY_TEMPLATE.md](WEEKLY_TEMPLATE.md).

**Workflow:**

1. Compute ISO week-year, ISO week number, and the Monday→Friday dates for this week (Pittsburgh time).
2. Compute the weekly file path: `📅/YYYY/MM/YYYY-W{NN}.md` where `MM` = month of the Monday.
3. Ensure the month folder exists (`mkdir -p`).
4. For each Mon–Fri date, check if `📅/YYYY/MM/YYYY-MM-D{DD}.md` exists; if so, `Read` it and extract every `- [ ]` line (unchecked only). Also `Glob("Scrum Teams/**/YYYY-MM-DD.md")` for the same date to collect team daily notes (parse team/INC/sprint from the path per step 5).
5. If a weekly file already exists at the target path, `Read` it and capture everything after the `<!-- end:auto-managed -->` marker — that user content must be preserved in the new write. If markers are missing, ask via `AskUserQuestion` before overwriting.
6. Render the new weekly content per [WEEKLY_TEMPLATE.md](WEEKLY_TEMPLATE.md): navigation links, week-at-a-glance (2-4 bullets pulled from today's executive summary + cross-day themes), and the per-day TODO rollup.
7. `Write` the file: auto-managed region + preserved user content (or default `## Notes` placeholder if none).
8. Print the saved weekly path back to the user as a clickable markdown link.

**Bidirectional links:** the daily note's frontmatter or footer should link to the weekly note (`[[YYYY-W{NN}]]`), and each daily entry in the weekly rollup links to the daily note. This makes Obsidian's graph and backlinks work both ways.

### 8. Update the monthly rollup note

After the weekly note is saved, regenerate the monthly rollup. Full template, week-selection logic, and preservation rules in [MONTHLY_TEMPLATE.md](MONTHLY_TEMPLATE.md).

**Workflow:**

1. Compute the current calendar year and month (Pittsburgh time).
2. Compute the monthly file path: `📅/YYYY/YYYY-M{MM}.md` — this lives in the **year folder**, not a month subfolder.
3. Ensure the year folder exists (`mkdir -p`).
4. Determine which weeks to include: all ISO weeks whose **Monday** falls in the current month, plus one trailing week for forward visibility (typically 5 entries; 6 if the month has 5 in-month Mondays).
5. For each selected week, check if `📅/YYYY/MM/YYYY-W{NN}.md` exists (where `MM` follows that week's Monday). If so, `Read` it and extract the bullets under the `#### 🎯 Week at a glance` section. Also `Glob("Scrum Teams/**/YYYY-MM-DD.md")` for each Mon–Fri of the week and aggregate the distinct **{team} {INC} / {sprint}** combinations that appeared at least once during the week.
6. If a monthly file already exists at the target path, `Read` it and capture everything after the `<!-- end:auto-managed -->` marker. If markers are missing (legacy format), prompt via `AskUserQuestion` before overwriting.
7. Render the new monthly content per [MONTHLY_TEMPLATE.md](MONTHLY_TEMPLATE.md): navigation with `<button class="date_button_today">This Month</button>`, month-at-a-glance (2-4 synthesized themes), and the per-week rollup with date ranges.
8. `Write` the file: auto-managed region + preserved user content.
9. Print the saved monthly path back to the user as a clickable markdown link.

**Note on cascading updates:** the monthly rollup reads from weekly notes, which read from daily notes. Always update in order — daily → weekly → monthly — so each layer reflects the latest data.

### 9. Propose focus blocks

Map open action items + prep needs onto free blocks. Present a proposal table:

| Time | Block | Purpose |
| :--- | :--- | :--- |

Then ask the user via `AskUserQuestion` which blocks (if any) to add to the calendar. For each approved block, call `outlook_check_calendar_conflict` first, then `outlook_create_meeting_draft` with no attendees and `busy_status="busy"`. Never create blocks without explicit approval.

## Rules

- **Read-only for Outlook.** Never mark emails read, flag, move, or delete during briefing.
- **Vault writes are scoped.** Only write to `{{vault_root}}\📅\` — never modify other vault folders.
- **Never overwrite an existing daily note silently.** Always confirm with the user first.
- **Weekly and monthly notes: auto-managed region only.** Regenerate the content between the `<!-- begin:auto-managed by daily-briefing skill -->` and `<!-- end:auto-managed -->` markers. Preserve everything outside those markers verbatim. If markers are missing on an existing weekly or monthly note (legacy hand-edited format), prompt before overwriting — there may be user-authored content to migrate.
- **Checkboxes for action items.** All open action items use `- [ ]` so the global `TODO.md` Dataview query (which scans `📅/`) picks them up automatically.
- **Wikilinks only when targets exist.** Verify the note exists before writing `[[Name]]` — broken links pollute the graph.
- **Drafts only for calendar blocks.** Use `outlook_create_meeting_draft` (opens for review); never auto-send or auto-create finalized appointments.
- **Confirm before drafting.** Always ask which proposed blocks to create — don't batch all.
- **Respect privacy.** If an email subject contains "confidential", "personal", "HR", or "legal", surface only "private email from <sender>" without subject details.
- **Time math.** All times reported in **local time** (America/New_York — EST/EDT, observes DST). Outlook returns local; if any value comes back in UTC or another zone, convert before display. When computing "24 hours ago", anchor on current Pittsburgh wall-clock time.
- **Unavailable MCP.** If `mcp__outlook__*` tools fail, tell the user the Outlook MCP isn't responsive and stop — don't fabricate a briefing.

## Edge cases

- **No emails / empty calendar:** still render the report with "Nothing to triage" / "Open day" — useful signal.
- **All-day events:** treat as context, not as schedule blockers (unless `busy_status="out_of_office"`).
- **Recurring meeting series:** count as one item in the briefing, not N.
- **Cancelled meetings:** call out separately so the user knows freed-up time.
- **No prior daily note found:** first run ever, or `📅/` is empty — fall back to the standard 24h window and tell the user.
- **Last note > 14 days ago:** long vacation / extended absence — cap the window at 14 days, generate the briefing, and warn the user that older items aren't included. Suggest searching email manually for anything time-sensitive from the gap period.
- **Today's note already exists (re-run same day):** treat the *prior* day's note as `last_run_date` (not today's) so the window doesn't collapse to zero. The "overwrite/append/skip" prompt for today's note still applies at save time.
- **Backfill briefing:** the daily note still saves to today's path only — one consolidated catch-up note, not one note per missing day. The window label and executive-summary callout make the multi-day scope explicit.

## Optional deep-dive

If the user asks for more detail on a specific thread, use `outlook_get_conversation(message_id=...)` to pull the full thread.
