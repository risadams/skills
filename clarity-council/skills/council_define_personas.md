# Skill: council_define_personas

## Purpose

View and override active persona behavior for this workspace/session.

## Input

- `overrides` (optional map by persona name)
  - `soul` (optional)
  - `focus` (optional list)
  - `constraints` (optional list)

## Output

- `personas[]` effective contracts after overrides

## Process

1. Start from canonical contracts in [personas/PERSONAS.md](personas/PERSONAS.md).
2. Apply overrides only for known persona names.
3. Return effective contracts.

## Calls

- none

## Escalate To

- [persona_consult.md](persona_consult.md)
- [council_consult.md](council_consult.md)
- [council_define_persona_groups.md](council_define_persona_groups.md) when a pre-made panel is a better fit than a one-off persona override
