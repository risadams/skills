# Skill: council_define_persona_groups

## Purpose

View and override active persona groups for this workspace/session.

## Input

- `overrides` (optional map by group name)
  - `members` (optional list)
  - `focus` (optional list)
  - `constraints` (optional list)
  - `description` (optional string)

## Output

- `persona_groups[]`

## Process

1. Start from canonical groups in [personas/GROUPS.md](personas/GROUPS.md).
2. Apply overrides only for known group names.
3. Return effective groups with resolved member persona names.

## Calls

- none

## Escalate To

- [council_consult.md](council_consult.md) when the user wants a focused multi-persona decision panel.
- [council_discuss.md](council_discuss.md) when the user wants the group panel to participate in a clarification loop.
