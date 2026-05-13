# Markdown Skill Runbook

## How to use in any harness

1. Load [SKILL_GRAPH.md](SKILL_GRAPH.md).
2. Pick one entry skill:
   - `persona_consult`
   - `council_consult`
   - `council_discuss`
3. Follow each skill's `Calls` / `Escalate To` links to chain skills.
4. Keep session state in plain JSON for `council_discuss`.

## Minimal conventions

- Inputs and outputs are plain markdown sections.
- State is explicit and caller-managed.
- Persona contracts are individual files in [personas/](personas/). See [PERSONAS.md](personas/PERSONAS.md) for the index.

## Why this is portable

- No transport assumptions.
- No vendor-specific SDK required.
- Works with prompt-based skill runners and custom orchestration.
