# Skill Graph

## Nodes

- `persona_consult`
- `council_consult`
- `council_define_personas`
- `council_discuss`

## Edges

- `council_consult -> persona_consult` (fan-out to multiple personas)
- `council_consult -> council_define_personas` (load/override persona contracts)
- `council_discuss -> council_consult` (build structured answer per turn)
- `council_discuss -> persona_consult` (deep dive on one persona)
- `persona_consult -> council_discuss` (escalate when problem needs multi-turn)
- `council_define_personas -> persona_consult` (apply updated persona profile)

## Suggested Execution Order

1. `council_define_personas` (optional)
2. `persona_consult` or `council_consult`
3. `council_discuss` when ambiguity or multi-turn debate is needed
