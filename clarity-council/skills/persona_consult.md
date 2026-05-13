# Skill: persona_consult

## Purpose

Get structured advice from exactly one persona.

## Input

- `persona_name` (required)
- `user_problem` (required)
- `context` (optional)
- `desired_outcome` (optional)
- `constraints` (optional list)
- `depth` (`brief` | `standard` | `deep`, optional)

## Output

- `summary`
- `advice`
- `assumptions`
- `questions`
- `next_steps`
- `confidence`

## Process

1. Load persona contract from the matching file in [personas/](personas/) (e.g. `senior-developer.md`). See [PERSONAS.md](personas/PERSONAS.md) for the full list.
2. Apply the persona's Decision Lens to the problem before generating advice.
3. Use the persona's Preferred Frameworks to structure reasoning.
4. Ask the persona's Default Clarifying Questions if the input is ambiguous; otherwise answer them inline as assumptions.
5. Generate advice consistent with all persona constraints.
6. Apply the persona's Output Requirements — include every mandatory field.
7. Check Escalation Conditions and flag if any are met.
8. Return structured bullets.

## Calls

- none

## Escalate To

- [council_consult.md](council_consult.md) when user asks for multiple perspectives.
- [council_discuss.md](council_discuss.md) when clarification loop is needed.
